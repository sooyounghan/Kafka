-----
### 커스텀 파티셔너를 가지는 프로듀서
-----
1. 프로듀서 사용환경에 따라 특정 데이터를 가지는 레코드를 특정 파티션으로 보내야 할 때가 존재
   - 예를 들어, Pangyo라는 값을 가진 메세지 키가 0번 파티션으로 들어가야 한다고 가정
   - 기본 설정 파티셔너를 사용할 경우 메세지 키의 해시값을 파티션에 매칭하여 데이터를 전송하므로 어느 파티션에 들어가는지 알 수 없음
   - 💡 이 때, Partitioner 인터페이스를 사용하여 사용자 정의 파티셔너를 생성하면 Pangyo라는 값을 가진 메세지 키에 대해 무조건 파티션 0번으로 지정하도록 설정 가능

2. 이렇게 지정할 경우 파티션 개수가 변경되더라도 Pangyo라는 메세지 키를 가진 데이터는 파티션 0번에 적재
   - CustomPartitioner
```java
package main.java.com.example;

import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.InvalidRecordException;
import org.apache.kafka.common.PartitionInfo;
import org.apache.kafka.common.utils.Utils;

import java.util.List;
import java.util.Map;

public class CustomPartitioner implements Partitioner { // Partitioner 인터페이스 구현

    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        if (keyBytes == null) {
            throw new InvalidRecordException("Need message key");
        }

        if (((String)key).equals("Pangyo")) // Pangyo라는 메세지 키를 가질 때
            return 0; // 0번 파티션으로 지정

        // Pangyo가 아닌 값에 대해서는 해시값 처리
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }

    @Override
    public void configure(Map<String, ?> configs) {}

    @Override
    public void close() {}
}
```
   - ProducerWithCustomPartitioner
```java
package main.java.com.example;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class ProducerWithCustomPartitioner {
    private final static String TOPIC_NAME = "test";
    private final static String BOOTSTRAP_SERVERS = "my-kafka:9092";

    public static void main(String[] args) {

        Properties configs = new Properties();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        configs.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, CustomPartitioner.class); // CustomPartitioner 클래스 지정

        KafkaProducer<String, String> producer = new KafkaProducer<>(configs);

        ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, "Pangyo", "Pangyo");
        producer.send(record);
        producer.flush();
        producer.close();
    }
}
```
