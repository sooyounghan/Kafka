-----
### 파일 싱크 커넥터 구현 예제
------
1. 로컬에 지정된 파일에 토픽의 데이터를 한 줄씩 저장하는 파일 싱크 커넥터 작성
```
simple-sink-connector
├── build.gradle
├── settings.gradle
└── src
    └── main
        └── java
            └── com
                └── example
                ├── SingleFileSinkConnector.java
                ├── SingleFileSinkConnectorConfig.java
                └── SingleFileSinkTask.java
```

2. SingleFileSinkConnectorConfig
```java
package com.example;

import org.apache.kafka.common.config.AbstractConfig;
import org.apache.kafka.common.config.ConfigDef;
import org.apache.kafka.common.config.ConfigDef.Importance;
import org.apache.kafka.common.config.ConfigDef.Type;

import java.util.Map;

public class SingleFileSinkConnectorConfig extends AbstractConfig { // AbstractConfig에는 topics라고 하는 기본 설정 값을 받음 (하지만, 파이프라인을 실행할 때, SinkConnector를 실행할 때는 topics 지정 필요)

    public static final String DIR_FILE_NAME = "file";
    private static final String DIR_FILE_NAME_DEFAULT_VALUE = "/tmp/kafka.txt";
    private static final String DIR_FILE_NAME_DOC = "저장할 디렉토리와 파일 이름";

    public static ConfigDef CONFIG = new ConfigDef().define(DIR_FILE_NAME,
                                                    Type.STRING,
                                                    DIR_FILE_NAME_DEFAULT_VALUE,
                                                    Importance.HIGH,
                                                    DIR_FILE_NAME_DOC);

    public SingleFileSinkConnectorConfig(Map<String, String> props) {
        super(CONFIG, props);
    }
}
```

3. SingleFileSinkConnector
```java
package com.example;

import org.apache.kafka.common.config.ConfigDef;
import org.apache.kafka.common.config.ConfigException;
import org.apache.kafka.connect.connector.Task;
import org.apache.kafka.connect.errors.ConnectException;
import org.apache.kafka.connect.sink.SinkConnector;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class SingleFileSinkConnector extends SinkConnector {

    private Map<String, String> configProperties;

    @Override
    public String version() {
        return "1.0";
    }

    @Override
    public void start(Map<String, String> props) {
        this.configProperties = props;
        try {
            new SingleFileSinkConnectorConfig(props);
        } catch (ConfigException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public Class<? extends Task> taskClass() {
        return SingleFileSinkTask.class;
    }

    @Override
    public List<Map<String, String>> taskConfigs(int maxTasks) {
        List<Map<String, String>> taskConfigs = new ArrayList<>();
        Map<String, String> taskProps = new HashMap<>();
        taskProps.putAll(configProperties);
        for (int i = 0; i < maxTasks; i++) {
            taskConfigs.add(taskProps);
        }
        return taskConfigs;
    }

    @Override
    public ConfigDef config() {
        return SingleFileSinkConnectorConfig.CONFIG;
    }

    @Override
    public void stop() {
    }
}
```

4. SingleFileSinkTask
```java
package com.example;

import org.apache.kafka.clients.consumer.OffsetAndMetadata;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.connect.errors.ConnectException;
import org.apache.kafka.connect.sink.SinkRecord;
import org.apache.kafka.connect.sink.SinkTask;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.Collection;
import java.util.Map;

public class SingleFileSinkTask extends SinkTask {
    private SingleFileSinkConnectorConfig config;
    private File file;
    private FileWriter fileWriter;

    @Override
    public String version() {
        return "1.0";
    }

    @Override
    public void start(Map<String, String> props) {
        try {
            config = new SingleFileSinkConnectorConfig(props);
            file = new File(config.getString(config.DIR_FILE_NAME));
            fileWriter = new FileWriter(file, true);
        } catch (Exception e) {
            throw new ConnectException(e.getMessage(), e);
        }

    }

    @Override
    public void put(Collection<SinkRecord> records) { // 토픽에서 polling 되어 처리가 되어야 하는 다수의 레코드
        try {
            for (SinkRecord record : records) {
                fileWriter.write(record.value().toString() + "\n");
            }
        } catch (IOException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public void flush(Map<TopicPartition, OffsetAndMetadata> offsets) { // 커밋 시점마다 호출되는 메서드
        try {
            fileWriter.flush();
        } catch (IOException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public void stop() {
        try {
            fileWriter.close();
        } catch (IOException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }
}
```

