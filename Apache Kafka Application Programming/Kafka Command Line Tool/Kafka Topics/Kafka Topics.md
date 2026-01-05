-----
### kafka-topics
-----
1. hello.kafka 토픽처럼 카프카 클러스터 정보와 토픽 이름 만으로 토픽 생성 가능
2. 클러스터 정보와 토픽 이름은 토픽을 만들기 위한 필수값
   - 만들어진 토픽은 파티션 개수, 복제 개수 등과 같이 다양한 옵션이 포함되어 있지만, 모두 브로커에 설정된 기본값으로 생성
   - 명령어
```
./bin/windows/kafka-topics.bat --create --bootstrap-server my-kafka:9092 --topic hello.kafka
./bin/windows/kafka-topics.bat --bootstrap-server my-kafka:9092 --topic hello.kafka --describe
```
<div align="center">
<img src="https://github.com/user-attachments/assets/3146be59-ccb2-4e5b-9847-b2273a51dcd2" />
</div>

3. 파티션 개수, 복제 개수, 토픽 데이터 유지 기간 옵션들을 지정하여 토픽을 생성하고 싶다면, 다음과 같이 명령을 실행
   - 생성된 토픽들의 이름을 조회하려면 --list 옵션 사용
   - 명령어
```
./bin/windows/kafka-topics.bat --create --bootstrap-server my-kafka:9092 --partitions 10 --replication-factor 1 --topic hello.kafka2 --config retention.ms=172800000
./bin/windows/kafka-topics.bat --bootstrap-server my-kafka:9092 --describe
./bin/windows/kafka-topics.bat --bootstrap-server my-kafka:9092 --topic hello.kafka2 --describe
```
<div align="center">
<img src="https://github.com/user-attachments/assets/5d0a99a5-d57c-40bd-9652-b9da1404b5a3" />
</div>

4. 파티션 개수를 늘리기 위해 --alter 옵션을 사용
    - 명령어
 ```
./bin/windows/kafka-topics.bat --create --bootstrap-server my-kafka:9092 --topic test
./bin/windows/kafka-topics.bat --bootstrap-server my-kafka:9092 --topic test --describe
./bin/windows/kafka-topics.bat --bootstrap-server my-kafka:9092 --topic test --alter --partitions 10
./bin/windows/kafka-topics.bat --bootstrap-server my-kafka:9092 --topic test --describe
```
<div align="center">
<img src="https://github.com/user-attachments/assets/703a203d-5f84-433f-bd03-8f76fd685ed5" />
</div>

5. 파티션 개수를 늘릴 수 있지만, 줄일 수는 없음
   - 다시 줄이는 명령을 내리면 InvaildPartitionsException 발생
   - 분산 시스템에서 이미 분산된 데이터를 줄이는 방법은 매우 복잡
   - 삭제 대상 파티션을 지정해야할 뿐만 아니라 기존에 저장되어 있던 레코드를 분산하여 저장하는 로직이 필요하기 때문임 : 이 떄문에 카프카에서는 파티션을 줄이는 로직을 제공하지 않음
   - 만약 피치 못할 사정으로 파티션 개수를 줄여야 할 때는 토픽을 새로 만드는 편이 좋음
<div align="center">
<img src="https://github.com/user-attachments/assets/567a6c42-954d-476f-808e-48d72b397cea" />
</div>
