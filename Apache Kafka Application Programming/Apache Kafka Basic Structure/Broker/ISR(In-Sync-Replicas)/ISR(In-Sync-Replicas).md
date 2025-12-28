-----
### ISR(In-Sync-Replicas)
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/eea188f4-713c-4b31-8c6a-92ece9ba2902" />
</div>

1. 리더 파티션과 팔로워 파티션이 모두 싱크(오프셋 개수가 같음)가 된 상태
2. 복제 개수가 2인 토픽을 가정
   - 이 토픽에는 리더 파티션 1개와 팔로워 파티션 1개가 존재할 것
   - 리더 파티션에 0부터 3의 오프셋이 있다고 가정할 때, 팔로워 파티션에 동기화가 완료되려면 0부터 3까지 오프셋이 존재해야 함
3. 동기화가 완료됐다는 의미는 리더 파티션의 모든 데이터가 팔로워 파티션에 복제된 상태를 말함

-----
### unclean.leader.election.enable
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/efdb10cb-ae03-490b-a40f-d08336fcd3c6" />
</div>

1. 리더 파티션의 데이터를 모두 복제하지 못한 상태이고, 이렇게 싱크가 되지 않은 팔로워 파티션이 리더 파티션으로 선출되면 데이터 유실 가능성 존재
2. 유실이 발생하더라도 서비스를 중단하지 않고, 지속적으로 토픽을 사용하고 싶다면 ISR이 아닌 팔로워 파티션을 리더로 선출하도록 설정 가능
   - unclean.leader.election.enable=true : 유실 감수 (복제가 안 된 팔로워 파티션을 리더로 승급)
   - unclean.leader.election.enable=false : 유실을 감수하지 않음 (해당 브로커가 복구될 때 까지 중단)
   
