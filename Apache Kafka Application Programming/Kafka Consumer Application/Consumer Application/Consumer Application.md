-----
### InteliJ에서 컨슈머 API 구현
-----
1. 카프카 브로커와 연동하기 위한 컨슈머를 개발하기 위해서는 라이브러리를 사용해야 함
2. 아파치 카프카는 공식 라이브러리로 자바 라이브러리를 지원하므로, 카프카 공식 자바 라이브러리를 사용하여 컨슈머 개발
3. 컨슈머를 구현하기 위해 카프카 클라이언트 라이브러리를 프로젝트에 추가
   - 라이브러리 추가는 build.gradle 파일에서 설정하여 추가
```gradle
plugins {
    id 'java'
}

group = 'org.example'
version = '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.apache.kafka:kafka-clients:2.5.0'
    implementation  'org.slf4j:slf4j-simple:1.7.30'

    testImplementation platform('org.junit:junit-bom:5.10.0')
    testImplementation 'org.junit.jupiter:junit-jupiter'
}

test {
    useJUnitPlatform()
}
```

4. SimpleConsumer
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

public class SimpleConsumer {
    private final static Logger logger = LoggerFactory.getLogger(SimpleConsumer.class);
    private final static String TOPIC_NAME = "test";
    private final static String BOOTSTRAP_SERVERS = "my-kafka:9092";
    private final static String GROUP_ID = "test-group";

    public static void main(String[] args) {
        Properties configs = new Properties();
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID); // 선택 옵션으로 group_id 설정 (subscribe()를 사용하므로 필수)
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName()); // 역직렬화 - String 
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName()); // 역직렬화 - String

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs); 

        consumer.subscribe(Arrays.asList(TOPIC_NAME));

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1)); // 레코드들을 poll() 메서드를 통해 레코드를 병렬 처리 또는 리스트를 순차 처리 : Return 값인 records는 각 개별 로크를 묶은 상태
            for (ConsumerRecord<String, String> record : records) { // 순차적으로 레코드를 로그로 출력
                logger.info("record:{}", record);
            }
        }
    }
}
```
```
PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic test
>hello
>kafka
```
```
[main] INFO com.example.SimpleConsumer - record:ConsumerRecord(topic = test, partition = 0, leaderEpoch = 0, offset = 0, CreateTime = 1767861334790, serialized key size = -1, serialized value size = 5, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello)
[main] INFO com.example.SimpleConsumer - record:ConsumerRecord(topic = test, partition = 3, leaderEpoch = 0, offset = 1, CreateTime = 1767861359437, serialized key size = -1, serialized value size = 5, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = kafka)
```

