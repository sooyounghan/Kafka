-----
### 스트림즈 DSL 라이브러리 추가
-----
1. build.gradle
```java
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
    implementation 'org.apache.kafka:kafka-streams:2.5.0'
    implementation  'org.slf4j:slf4j-simple:1.7.30'


    testImplementation platform('org.junit:junit-bom:5.10.0')
    testImplementation 'org.junit.jupiter:junit-jupiter'
}

test {
    useJUnitPlatform()
}
```

2. SimpleStreamApplication
```java
package com.example;

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;

import java.util.Properties;

public class SimpleStreamApplication {

    private static String APPLICATION_NAME = "streams-application";
    private static String BOOTSTRAP_SERVERS = "my-kafka:9092";
    private static String STREAM_LOG = "stream_log";
    private static String STREAM_LOG_COPY = "stream_log_copy";

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, APPLICATION_NAME);
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> stream = builder.stream(STREAM_LOG);

        stream.to(STREAM_LOG_COPY);

        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

    }
}
```

-----
### 필터링 스트림즈 애플리케이션
-----
1. 토픽으로 들어온 문자열 데이터 중 문자열 길이가 5보다 큰 경우만 필터링하는 스트림즈 애플리케이션을 스트림 프로세서를 사용하여 만들 수 있음
2. 메세지 키 또는 메세지 값을 필터링하여 특정 조건에 맞는 데이터를 골라낼 때는 filter() 메서드를 사용하면 됨
   - filter() 메서드는 스트림 DSL에서 사용 가능한 필터링 스트림 프로세서

<div align="center">
<img src="https://github.com/user-attachments/assets/1743ebc2-f9d3-4f33-9ca6-fd559998fb76" />
</div>

2. StreamFilter
```java
package com.example;

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;

import java.util.Properties;

public class StreamsFilter {

    private static String APPLICATION_NAME = "streams-filter-application";
    private static String BOOTSTRAP_SERVERS = "my-kafka:9092";
    private static String STREAM_LOG = "stream_log";
    private static String STREAM_LOG_FILTER = "stream_log_filter";

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, APPLICATION_NAME);
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();

        KStream<String, String> streamLog = builder.stream(STREAM_LOG); // 소스 프로세서로 데이터를 가져옴 (stream() 메서드를 통해 KStream으로 데이터를 가져옴) - STREAM_LOG라는 토픽을 각 레코드에 대해 스트림 처리하기 위해 streamLog라는 인스턴스 생성

        KStream<String, String> filteredStream = streamLog
                                                  .filter((key, value) -> value.length() > 5); // 메세지 값이 길이가 5를 초과하는 경우만 필터링 (filter() : 스트림 프로세서)
        filteredStream.to(STREAM_LOG_FILTER); // 필터링된 스트림을 기준으로 특정 토픽 데이터 저장 : to() 사용
        // streamLog.filter((key, value) -> value.length() > 5).to(STREAM_LOG_FILTER);

        KafkaStreams streams;
        streams = new KafkaStreams(builder.build(), props);
        streams.start();
    }
}
```

