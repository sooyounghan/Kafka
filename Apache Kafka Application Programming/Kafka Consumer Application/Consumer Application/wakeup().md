-----
### 컨슈머의 안전한 종료
-----
1. 컨슈머 애플리케이션은 안전하게 종료되어야 함
2. 정상적으로 종료되지 않은 컨슈머는 세션 타임아웃이 발생할 때 까지 컨슈머 그룹에 남게 됨
3. 컨슈머를 안전하게 종료하기 위해 KafkaConsumer 클래스는 wakeup() 메서드를 지원
   - wakeup() 메서드를 실행하여 KafkaConsumer 인스턴스를 안전하게 종료할 수 있음
   - wakeup() 메서드가 실행된 이후 poll() 메서드가 호출되면 WakeupException 예외가 발생
   - WakeupException 예외를 받은 뒤에는 데이터 처리를 위해 사용한 자원들을 해제하면 됨

```java
package com.example;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.errors.WakeupException;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

public class ConsumerWithSyncOffsetCommitShutdownHook {
    private final static Logger logger = LoggerFactory.getLogger(ConsumerWithSyncOffsetCommit.class);
    private final static String TOPIC_NAME = "test";
    private final static String BOOTSTRAP_SERVERS = "my-kafka:9092";
    private final static String GROUP_ID = "test-group";
    private static KafkaConsumer<String, String> consumer;

    public static void main(String[] args) {
        Runtime.getRuntime().addShutdownHook(new ShutdownThread()); // Runtime에 Shutdown Hook을 추가 : 자바 애플리케이션에 명시적으로 Shutdown Hook을 처리 가능 - wakeup() 메서드를 통해 안전하게 데이터를, 즉 컨슈머 애플리케이션 종료 가

        Properties configs = new Properties();
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID);
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        configs.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);

        consumer = new KafkaConsumer<>(configs);
        consumer.subscribe(Arrays.asList(TOPIC_NAME));

        try {
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
                for (ConsumerRecord<String, String> record : records) {
                    logger.info("{}", record);
                }
                consumer.commitSync();
            }
        } catch (WakeupException e) {
            logger.warn("Wakeup consumer");
        } finally {
            logger.warn("Consumer close");
            consumer.close();
        }
    }

    static class ShutdownThread extends Thread {
        public void run() {
            logger.info("Shutdown hook");
            consumer.wakeup();
        }
    }
}
```
