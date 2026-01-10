-----
### 커스텀 싱크 커넥터
-----
<img width="692" height="499" alt="image" src="https://github.com/user-attachments/assets/bc2ab153-7af3-4f39-ba35-92edd51a34fb" />

1. 싱크 커넥터는 토픽의 데이터를 타깃 애플리케이션 또는 타깃 파일로 저장하는 역할
2. 카프카 커넥트 라이브러리에서 제공하는 SinkConnector와 SinkTask 클래스를 사용하면 직접 싱크 커넥터 구현 가능
3. 직접 구현한 싱크 커넥트는 빌드하여 jar로 만들고, 커넥트의 플러그인으로 추가하여 사용할 수 있음
4. SinkConnector
```java
public class TestSinkConnector extends SinkConnector {
    @Override
    public String version() {}

    @Override
    public void start(Map<String, String> props) {}

    @Override
    public Class<? extends Task> taskClass() {}

    @Override
    public List<Map<String, String>> taskConfigs(int maxTasks) {}

    @Override
    public ConfigDef config() {}

    @Override
    public void stop() {}
}
```

2. SinkTask
```java
public class TestSinkTask extends SinkTask {
    @Override
    public String version(){}

    @Override
    public void start(Map<String, String> props) {}

    @Override
    public void put(Collection<SinkRecord> records) {}

    @Override
    public void flush(Map<TopicPartition, OffsetAndMetadata> offsets) {}

    @Override
    public void stop() {}
}
```
