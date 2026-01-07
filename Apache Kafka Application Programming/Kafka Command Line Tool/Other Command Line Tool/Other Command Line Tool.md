-----
### kakfa-producer-perf-test
-----
1. 카프카 프로듀서로 퍼포먼스를 측정할 때 사용
2. 명령어
```
./bin/windows/kafka-producer-perf-test.bat --producer-props bootstrap.servers=my-kakfa:9092 --topic hello.kafka --num-records 10 --throughput 1 --record-size 100 --print-metric
```
<div align="center">
<img width="1444" height="540" alt="image" src="https://github.com/user-attachments/assets/810598fd-75f8-4ae4-974c-4206dc4208aa" />
</div>

-----
### kakfa-consumer-perf-test
-----
1. 카프카 컨슈머로 퍼포먼스를 측정할 때 사용
2. 카프카 브로커와 컨슈머(해당 스크립트를 돌리는 호스트) 간의 네트워크를 체크할 때 사용 가능
3. 명령어
```
./bin/windows/kafka-consumer-perf-test.bat --bootstrap.server my-kakfa:9092 --topic hello.kafka --messages 10 --throughput 1 --record-size 100 --show-detailed-stats
```
<div align="center">
<img width="1404" height="452" alt="image" src="https://github.com/user-attachments/assets/04eac01e-f02d-450c-9861-b0a6212a7de9" />
</div>

-----
### kafka-reassgin-partitions
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/f45c6a9c-8768-4ddc-953e-8322c943e845" />
</div>

1. 리더 파티션과 팔로워 파티션이 위치를 변경할 수 있음
2. 카프카 브로커에는 auto.leader.rebalance.enable 옵션이 있는데, 이 옵션의 기본값은 true로써, 클러스터 단위에서 리더 파티션을 자동 리밸런싱하도록 도와줌
3. 브로커의 백그라운드 스레드가 일정한 간격으로 리더의 위치를 파악하고, 필요시 리더 리밸런싱을 통해 리더의 위치가 알맞게 배분됨
4. 명령어
```
cat partitions.json
{
    "partitions":
    [ { "topic": "hello.kafka", "partition": 0, "replicas": [ 0 ] } ]
    ,"version": 1
}

./bin/windows/kafka-reassign-partitions.bat --zookeeper my-kafka:2181 --reassignment-json-file partitions.json --execute
```
<div align="center">
<img src="https://github.com/user-attachments/assets/893d83f3-a699-415c-813f-2aa3d4c63289" />
</div>

-----
### kakfa-delete-record
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/d269f34f-0ef2-4396-8fab-96ce7085a718" />
</div>

  - 0번 파티션의 0 ~ 5번 오프셋의 레코드를 삭제하겠다는 의미

-----
### kafka-dump-log
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/b28b01ca-cab3-455e-a3db-62583d92b794" />
</div>

1. data라는 디렉토리에 데이터를 넣을 때, hello-kafka라는 토픽에 0번 파티션에 해당하는 데이터가 들어가게 됨
2. 인덱스, 로그, 타입 인덱스와 같이 세그먼트 단위로 데이터가 들어감
3. 이러한 데이터, 즉 카프카 프로듀서로 보낸 데이터가 어떤 오프셋으로 저장되어 있는지 여부를 확인하기 위해 kafka-dump-log를 확인하면 됨
   - --files : 특정 파일에 대해서 데이터 지정
   - --deep-iteration : 상세 로그 확인 가능
