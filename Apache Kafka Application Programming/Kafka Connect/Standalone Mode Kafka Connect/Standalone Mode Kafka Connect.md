-----
### 단일 모드 커넥트 설정 및 실행
-----
1. 단일 모드 커넥트를 실행하기 위해서는 단일 모드 커넥트를 참조하는 설정 파일인 conenct-standalone.properties 파일 수정
   - 커넥트 설정 파일
```properties
bootstrap.servers=my-kafka:9092

key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter

key.converter.schemas.enable=false
value.converter.schemas.enable=false

offset.storage.file.filename=/tmp/connect.offsets
offset.flush.interval.ms=10000

plugin.path=/usr/local/share/java,/usr/local/share/kafka/plugins
```

2. 단일 모드 커넥트를 실행 시 파라미터로 커넥트 설정파일과 커넥터 설정 파일을 차레로 넣어 실행
   - 커넥터 설정 파일
```properties
name=local-file-source
connector.class=FileStreamSource
tasks.max=1
file=/tmp/test.txt
topic=connect-test
```
```
./bin/windows/connect-standalone.bat ./config/connect-standalone.propertoes ./config/connect-file-source.properties
```
