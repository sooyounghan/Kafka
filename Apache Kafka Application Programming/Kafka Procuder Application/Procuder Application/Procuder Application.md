-----
### 프로듀서 애플리케이션 개발
-----
1. 인텔리제이에서 프로듀서API 구현하기
   - 카프카 브로커와 연동하기 위한 프로듀서를 사용하기 위해서는 라이브러를 사용해야 함
   - 아파치 카프카는 공식 라이브러리로 자바 라이브러리를 제공

<img width="1145" height="665" alt="image" src="https://github.com/user-attachments/assets/010775ab-b615-4dff-8b46-b3f19ed3efcb" />
<img width="1232" height="672" alt="image" src="https://github.com/user-attachments/assets/31ea59cd-c297-468f-8b9c-8c53ed405ea9" />

2. 프로듀서를 구현하기 위해 카프카 클라이언트 라이브러리를 프로젝트에 추가해야 함
   - 라이브러리 추가는 build.gradle 파일에서 설정하여 추가 가능
```gradle
plugins {
    id 'java'
}
group 'com.example'
version '1.0'
repositories { mavenCentral() }
dependencies {
    implementation  'org.apache.kafka:kafka-clients:2.5.0'
    implementation  'org.slf4j:slf4j-simple:1.7.30'
}
```

3. 예제 코드
```java
public class SimpleProducer {
    private final static Logger logger = LoggerFactory.getLogger(SimpleProducer.class);
    private final static String TOPIC_NAME = "test"; // 토픽 이름 설정
    private final static String BOOTSTRAP_SERVERS = "my-kafka:9092"; // 부트스트랩 서버 설정

    public static void main(String[] args) {
        Properties configs = new Properties(); // Properties 지정

        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS); // BootStrap 서버 설정
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,  StringSerializer.class.getName()); // KeySerializer 설정
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName()); // ValueSerializer 설정

        KafkaProducer<String, String> producer = new KafkaProducer<>(configs); // 프로듀서 Instance 생성 (Message Key와 Message Value를 직렬화하여 전송)

        String messageValue = "testMessage";
        ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, messageValue); // ProducerRecord 생성 (토픽 이름과, MessageValue 값을 받음)
        producer.send(record); // record 인스턴스를 통해 레코드가 브로커로 전송 (Accumulator가 배치로 모아 한 번에 전송)
        logger.info("{}", record);
        producer.flush(); // 강제로 브로커로 전송
        producer.close(); // 자원 정리
    }
}
```
