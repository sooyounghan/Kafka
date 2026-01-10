-----
### 프로세서 API
-----
1. 스트림즈 DSL보다 투박한 코드를 가지지만, 토폴로지를 기준으로 데이터를 처리한다는 관점에서 동일한 역할
2. 스트림즈 DSL은 데이터 처리 / 분기 / 조인을 위한 다양한 메서드를 제공 : 추가적인 상세 로직의 구현이 필요하다면 프로세서 API를 활용할 수 있음
3. 프로새서 API에서는 스트림즈 DSL에서 사용했던 KStream, KTable, GlobalKTable 개념이 없다는 점을 주의해야함
   - 다만, 스트림즈 DSL과 프로세서 API는 함께 구현하여 사용할 때는 활용 가능
4. 프로세서 API를 구현하기 위해서는 Processor 또는 Transformer 인터페이스로 구현한 클래스가 필요
   - Processor 인터페이스는 일정 로직이 이루어진 뒤, 다음 프로세서로 데이터가 넘어가지 않을 때 사용 (ProcessContext를 통해 데이터 넘기기 가능)
   - 반면, Transformer 인터페이스는 일정 로직이 이루어진 뒤, 다음 프로세서로 데이터를 넘길 때 사용
5. FilterProcessor
```java
package com.example;

import org.apache.kafka.streams.processor.Processor;
import org.apache.kafka.streams.processor.ProcessorContext;

public class FilterProcessor implements Processor<String, String> { // Processor 인터페이스 구현 (메세지 키, 메세지 값)

    private ProcessorContext context;

    @Override
    public void init(ProcessorContext context) {
        this.context = context; // 필요한 리소스 지정
    }

    @Override
    public void process(String key, String value) { // 레코드에 대해 메세지 키와 메세지 값을 내부적으로 처리
        if (value.length() > 5) { // 메세지 값이 5가 초과할 경우, 다음 프로세서로 넘김
            context.forward(key, value); // ProcessorContext를 사용해 특정 데이터를 forward, 다음 프로세서로 데이터를 넘김
        }
        context.commit(); // 명시적 커밋 (레코드 처리)
    }

    @Override
    public void close() { // 안전적 종료
    }

}
```

6. SimpleKafkaProcessor
```java
package com.example;

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;

import java.util.Properties;

public class SimpleKafkaProcessor {

    private static String APPLICATION_NAME = "processor-application";
    private static String BOOTSTRAP_SERVERS = "my-kafka:9092";
    private static String STREAM_LOG = "stream_log";
    private static String STREAM_LOG_FILTER = "stream_log_filter";

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, APPLICATION_NAME);
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        Topology topology = new Topology(); // 토폴리지 사용
        topology.addSource("Source", // 소스 프로세서
                        STREAM_LOG)
                .addProcessor("Process", // 스트림 프로세서
                        () -> new FilterProcessor(), // 데이터 처리 후, 출력
                        "Source")
                .addSink("Sink", // 싱크 프로세서
                        STREAM_LOG_FILTER,
                        "Process");

        KafkaStreams streaming = new KafkaStreams(topology, props);
        streaming.start();
    }
}
```
