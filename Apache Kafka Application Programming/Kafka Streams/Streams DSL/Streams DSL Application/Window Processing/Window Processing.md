-----
### 스트림즈 DSL - Window Processing
-----
1. 스트림 데이터를 분석할 때 가장 많이 활용하는 프로세싱 중 하나는 윈도우 연산
   - 윈도우 연산 : 특정 시간에 대응하여 취합 연산을 처리할 때 활용
   - 카프카 스트림즈에서 제공하는 윈도우 프로세싱은 4가지를 지원

2. 모든 프로세싱은 메세지 키를 기준으로 취함 : 그러므로 해당 토픽에 동일한 파티션에는 동일한 메세지 키가 있는 레코드가 존재해야지만 정확한 취합이 가능
3. 만약 커스텀 파티셔너를 사용하여 동일 메세지 키가 동일한 파티션에 저장되는 것을 보장하지 못하거나 메세지 키를 넣지 않으면 관련 연산이 불가능
4. 카프카 스트림즈에서 제공하는 윈도우 연산 종류
   - 텀블링 윈도우
   - 호핑 윈도우
   - 슬라이딩 윈도우
   - 세션 윈도우

-----
### 텀블링 윈도우
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/28152a47-4a82-4958-8afa-838f819f7f89" />
</div>

1. 서로 겹치지 않은 윈도우를 특정 간격으로 지속적으로 처리할 때 사용
2. 윈도우 최대 사이즈에 도달하면 해당 시점에 데이터를 취합하여 결과를 도출
3. 단위 시간 당 데이터가 필요할 경우 사용할 수 있음
4. 예) 매 5분간 접속한 고객의 수를 측정하여 방문자 추이를 실시간으로 취합하는 경우
4. 예제 코드
   - 카프카 스트림즈에서 텀블링 윈도우를 사용하기 위해 groupByKey와 windowedBy를 사용
   - windowedBy의 파라미터는 텀블링 윈도우의 사이즈를 뜻함
   - 이후 텀블링 연산으로 출력된 데이터는 KTable으로 커밋 interval마다 출력
```java
package com.example;

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.Properties;

public class KStreamCountApplication {

    private final static Logger log = LoggerFactory.getLogger(KStreamCountApplication.class);

    private static String APPLICATION_NAME = "stream-count-application";
    private static String BOOTSTRAP_SERVERS = "my-kafka:9092";
    private static String TEST_LOG = "test";

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, APPLICATION_NAME);
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.COMMIT_INTERVAL_MS_CONFIG, 10000);

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> stream = builder.stream(TEST_LOG);

        KTable<Windowed<String>, Long> countTable = stream.groupByKey()
                .windowedBy(TimeWindows.of(Duration.ofSeconds(5)))
                .count();
        countTable.toStream().foreach(((key, value) -> {
            log.info(key.key() + " is [" + key.window().startTime() + "~" + key.window().endTime() + "] count : " + value);
        }));

        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

    }
}
```

-----
### 호핑 윈도우
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/44cf66d9-5c71-40e4-91da-78cd0607d6ba" />
</div>

1. 일정 시간 간격으로 겹치는 윈도우가 존재하는 윈도우 연산을 처리할 경우 사용
2. 호핑 윈도우는 윈도우 사이즈와 윈도우 간격 2가지 변수를 가짐
   - 윈도우 사이즈는 연산을 수행할 때 최대 윈도우 사이즈를 뜻함
   - 윈도우 간격은 서로 다른 윈도우 간 간격을 뜻함
3. 텀블링 윈도우와 다르게 동일한 키의 데이터는 서로 다른 윈도우에서 여러 번 연산될 수 있음

-----
### 슬라이딩 윈도우
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/d4d9433c-b246-47b8-98aa-fcf0cb663f28" />
</div>

: 호핑 윈도우와 유사하지만 데이터의 정확한 시간을 바탕으로 윈도우 사이즈에 포함되는 데이터를 모두 연산에 포함시키는 특징이 있음

-----
### 세션 윈도우
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/d0c3a55d-202e-48e3-a7c0-12cc713fd937" />
</div>

1. 동일 메세지 키의 데이터를 한 세션에 묶어 사용할 때 사용
2. 세션의 최대 만료시간에 따라 윈도우 사이즈가 달라짐
3. 세션 만료 시간이 지나게 되면 세션 윈도우가 종료되고, 해당 윈도우의 모든 데이터를 취합하여 연산
4. 따라서, 세션 윈도우의 윈도우 사이즈는 가변적

-----
### 윈도우 연산 시 주의사항
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/7b817f42-58cc-4e8d-be88-a7e149a64f93" />
</div>

1. 카프카 스트림즈는 커밋(기본값 30초)을 수행할 때 윈도우 사이즈가 종료되지 않아도 중간 정산 데이터를 출력
2. 커밋 시점마다 윈도우 연산 데이터를 출력하므로 동일 윈도우 사이즈(시간) 데이터는 2개 이상 출력될 수 있음

<div align="center">
<img src="https://github.com/user-attachments/assets/7dac9b6e-aef3-4336-b4a5-ed305a392cd9" />
</div>

3. 최종적으로 각 윈도우에 맞는 데이터를 출력하고 싶다면, Windowed를 기준으로 동일 윈도우 시간 데이터는 겹쳐쓰기(upsert)하는 방식으로 처리하는 것이 좋음
   - 예를 들어, 0 ~ 5초의 A 데이터가 포함된 윈도우 취합 데이터가 들어오면, 해당 데이터를 유니크 키로 설정하고 새로 들어온 데이터를 겹쳐쓰는 것
   - 위 경우에는 최초 0 ~ 5초 A 데이터가 2개 취합된 데이터가 처음 저장되고, 추후 6초에 출력된 3개 취합된 데이터가 최종 저장
   - 결과적으로 A가 0 ~ 5초에 3개 count된 것을 확인할 수 있게됨

   
