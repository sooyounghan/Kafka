-----
### 리밸런스 리스너를 가진 컨슈머
-----
1. 리밸런스 발생을 감지하기 위해 카프카 라이브러리는 ConsumerRebalanceListener 인터페이스를 지원
2. ConsumerRebalanceListener 인터페이스로 구현된 클래스는 onPartitionAssigned() 메서드와 onPartitionRevoked() 메서드로 이루어져 있음
   - onPartitionAssigned() : 리밸런스가 끝난 뒤 파티션이 할당 완료되면 호출되는 메서드 (assigned 메서드 내부에서 현재 할당되어 있는 파티션, 토픽에 대한 정보 확인 가능)
   - onPartitionRevoked() : 리밸러스가 시작되기 직전에 호출되는 메서드
     + 마지막으로 처리한 레코드를 기준으로 커밋을 하기 위해서는 리밸런스가 시작하기 직전에 커밋을 하면 됨
     + onPartitionRevoked() 메서드에 커밋을 구현하여 처리할 수 있음
3. RebalanceListener
```java
package com.example;

import org.apache.kafka.clients.consumer.ConsumerRebalanceListener;
import org.apache.kafka.common.TopicPartition;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Collection;

public class RebalanceListener implements ConsumerRebalanceListener { // ConsumerRebalanceListener 인터페이스 구현
    private final static Logger logger = LoggerFactory.getLogger(RebalanceListener.class);

    public void onPartitionsAssigned(Collection<TopicPartition> partitions) { // 리밸런싱이 발생하고 난 뒤, 현재 컨슈머에 할당된, 현재 컨슈머 스레드에 할당된 파티션의 정보 (토픽, 파티션) 
        logger.warn("Partitions are assigned : " + partitions.toString());
    }

    public void onPartitionsRevoked(Collection<TopicPartition> partitions) { // 리밸런싱이 발생되기 직전 파티션이 어떤게 할당되었는지 정보를 확인 가능
        logger.warn("Partitions are revoked : " + partitions.toString());
    }
}
```

4. ConsumerWithRebalanceListener
```java
package com.example;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

public class ConsumerWithRebalanceListener {
    private final static Logger logger = LoggerFactory.getLogger(ConsumerWithRebalanceListener.class);
    private final static String TOPIC_NAME = "test";
    private final static String BOOTSTRAP_SERVERS = "my-kafka:9092";
    private final static String GROUP_ID = "test-group";


    private static KafkaConsumer<String, String> consumer;

    public static void main(String[] args) {
        Properties configs = new Properties();
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID);
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

        consumer = new KafkaConsumer<>(configs);
        consumer.subscribe(Arrays.asList(TOPIC_NAME), new RebalanceListener()); // subscribe()에 RebalanceListener를 추가 : Subscribe 자체가 구독하는 것이므로 구독을 하는 컨슈머 그룹이 어떻게 리밸런싱 되는지확인하기 위해 파라미터로 추가

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> record : records) {
                logger.info("{}", record);
            }
        }
    }
}
```
