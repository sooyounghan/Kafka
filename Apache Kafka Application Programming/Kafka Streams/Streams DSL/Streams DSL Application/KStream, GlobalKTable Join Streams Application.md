-----
### 스트림즈 DSL - GlobalKTable과 KStream을 join() 
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/93fcd9db-ede9-46a8-8387-87dfb24e33d5" />
</div>

1. order 토픽과 address 토픽은 코파티셔닝 되어있으므로 각각 KStream과 KTable로 선언해서 조인 가능
2. 그러나 코파티셔닝이 되어있지 않은 토픽을 조인해야 할 경우
   - 코파티셔닝되지 않은 데이터를 조인하는 방법
     + 리파티셔닝(특정 토픽에 있는 파티션을 조인하고자 하는 토픽의 파티션 개수와 맞춰주는 작업)을 수행한 이후 코파티셔닝이 된 상태로 조인 처리
     + KTable로 사용하는 토픽을 GlobalKTable로 선언하여 사용

   - 파티션 개수가 다른 2개의 토픽을 조인하는 예제
     + 파티션 2개로 이루어진 address_v2 토픽 새로 생성
     + address_v2 토픽은 이전 예제에서 진행했던 address 토픽과 동일하게 이름과 주소로 이루어진 레코드 저장
     + 다만, adddress_v2 토픽은 파티션이 2개이고, KStream으로 사용하는 order는 파티션이 3개이므로 코파티셔닝이 되지 않은 상태
```
PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-topics.bat --create --bootstrap-server my-kafka:9092 --partitions 2 --topic address_v2

Created topic address_v2.
```

3. KStreamJoinGlobalKTable
```java
package com.example;

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.GlobalKTable;
import org.apache.kafka.streams.kstream.KStream;

import java.util.Properties;

public class KStreamJoinGlobalKTable {

    private static String APPLICATION_NAME = "global-table-join-application";
    private static String BOOTSTRAP_SERVERS = "my-kafka:9092";
    private static String ADDRESS_GLOBAL_TABLE = "address_v2"; // 파티션이 2개
    private static String ORDER_STREAM = "order";
    private static String ORDER_JOIN_STREAM = "order_join";

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, APPLICATION_NAME);
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();
        GlobalKTable<String, String> addressGlobalTable = builder.globalTable(ADDRESS_GLOBAL_TABLE);
        KStream<String, String> orderStream = builder.stream(ORDER_STREAM);

        orderStream.join(addressGlobalTable,
                        (orderKey, orderValue) -> orderKey // orderKey를 기준으로 join
                        (order, address) -> order + " send to " + address)
                .to(ORDER_JOIN_STREAM);

        KafkaStreams streams;
        streams = new KafkaStreams(builder.build(), props);
        streams.start();

    }
}
```

```
PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic address_v2 --property "parse.key=true" --property "key.separator=:"
>wonyoung:Busan

PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic order --property "parse.key=true" --property "key.separator=:"
>wonyoung:Tesla

PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic order_join --from-beginning
Tesla send to Busan
```
