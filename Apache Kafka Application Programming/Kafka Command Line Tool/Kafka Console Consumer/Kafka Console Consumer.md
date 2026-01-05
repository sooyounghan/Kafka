-----
### kafka-console-consumer
-----
1. hello.kafka 토픽으로 전송한 데이터는 kafka-console-consumer.bat 명령어로 확인 가능
   - 이 때, 필수 옵션으로 --bootstrap-server에 카프카 클러스터 정보, --topic에 토픽 이름이 필요
   - 추가로, --from-beginning 옵션을 주면 토픽에 저장된 가장 처음 데이터부터 출력
   - 명령어
```
./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic hello.kafka --from-beginning
```
<div align="center">
<img src="https://github.com/user-attachments/assets/df0fb6c4-bd67-4fd1-809d-ea36a4b5904f" />
</div>

2. kafka-console-producer.bat로 보낸 메세지 값이 출력된 것 확인 가능
   - 만약, 레코드의 메세지 키와 메세지 값을 확인하고 싶다면 --property 옵션을 사용하면 됨
   - 명령어
```
./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic hello.kafka --property print.key=true --property key.separator="-" --from-beginning
```
<div align="center">
<img src="https://github.com/user-attachments/assets/2ef813f8-efcb-44ca-8ecf-42cc6dda52fd" />
</div>

   - --max-messages 옵션을 사용하면 최대 Consume 메세지 개수를 설정할 수 있음
   - 명령어
```
./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic hello.kafka --from-beginning --max-messages 1
```
<div align="center">
<img src="https://github.com/user-attachments/assets/fd19dc98-1383-4191-b949-4b70817ec41f" />
</div>

   - --partition 옵션을 사용하면 특정 파티션만 Consume 가능
```
./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic hello.kafka --partition 0 --from-beginning
```
<div align="center">
<img src="https://github.com/user-attachments/assets/57e0b9ca-2cb4-43ca-8334-5af73a7bf2a0" />
</div>

   - --group 옵션을 사용하면 컨슈머 그룹을 기반으로 kafka-console-consumer가 동작
     + 컨슈머 그룹 : 특정 목적을 가진 컨슈머들을 묶음으로 사용하는 것을 뜻함
     + 컨슈머 그룹으로 토픽의 레코드를 가져갈 경우, 어느 레코드까지 읽었는지에 대한 데이터가 카프카 브로커에 저장
```
./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic hello.kafka --group hello-group --from-beginning
```
<div align="center">
<img src="https://github.com/user-attachments/assets/a13292c6-a066-4836-9820-ac1367b1b529" />
</div>
