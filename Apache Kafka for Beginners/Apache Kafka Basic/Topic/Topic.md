-----
### 토픽 (Topic)
-----
1. 카프카에는 다양한 데이터가 저장 가능한데, 데이터가 들어가는 공간을 의미
2. 카프카는 토픽을 여러 개 생성 가능
<div align="center">
<img src="https://github.com/user-attachments/assets/86579928-26ae-480f-bde5-ebbb82f2b0f1">
</div>

   - 토픽은 데이터베이스의 테이블이나 파일 시스템의 폴더와 유사한 성질을 가지고 있음
   - 토픽에 Kafka Producer가 데이터를 넣게 되고, Kafka Consumer가 데이터를 가져오는 역할
   - 토픽은 이름을 가질 수 있는데, 목적에 따라 명명 가능 (무슨 데이터를 담는지 명명하면, 추후 유지보수 시 편리하게 관리 가능)

3. 토픽 내부
<div align="center">
<img src="https://github.com/user-attachments/assets/b79b4ce0-fafb-45a1-bc18-5296e736a4b6">
</div>

   - 하나의 토픽은 여러 개의 파티션으로 구성될 수 있으며, 첫 번째 파티션 번호는 0번부터 시작
   - 하나의 파티션은 큐와 같이 내부의 데이터가 파티션 끝에서부터 쌓이게 됨
   - Kakfa Consumer가 붙게 되면, 데이터를 가장 오래된 순서대로 가져오게 됨
     + 만약, 더 이상 데이터가 들어오지 않고, 가져올 데이터가 없다면 Consumer는 다른 데이터가 들어올 때까지 대기함
     + Consumer가 토픽 내부의 파티션에서 데이터(Record)들을 가져가더라도, 데이터는 삭제되지 않음 (파티션에 그대로 보존)
     + 파티션에 보존되는 데이터는 새로운 Consumer가 연결될 때, 다시 0번부터 가져가서 사용 가능 : 다만, Consumer 그룹이 달라야 하며, auto.offset.reset=earliest로 설정되어야 함
     + 즉, 동일 데이터에 대해 두 번 처리할 수 있음
<div align="center">
<img src="https://github.com/user-attachments/assets/809eced7-e3e7-4eea-81e9-d1744e96aafe">
</div>

4. 파티션이 2개 이상인 경우
   - 데이터 7이 토픽에 저장되어야 하는데, 이 때는 프로듀서가 데이터를 보낼 때 키를 지정 가능
     + 키가 null이고, 기본 파티셔너를 사용할 경우 : 라운드 로빈(Round-Robin)으로 할당
     + 키가 있고, 기본 파티셔너를 사용할 경우 : 키의 해시(Hash) 값을 구하고, 특정 파티션에 할당
<div align="center">
<img src="https://github.com/user-attachments/assets/87f3160a-2d54-4402-9860-4f85e8269de7">
</div>

   - 여기서는 키가 없다고 가정하므로, 라운드 로빈(Robin-Robin) 방식으로 데이터가 파티션에 나눠져서 들어가게 됨

5. 파티션이 3개가 되는 경우
<div align="center">
<img src="https://github.com/user-attachments/assets/b46bb0cf-9cc7-45a9-9e2f-0a92a0a14fa9">
</div>

6. 하지만, 파티션을 늘리는 것은 아주 조심해야 함
   - 파티션을 늘리는 것은 가능하지만, 다시 줄일 수 없음
   - 늘리는 이유는 컨슈머의 개수를 늘려서, 데이터를 분산 처리할 수 있음

7. 파티션의 데이터(Record)의 삭제
   - 옵션에 따라 다름 : 레코드에 저장되는 최대 시간과 크기를 지정할 수 있음
     + log.retention.ms : 최대 Record 보존 시간
     + log.retention.byte : 최대 Record 보존 크기(Byte)
   - 즉, 이를 지정하면 일정 기간 또는 용량 동안 데이터를 저장할 수 있게 되고, 적절하게 데이터가 삭제될 수 있도록 설정 가능
  
   
