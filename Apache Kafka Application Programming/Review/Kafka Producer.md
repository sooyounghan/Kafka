-----
### 카프카 프로듀서
-----
1. Kafka Producer : 카프카 브로커로 데이터를 전달하는 역할을 하는 애플리케이션
2. Partitioner : 메세지 키를 토대로 파티션을 지정하는 Class로, 커스텀 클래스를 사용하여 로직 변경 가능
3. Accumualator : 레코드 전송 시 배치로 묶는 역할
4. acks : 레코드를 카프카 클러스터로 전송 시 전달 신뢰성 지정
5. min.insync.replicas : acks=all일 경우, 최소 적재 보장 파티션 개수
   - 💡 acks=all(-1), Replication Factor=3, min.insync.replicas=2로 설정하는 것이 가장 안정적인 데이터 처리
6. enable.idempotence : 멱등성 프로듀서로 동작하기 위해 설정하는 옵션
7. transactional.id : 트랜잭션 프로듀서로 동작하기 위해 설정하는 옵션
    - UUID로 지정하게 되면, 이를 기반으로 특정 레코드들에 대해 원자 단위로 하나로 묶어서 데이터를 보내고, 적재
    - 컨슈머 입장에서도 하나의 트랜잭션으로 묶인 레코드로 한 번에 처리하고 싶을 때 유용하게 처리 가능 
