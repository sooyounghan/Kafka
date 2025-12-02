-----
### 💡 Broker, Replication, ISR(In-Sync-Replication)
-----
1. 에러가 발생할 때, 카프카의 가용성을 보장하는 가장 좋은 방법이 복제(Replication)
2. Kafka Broker : 카프카가 설치되어 있는 서버 단위
   - 일반적으로 3개 이상의 Broker로 구성하여 사용하는 것을 권장
<img width="1888" height="455" alt="image" src="https://github.com/user-attachments/assets/32524977-f397-45dd-b7cd-d742086555b7" />

   - 만약 파티션이 1개이고, Replication이 1인 토픽이 존재하며, 브로커가 3대 : 브로커 3대 중 1대에 해당 토픽이 저장

3. Replication : Partition의 복제를 의미
   - 만약 이 값이 1이라면, 파티션은 1개만 존재한다는 뜻을 의미
   - 만약 이 값이 2이라면, 원본 1개와 복제본 1개로 총 2개가 존재 (복제본은 Replicated Partition)
   - 만약 이 값이 3이라면, 원본 1개와 복제본 2개로 총 3개가 존재
<img width="1863" height="688" alt="image" src="https://github.com/user-attachments/assets/6f610889-54ce-44e8-b599-7acfb78abceb" />

   - 다만, 브로커 개수에 따라 Replication 개수가 제한
     + 브로커 개수가 3이면, Replication은 4가 될 수 없음
     + 원본 1개 파티션은 Leader Partition이라 칭하며, 나머지 2개 복제본 파티션은 Follower Partition으로, 이 둘을 합쳐서 ISR(In-Sync-Replica)라고 부름
<img width="1777" height="563" alt="image" src="https://github.com/user-attachments/assets/b6598051-ad5d-4123-82a1-3388fce62cdc" />

4. Replication 사용 이유
   - 파티션의 고가용성을 위해 사용
   - 만약 브로커가 3개인 파티션에서 Replication이 1이고, Partition이 1인 Topic이 존재한다고 가정
     + 갑자기, 브로커가 어떠한 이유로 사용할 수 없게 되면, 해당 파티션은 더 이상 복구 불가능
     + 하지만, Replication이 2라면, 브로커 1개가 죽더라도 Follower Parition이 존재하므로 복제본으로 복구 가능 = 즉, Leader Parition으로 역할을 승계

5. Leader Partition과 Follower Patition의 역할
   - Producer가 토픽의 파티션에 데이터를 전달할 때, 전달받는 주체가 바로 Leader Partition
   - Producer에는 ack라는 상세 옵션이 존재 : 이를 통해 고강용성을 유지하는데, Parition의 Replicaiton과 연관이 있음
     + 0과 1, all 옵션 3개 중 1개 설정 가능
     + ack = 0일 경우, Producer는 Leader Partition에 데이터를 전송하고, 응답값을 받지 않으므로, Leader Partition에 데이터가 정상적으로 전송됐는지 나머지 Partition에 복제가 되었는지 보장 불가 (즉, 속도는 빠르지만 데이터 유실 가능성 존재)
     + ack = 1일 경우, Leader Partition에 데이터를 전송하고, 데이터를 정상적으로 받았는지 응답값을 받음 (다만, 나머지 파티션에 복제되었는지는 알 수 없음)
       * 만약, Leader Partition이 데이터를 받은 즉시 브로커가 장애가 난다면, 나머지 파티션에 데이터가 미처 전송되지 못한 상태이므로 ack = 0 옵션과 마찬가지로 데이터 유실가능성 존재
     + ack = all은 1 옵션에 추가로, Follower Parition에도 잘 전송되었는지 응답을 받음
       * 즉, Leader Partition에 데이터를 보낸 후, 나머지 Follower Partition에도 잘 복제가 되었는지 확인하는 절차를 가짐
       * 데이터 유실 문제는 없지만, 확인하는 부분이 많아 속도가 현저히 느림

6. Replication Count
   - Replication의 개수가 많아지면, 그만큼 브로커의 리소스 사용량도 증가
   - 따라서 카프카에 들어오는 데이터량과 Retention Date, 즉 저장 시간을 잘 생각해서 개수를 지정해야 함
   - 3개 이상 브로커를 사용할 때는 Replication 3으로 설정하는 것을 추천
