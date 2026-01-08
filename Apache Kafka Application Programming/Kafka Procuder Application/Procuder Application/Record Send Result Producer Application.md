-----
### 레코드 전송 결과를 확인하는 프로듀서
-----
1. KafkaProducer의 send() 메서드는 Future 객체를 반환
   - 이 객체는 RecordMetadata의 비동기 결과를 표현하는 것으로 ProducerRecord가 카프카 브로커에 정상적으로 적재되었는지에 대한 데이터가 포함되어 있음
2. get() 메서드를 사용하면 프로듀서로 보낸 데이터의 결과를 동기적으로 가져올 수 있음
   - ProducerWithSyncCallback
```java
package main.java.com.example;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Properties;

public class ProducerWithSyncCallback {
    private final static Logger logger = LoggerFactory.getLogger(ProducerWithSyncCallback.class);
    private final static String TOPIC_NAME = "test";
    private final static String BOOTSTRAP_SERVERS = "my-kafka:9092";

    public static void main(String[] args) {

        Properties configs = new Properties();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        // configs.put(ProducerConfig.ACKS_CONFIG, "0"); // acks=0으로 설정하면, 메타데이터 값은 의미가 없어짐 (오프셋 -1 : 응답을 받지 않았음을 의미)

        KafkaProducer<String, String> producer = new KafkaProducer<>(configs);

        ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, "Pangyo", "Pangyo");
        try {
            RecordMetadata metadata = producer.send(record).get(); // 동기로 받음
            logger.info(metadata.toString()); // 문제가 발생했을 때마다 예외로만 처리하는 것이 옵션
        } catch (Exception e) {
            logger.error(e.getMessage(),e);
        } finally {
            producer.flush();
            producer.close();
        }
    }
}
```
```
[main] INFO org.apache.kafka.clients.producer.ProducerConfig - ProducerConfig values: 
	acks = 1
  ...

[main] INFO main.java.com.example.ProducerWithSyncCallback - test-7@3 // 토픽-파티션@오프셋
```
