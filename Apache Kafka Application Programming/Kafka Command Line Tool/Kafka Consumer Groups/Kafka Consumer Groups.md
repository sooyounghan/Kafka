-----
### kafka-consumer-groups
-----
1. hello-group 이름의 컨슈머 그룹으로 생성된 컨슈머로 hello.kafka 토픽의 데이터를 가져갔음
   - 컨슈머 그룹은 따로 생성하는 명령을 날리지 않고, 컨슈머를 동작할 때 컨슈머 그룹 이름을 지정하면 새로 생성
   - 생성된 컨슈머 그룹의 리스트는 kafka-consumer-groups.bat 명령어로 확인 가능
   - 명령어
```
./bin/windows/kafka-consumer-groups.bat --bootstrap-server my-kafka:9092 --list
```
```
./bin/windows/kafka-consumer-groups.bat --bootstrap-server my-kafka:9092 --group hello-group --describe
```
<div align="center">
<img src="https://github.com/user-attachments/assets/1d26f272-4022-4e6d-b076-5db3f218395f" />
</div>

2. --describe 옵션을 사용하면 해당 컨슈머 그룹이 어떤 토픽을 대상으로 레코드를 가져갔는지 상태를 확인할 수 있음
   - 파티션 번호, 현재까지 가져간 레코드의 오프셋, 파티션 마지막 레코드의 오프셋, 컨슈머 랙, 컨슈머 ID, 호스트를 알 수 있으므로 컨슈머의 상태를 조회할 때 유용
   - 명령어
```
./bin/windows/kafka-consumer-groups.bat --bootstrap-server my-kafka:9092 --group hello-group --describe
```
<div align="center">
<img src="https://github.com/user-attachments/assets/b66f3fb6-0192-4aa8-ac19-d75758bb365b" />
</div>

3. kafka-consumer-groups 오프셋 리셋
   - 명령어
```
./bin/windows/kafka-consumer-groups.bat --bootstrap-server my-kafka:9092 --group hello-group --topic hello.kafka --reset-offsets --to-earliest --execute
```
```
./bin/windows/kafka-console-consumer.bat --bootstrap-server my-kafka:9092 --topic hello.kafka --group hello-group
```
<div align="center">
<img src="https://github.com/user-attachments/assets/8c26bfdc-4e70-4d94-847f-1f531ea90f25" />
</div>

4. kafka-consumer-groups 오프셋 리셋 종류
   - --to-earliest : 가장 처음 오프셋(작은 번호)으로 리셋
   - --to-latest : 가장 마지막 오프셋(큰 번호)으로 리셋
   - --to-current : 현 시점 기준 오프셋으로 리셋
   - --to-datetime {YYYY-MM-DDTHH:mmSS.sss} : 특정 일시로 오프셋 리셋(레코드 타임스탬프 기준)
   - --to-offset {long} : 특정 오프셋으로 리셋
   - --shift-by {+/- long} : 현재 컨슈머 오프셋에서 앞/뒤로 옮겨서 리셋
   - 명령어 예시
```
./bin/windows/kafka-consumer-groups.bat --bootstrap-server my-kafka:9092 --group hello-group --topic hello.kafka --reset-offsets --to-earliest --execute
```
<div align="center">
<img src="https://github.com/user-attachments/assets/031216da-7975-4aa3-ba8a-4ac0cd3d1958" />
</div>
