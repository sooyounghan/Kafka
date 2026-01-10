-----
### 분산모드 카프카 커넥트 실습
-----
1. connect-distributed.properties
```
PS C:\kafka_2.12-2.5.0> cat config/connect-distributed.properties
bootstrap.servers=my-kafka:9092 # 연동할 카프카 클러스터
group.id=connect-cluster # 그룹 아이디 : 동일한 그룹 아이디로 지정되어 실행 (커넥터를 실행시키거나 태스크 증가 시, 안정적 스케일 아웃이 될 때 용이하게 사용)

key.converter=org.apache.kafka.connect.storage.StringConverter # 컨버터 사용 여부 (StringConverter)
value.converter=org.apache.kafka.connect.storage.StringConverter
key.converter.schemas.enable=false # 스키마 사용 여부
value.converter.schemas.enable=false

# 데이터 및 상태 저장 용도 토픽 : 유일무이한 토픽으로 지정해도 되며, 접근 권한이나 또 다른 데이터를 이메일로 추가해도 가능
offset.storage.topic=connect-offsets
offset.storage.replication.factor=1

config.storage.topic=connect-configs
config.storage.replication.factor=1

status.storage.topic=connect-status
status.storage.replication.factor=1

offset.flush.interval.ms=10000

# plugin.path=따로 지정하지 않음 (커스텀 카프카 커넥트 플러그인 추가 또는 오픈소스 아파치 카프카 커넥터들을 추가한다면 디렉토리 단위로 추가)
```

2. 분산 모드 커넥트 실행, 플러그인 확인
```
$ bin/connect-distributed.sh config/connect-distributed.properties
[2021-12-03 14:01:14,219] INFO WorkerInfo values:
jvm.args = -Xms256M, -Xmx2G, -XX:+UseG1GC, -XX:MaxGCPauseMi
...
```
```
./bin/windows/connect-distributed.bat config/connect-distributed.properties
```
```
$ curl -X GET http://localhost:8083/connector-plugins
[
    {
    "class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
    "type": "sink",
    "version": "2.5.0"
    }...
```
```
irm http://localhost:8083/connector-plugins
```
```
PS C:\kafka_2.12-2.5.0> irm http://localhost:8083/connector-plugins

class                                                     type   version
-----                                                     ----   -------
org.apache.kafka.connect.file.FileStreamSinkConnector     sink   2.5.0
org.apache.kafka.connect.file.FileStreamSourceConnector   source 2.5.0
org.apache.kafka.connect.mirror.MirrorCheckpointConnector source 1
org.apache.kafka.connect.mirror.MirrorHeartbeatConnector  source 1
org.apache.kafka.connect.mirror.MirrorSourceConnector     source 1
```

3. FileStreamSinkConnector 테스트
```
$ curl -X POST \
http://localhost:8083/connectors \
-H 'Content-Type: application/json' \ # JSON 형태로 Payload 지정
-d '{
"name": "file-sink-test",  # 해당 이름을 바탕으로 파이프라인이 생성
"config":
    {
    "topics":"test", # 특정 토픽에
    "connector.class":"org.apache.kafka.connect.file.FileStreamSinkConnector", # FileStreamSinkConnector 클래스를 기반으로 파이프라인을 신규로 생성
    "tasks.max":1, # 실행할 테스트 스레드 개수 지정 (보통 파티션 개수와 동일하게 운용하는 것이 일반적)
    "file":"/tmp/connect-test.txt" # 해당 파일 저장
    }
}
```
```
$body = @'
{
  "name": "file-sink-test",
  "config": {
    "topics": "test",
    "connector.class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
    "tasks.max": "1",
    "file": "C:/tmp/connect-test.txt"
  }
}
'@

Invoke-RestMethod -Method Post -Uri http://localhost:8083/connectors -ContentType "application/json" -Body $body
```
```
PS C:\kafka_2.12-2.5.0> $body = @'
>> {
>>   "name": "file-sink-test",
>>   "config": {
>>     "topics": "test",
>>     "connector.class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
>>     "tasks.max": "1",
>>     "file": "C:/tmp/connect-test.txt"
>>   }
>> }
>> '@
PS C:\kafka_2.12-2.5.0>
PS C:\kafka_2.12-2.5.0> Invoke-RestMethod -Method Post -Uri http://localhost:8083/connectors -ContentType "application/json" -Body $body

name           config
----           ------
file-sink-test @{topics=test; connector.class=org.apache.kafka.connect.file.FileStreamSinkConnector; tasks.max=1; fi...
```

```
irm http://localhost:8083/connectors
```
```
PS C:\kafka_2.12-2.5.0> irm http://localhost:8083/connectors
file-sink-test
```

4. FileStreamSinkConnector 실행 확인
```
$ curl http://localhost:8083/connectors/file-sink-test/status
{
    "name": "file-sink-test",
    "connector": { # 커넥터
    "state": "RUNNING",
    "worker_id": "127.0.0.1:8083"
    },
    "tasks": [ # 태스크
    {
        "id": 0,
        "state": "RUNNING",
        "worker_id": "127.0.0.1:8083"
    }
    ],
    "type": "sink" # 타입
}
```
```
irm http://localhost:8083/connectors/file-sink-test/status
JSON : irm http://localhost:8083/connectors/file-sink-test/status | ConvertTo-Json -Depth 5
```
```
PS C:\kafka_2.12-2.5.0> irm http://localhost:8083/connectors/file-sink-test/status

name           connector                                        tasks
----           ---------                                        -----
file-sink-test @{state=RUNNING; worker_id=169.254.232.148:8083} {@{id=0; state=RUNNING; worker_id=169.254.232.148:80...
```
```
PS C:\kafka_2.12-2.5.0> irm http://localhost:8083/connectors/file-sink-test/status | ConvertTo-Json -Depth 5
{
    "name":  "file-sink-test",
    "connector":  {
                      "state":  "RUNNING",
                      "worker_id":  "169.254.232.148:8083"
                  },
    "tasks":  [
                  {
                      "id":  0,
                      "state":  "RUNNING",
                      "worker_id":  "169.254.232.148:8083"
                  }
              ],
    "type":  "sink"
}
```

5. FileStreamSinkConnector로 생성된 파일 확인
```
$ bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic test
>a
>b
>c
>d
>e
>f
>g

$ cat /tmp/connect-test.txt
a
b
c
d
e
```
```
PS C:\kafka_2.12-2.5.0> ./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic test
>a
>b
>c
>d
>e
>f
>g

PS C:\kafka_2.12-2.5.0> cat /tmp/connect-test.txt
a
b
c
d
e
```

6. FileStreamSinkConnector 종료
```
$ curl http://localhost:8083/connectors # 해당 커넥터 이름 확인 
["file-sink-test"]

$ curl -X DELETE http://localhost:8083/connectors/file-sink-test # 해당 커넥트 종료

$ curl http://localhost:8083/connectors
[]
```
```
irm http://localhost:8083/connectors
file-sink-test

Invoke-RestMethod -Method Delete -Uri http://localhost:8083/connectors/file-sink-test

Invoke-RestMethod http://localhost:8083/connectors
```
