-----
### 분산 모드 커넥트
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/a0b73bde-9319-4dca-a32c-9362e99d0138" />
</div>

1. 단일 모드 커넥트와 다르게 2개 이상의 프로세스가 1개의 그룹으로 묶여서 운영 (지속적으로 Scale-Out이 가능)
   - 동일한 group.id로 설정이 필요하며, 동일한 bootstrap.server로 실행되어 여러 개의 커넥터들은 하나의 클러스터로 분산되어 묶어져서 운영
2. 이를 통해 1개의 커넥트 프로세스에 이슈가 발생하여 종료되더라도 살아있는 나머지 1개 커넥트 프로세스가 커넥터를 이어받아서 파이프라인을 지속적으로 실행할 수 있다는 특징이 존재
3. 분산 모드 커넥트 운영을 위한 설정 : 분산 모드 설정 파일인 connect-distributed.properties (카프카 바이너리 디렉토리의 config 디렉토리에 존재)
```properties
bootstrap.servers=my-kafka:9092
group.id=connect-cluster

key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false

offset.storage.topic=connect-offsets
offset.storage.replication.factor=1

config.storage.topic=connect-configs
config.storage.replication.factor=1

status.storage.topic=connect-status
status.storage.replication.factor=1

offset.flush.interval.ms=10000
plugin.path=/usr/local/share/java,/usr/local/share/kafka/plugins
```
   - 커넥트 프로세스를 실행하면, 기본적으로 토픽(offset, config, status)이 있는지 확인하고, 없다면 생성한 뒤, 카프카 클러스터와 연동하면서 플러그인에 있는 디렉토리에 있는 플러그인들을 추가시켜서 실행
   - 실행된 커넥트 자체는 파이프라인 역할을 하는 것이 아닌, 파이프라인 역할을 하는 스레드를 실행시키려면 사용자가 REST API라는 HTTP 통신을 통해 스레드를 실행시키라는 명령어로 실행
