-----
### kafka-configs
-----
1. 토픽의 일부 옵션을 설정하기 위해서는 kafka-configs.bat 명령어를 사용해야 함
   - --alter과 --add-config 옵션을 사용하여 min.insync.replicas 옵션을 토픽별로 설정할 수 있음
   - 명령어
```
./bin/windows/kafka-topics.bat --bootstrap-server my-kafka:9092 --topic test --describe
./bin/windows/kafka-configs.bat --bootstrap-server my-kafka:9092 --alter --add-config min.insync.replicas=2 --topic test
```
   - min.insync.replicas : 프로듀서로 데이터를 보낼 때, 그리고 컨슈머가 데이터를 읽을 때 워터마크 용도로 사용되고, 얼마나 데이터를 좀 더 안전하게 데이터를 보내야 하는지에 대해서도 명확하게 설정할 때 활용됨
     + 이 옵션을 설정 또는 수정하기 위해서 kafka-conifgs.bat 옵션을 사용해야 함
   - kafka-configs.bat와 --alter, --add-config를 통해서 기존에 생성되어 있는 테스트라고 하는 토픽에 대해 min.insync.replicas=2로 설정
   - 설정이 완료되면 Completing Updating가 출력되며, test라는 토픽에 대해 --describe를 하게 되면, 옵션이 다음과 같이 정상적으로 설정됨을 알 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/5103f5a3-d361-401a-a7bd-0de7060cbb2f" />
</div>

2. 브로커에 설정된 각종 기본값은 --broker, --all, --describe 옵션을 사용하여 조회 가능
   - server.properties 파일을 직접 확인하지 않더라도 브로커에 설정된 각종 기본값을 --broker --all --describe 옵션을 통해 조회 가능
   - 이러한 옵션을 통해 실질적으로 필요한 파티션이 현재 몇 개로 기본 설정이 되어있는지, 그리고 나머지 네트워크 설정 같은 것들도 함께 활용할 때 그리고 확인할 때 다 같이 조회해서 사용 가능
   - 명령어
```
./bin/windows/kafka-configs.bat --bootstrap-server my-kafka:9092 --broker 0 --all -describe
```

<div align="center">
<img src="https://github.com/user-attachments/assets/c2d84438-a046-496e-b690-1d2a6805e7e7" />
</div>
