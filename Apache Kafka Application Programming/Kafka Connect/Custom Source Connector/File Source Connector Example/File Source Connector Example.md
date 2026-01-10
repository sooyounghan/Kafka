-----
### 파일 소스 커넥터 구현 예제
-----
1. 로컬에 지정된 파일을 토픽으로 한 줄 씩 읽어서 토픽으로 보내는 파일 소스 커넥터 작성
2. 소스 커넥터를 구현하기 전 build.gradle에 connect-api 라이브러리와 빌드된 파일을 jar로 압축하기 위한 스크립트 작성
3. 카프카 커넥터를 직접 개발하고 플러그인으로 커넥트를 추가할 때 주의할 점 : 사용자가 직접 작성한 클래스 뿐만 아니라 라이브러리도 함께 빌드하여 jar로 압축해야 함
   - 만약, 개발 시 참조했던 Dependency을 같이 빌드하지 않고, 커넥트의 플러그인으로 추가하면 실행 시 참조하는 클래스를 찾지 못하고 ClassNotFoundException 발생 가능
```gradle
jar {
    from {
        configurations.compile.collect { it.isDirectory() ? it : zipTree(it) }
    }
}
```

4. 파일 구조
```
simple-source-connector
    ├── build.gradle
    ├── settings.gradle
    └── src
        └── main
            └── java
                └── com
                    └── example
                    ├── SingleFileSourceConnector.java
                    ├── SingleFileSourceConnectorConfig.java
                    └── SingleFileSourceTask.java
```

5. SingleFileSourceConnectorConfig
```java
package com.example;

import org.apache.kafka.common.config.AbstractConfig;
import org.apache.kafka.common.config.ConfigDef;
import org.apache.kafka.common.config.ConfigDef.Importance;
import org.apache.kafka.common.config.ConfigDef.Type;

import java.util.Map;

public class SingleFileSourceConnectorConfig extends AbstractConfig { // 템플릿 형태

    public static final String DIR_FILE_NAME = "file"; // 파일 (어떤 파일을 읽을 것인가?)
    private static final String DIR_FILE_NAME_DEFAULT_VALUE = "/tmp/kafka.txt"; // DEFAULT_VALUE
    private static final String DIR_FILE_NAME_DOC = "읽을 파일 경로와 이름"; // 정보

    public static final String TOPIC_NAME = "topic"; // 토픽 (어떤 토픽에 넣을 것인가?)
    private static final String TOPIC_DEFAULT_VALUE = "test";
    private static final String TOPIC_DOC = "보낼 토픽명";

    public static ConfigDef CONFIG = new ConfigDef().define(DIR_FILE_NAME,
                                                    Type.STRING,
                                                    DIR_FILE_NAME_DEFAULT_VALUE,
                                                    Importance.HIGH,
                                                    DIR_FILE_NAME_DOC)
                                                    .define(TOPIC_NAME,
                                                            Type.STRING,
                                                            TOPIC_DEFAULT_VALUE,
                                                            Importance.HIGH,
                                                            TOPIC_DOC);

    public SingleFileSourceConnectorConfig(Map<String, String> props) {
        super(CONFIG, props);
    }
}
```

6. SingleFileSourceConnector
```java
package com.example;

import org.apache.kafka.common.config.ConfigDef;
import org.apache.kafka.common.config.ConfigException;
import org.apache.kafka.connect.connector.Task;
import org.apache.kafka.connect.errors.ConnectException;
import org.apache.kafka.connect.source.SourceConnector;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class SingleFileSourceConnector extends SourceConnector { // SourceConnector 상속 (하나의 파일을 읽어서 토픽에 전송)

    private final Logger logger = LoggerFactory.getLogger(SingleFileSourceConnector.class);

    private Map<String, String> configProperties;

    @Override
    public String version() {
        return "1.0";
    }

    @Override
    public void start(Map<String, String> props) { // REST API로 템플릿 형태 (FILE, TOPIC)
        this.configProperties = props;
        try {
            new SingleFileSourceConnectorConfig(props);
        } catch (ConfigException e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public Class<? extends Task> taskClass() { // 실행할 TaskClass 지정
        return SingleFileSourceTask.class; // 여러 개라면, 상황에 따라 지정하면 됨
    }

    @Override
    public List<Map<String, String>> taskConfigs(int maxTasks) { // TASK에 어떤 Config를 넣을지 지정
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
        return SingleFileSourceConnectorConfig.CONFIG;
    }

    @Override
    public void stop() {
    }
}
```

7. SingleFileSourceTask
```java
package com.example;

import org.apache.kafka.connect.data.Schema;
import org.apache.kafka.connect.errors.ConnectException;
import org.apache.kafka.connect.source.SourceRecord;
import org.apache.kafka.connect.source.SourceTask;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.BufferedReader;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class SingleFileSourceTask extends SourceTask {
    private Logger logger = LoggerFactory.getLogger(SingleFileSourceTask.class);

    public final String FILENAME_FIELD = "filename";
    public final String POSITION_FIELD = "position";

    private Map<String, String> fileNamePartition;
    private Map<String, Object> offset;
    private String topic;
    private String file;
    private long position = -1;


    @Override
    public String version() {
        return "1.0";
    }

    @Override
    public void start(Map<String, String> props) { // 리소스 초기화 용도
        try {
            // Init variables
            SingleFileSourceConnectorConfig config = new SingleFileSourceConnectorConfig(props);
            topic = config.getString(SingleFileSourceConnectorConfig.TOPIC_NAME);
            file = config.getString(SingleFileSourceConnectorConfig.DIR_FILE_NAME);
            fileNamePartition = Collections.singletonMap(FILENAME_FIELD, file);
            offset = context.offsetStorageReader().offset(fileNamePartition); // 소스 커넥터에서 관리하는 내부 번호를 기록하는 용도 (소스 태스크 내부에서 관리하는 offset)

            // Get file offset from offsetStorageReader
            if (offset != null) { // 오프셋이 없을 경우 
                Object lastReadFileOffset = offset.get(POSITION_FIELD);
                if (lastReadFileOffset != null) {
                    position = (Long) lastReadFileOffset; // 만약 기존에 저장된 내부 번호가 있다면 해당 번호부터 시작
                }
            } else { // 만약 기존에 저장된 내부 번호가 없다면 0부터 시작
                position = 0; // 최초 0부터 시작작
            }

        } catch (Exception e) {
            throw new ConnectException(e.getMessage(), e);
        }
    }

    @Override
    public List<SourceRecord> poll() { // 데이터 처리 (List<SoruceRecord>는 send()를 통해 토픽에 전송)
        List<SourceRecord> results = new ArrayList<>();
        try {
            Thread.sleep(1000);

            List<String> lines = getLines(position); // 가져가고 싶은 position(내부 번호)부터 데이터를 읽어감

            if (lines.size() > 0) {
                lines.forEach(line -> {
                    Map<String, Long> sourceOffset = Collections.singletonMap(POSITION_FIELD, ++position);
                    SourceRecord sourceRecord = new SourceRecord(fileNamePartition, sourceOffset, topic, Schema.STRING_SCHEMA, line);
                    results.add(sourceRecord); // 토픽으로 보내고 싶은 데이터는 List<SourceRecord>에 element 추가
                });
            }
            return results; // 최종적으로 토픽으로 전송되는 List
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
            throw new ConnectException(e.getMessage(), e);
        }
    }

    private List<String> getLines(long readLine) throws Exception {
        BufferedReader reader = Files.newBufferedReader(Paths.get(file));
        return reader.lines().skip(readLine).collect(Collectors.toList());
    }

    @Override
    public void stop() {
    }
}
```
