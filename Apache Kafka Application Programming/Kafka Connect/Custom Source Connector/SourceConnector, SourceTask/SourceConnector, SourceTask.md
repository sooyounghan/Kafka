-----
### SourceConnector, SourceTask
-----
1. SourceConntector
   - 태스크를 실행하기 전 커넥터 설정파일을 초기화하고 어떤 태스크 클래스를 사용할 것인지 정의하는데 사용
   - 그렇기 떄문에, 실질적인 데이터를 다루는 부분이 들어가지 않음
```java
public class TestSourceConnector extends SourceConnector {

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

2. SourceTask
   - 실제 데이터를 다루는 클래스
   - 소스 애플리케이션 또는 소스 파일로부터 데이터를 가져와 토픽으로 데이터를 보내는 역할 수행
   - 💡 토픽에서 사용하는 오프셋이 아닌 자체적으로 사용하는 오프셋을 사용
   - 즉, SourceTask에서 사용하는 오프셋은 소스 애플리케이션 또는 소스 파일을 어디까지 읽었는지 저장하는 역할을 함 (따라서, 어떤 키 값이 있어야하고, 그 키 값에 대해 어디까지 읽었는지 반드시 저장해야 함)
     + 이 오프셋을 통해 데이터를 중복해서 토픽으로 보내는 것을 방지할 수 있음
     + 예를 들어, 파일의 데이터를 한 줄씩 읽어서 토픽으로 데이터를 보낸다면, 토픽으로 데이터를 보낸 줄 번호(Line)를 오프셋에 저장 가능
```java
public class TestSourceTask extends SourceTask {

    @Override
    public String version() {}

    @Override
    public void start(Map<String, String> props) {}

    @Override
    public List<SourceRecord> poll() {}

    @Override
    public void stop() {}
}
```
