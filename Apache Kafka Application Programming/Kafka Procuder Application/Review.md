-----
### 정리
-----
1. 프로듀서는 데이터를 카프카 클러스토로 보냄
2. 프로듀서는 Partitioner, Accumulator, Sender를 지나 카프카 클러스터로 데이터가 보내짐
3. 프로듀서의 기본 파티셔너는 UniformStickyPartitioner와 RoundRobinPartitioner가 있음
   - 메세지 키가 있을 경우 UniformStickyPartitioner와 RoundRobinPartitioner의 동작은 동일
   - UniformStickyPartitioner은 RoundRobinPartitioner을 개선한 파티셔너

4. ISR은 리더 파티션과 팔로워 파티션 레코드가 모두 동일한 개수로 동기화된 완료된 묶음을 뜻함
5. acks는 0, 1, all(-1)로 설정할 수 있으며, 데이터 저장 신뢰도를 정할 때 사용
6. min.insync.replicas는 acks가 all(-1)일 때 유효한 설정

-----
### 퀴즈
-----
1. 프로듀서에서 데이터를 전송할 때 반드시 배치로 묶어 전송함 (X)
   - 레코드를 만들고, send()를 호출하면, 파티셔너와 Accumulator를 지나면서, 상황에 따라 배치로 반드시 묶이는 것은 아님

2. 메세지 키를 지정하지 않으면 RoundRobinPartitioner로 파티셔너가 지정됨 (X)
   - 메세지 키를 지정 여부에 따라 파티셔너가 지정되는 것이 아님
   - 2.5.0버전의 경우에는 UniformStickyPartitioner가 DefaultPartitioner

3. ISR은 복제 개수가 2 이상일 경우에만 존재함 (X)
   - ISR은 1인 경우에도 존재

4. acks가 all(-1)일 경우, 데이터 전송 속도가 가장 빠름 (X)
   - acks=0, 1의 경우 데이터 전송 속도를 높일 수 있음

5. min.insync.replicas=3, 복제 개수가 2일 때 가장 신뢰도 높게 데이터를 전송할 수 있음 (X)
   - min.insync.replicas=3 : 리더 파티션 1개, 팔로워 파티션 2개
   - 복제 개수가 2개 이므로, 전제 조건이 틀림
   - acks=all, min.insync.replicas=2, replicas factor = 3로 하는 것이 가장 신뢰도 높게 데이터 전송 가능
