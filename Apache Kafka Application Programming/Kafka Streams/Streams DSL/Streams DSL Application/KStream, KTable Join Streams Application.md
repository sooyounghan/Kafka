-----
### 스트림즈 DSL - KTable과 KStream을 join()
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/cfcd3772-6456-42d2-b98e-69b5de73ceb4" />
</div>

1. KTable과 KStream은 메세지 키를 기준으로 조인할 수 있음
   - 대부분 데이터베이스는 정적으로 저장된 데이터를 조인하여 사용했지만, 카프카에서는 실시간으로 들어오는 데이터들을 조인 가능
   - 사용자의 이벤트 데이터를 데이터베이스에 저장하지 않고도 조인하여 스트리밍 처리 가능하다는 장점 존재
   - 이를 통해 이벤트 기반 스트리밍 데이터 파이프라인을 구성할 수 있음

2. 이름을 메세지 키, 주소를 메세지 값으로 가지고 있는 KTable이 있고, 주문할 물품을 메세지 값으로 가지고 있는 KStream이 존재한다고 가정
   - 사용자가 물품을 주문하면 이미 토픽에 저장된 이름:주소로 구성된 KTable과 조인하여 물품과 주소가 조합된 데이터를 새로 생성 가능
<div align="center">
<img src="https://github.com/user-attachments/assets/202bd59c-eb93-49ef-a605-1befc9172277" />
</div>


3. 만약, 사용자의 주소가 변경되는 경우에는 KTable은 동일한 메세지 키가 들어올 경우 가장 마지막 레코드를 유효한 레코드가 되므로 가장 최근에 바뀐 주소로 조인 수행
<div align="center">
<img src="https://github.com/user-attachments/assets/bdd2ef0a-a422-4f55-b3dd-54e80a7a750c" />
</div>

4. 스트림 데이터 join을 위한 토픽 생성 (코파티셔닝 과정 실시 : 파티션이 동일하게 3개를 가지고 Default Parition을 사용)
```
./bin/windows/kafka-topics.bat --create --bootstrap-server my-kafka:9092 --partitions 3 --topic address
./bin/windows/kafka-topics.bat --create --bootstrap-server my-kafka:9092 --partitions 3 --topic order
./bin/windows/kafka-topics.bat --create --bootstrap-server my-kafka:9092 --partitions 3 --topic order_join
```
<div align="center">
<img src="https://github.com/user-attachments/assets/b4cef223-c739-4fa7-8c26-7b382c63264b" />
</div>

5. KStreamJoinKTable
```java
package com.example;

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KTable;

import java.util.Properties;

public class KStreamJoinKTable {

    private static String APPLICATION_NAME = "order-join-application";
    private static String BOOTSTRAP_SERVERS = "my-kafka:9092";
    private static String ADDRESS_TABLE = "address";
    private static String ORDER_STREAM = "order";
    private static String ORDER_JOIN_STREAM = "order_join";

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, APPLICATION_NAME);
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();
        KTable<String, String> addressTable = builder.table(ADDRESS_TABLE); // adress를 위한 프로세서 (table() : KTable을 만들기 위해 필요한 소스 프로세서)
        KStream<String, String> orderStream = builder.stream(ORDER_STREAM); // order를 위한 프로세서 (stream() : KStream으로 데이터를 한 번씩 가져오기 위해 만드는 소스 프로세서)

        orderStream.join(addressTable, (order, address) -> order + " send to " + address).to(ORDER_JOIN_STREAM); // join(조인을 수행할 KTable, (메세지 값, 메세지 값) -> stream) - 두 메세지 값에 대해 메세지 키가 동일할 경우에만 join / to() : 데이터 저장

        KafkaStreams streams;
        streams = new KafkaStreams(builder.build(), props);
        streams.start();

    }
}
```

```
PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic address --property "parse.key=true" --property "key.separator=:"
>wonyoung:Seoul
>somin:Newyork
>wonyoung:Seoul
>somin:Newyork

PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic order --property "parse.key=true" --property "key.separator=:"
>somin:cup
>somin:cup
>wonyoung:iPhone

PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic order_join --from-beginning
cup send to Newyork
cup send to Newyork
cup send to Newyork
iPhone send to Busan
```

   - address, order 토픽 데이터 추가
```
PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic address --property "parse.key=true" --property "key.separator=:"
>wonyoung:Seoul
>somin:Busan

PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic order --property "parse.key=true" --property "key.separator=:"
>somin:iPhone
>wonyoung:Galaxy
```

   - order 토픽, address 토픽 join() 결과
```
PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic order_join --property print.key=true --property key.separator=":" --from-beginning
somin:iPhone send to Busan
wonyoung:Galaxy send to Seoul
```

   - 신규 address, order 데이터 추가
```
PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic address --property "parse.key=true" --property "key.separator=:"
>wonyoung:Jeju

PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic order --property "parse.key=true" --property "key.separator=:"
>wonyoung:Tesla
```

   - 신규 데이터 join() 결과
```
PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic order_join --property print.key=true --property key.separator=":" --from-beginning
somin:iPhone send to Busan
wonyoung:Galaxy send to Seoul
wonyoung:Tesla send to Jeju
```
<div align="center">
<img src="https://github.com/user-attachments/assets/c902bc88-d452-4d9a-abd8-65d29d69b687" />
</div>
