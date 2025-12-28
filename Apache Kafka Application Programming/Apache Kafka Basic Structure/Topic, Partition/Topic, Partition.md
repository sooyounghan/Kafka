-----
### 토픽과 파티션
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/60f29b79-2d8a-46d3-9039-bee8a59b786d" />
</div>

1. 토픽은 카프카에서 데이터를 구분하기 위해 사용하는 단위
2. 토픽은 1개 이상 파티션을 소유하고 있음
   - 파티션에는 프로듀서가 보낸 데이터들이 들어가 저장되는데, 이 데이터를 레코드(Record)라고 부름
   - 파티션은 자료구조에서 접하는 큐(Queue)와 비슷한 구조 : First-In First-Out(FIFO) 구조와 같이 먼저 들어간 레코드는 컨슈머가 먼저 가게 됨
   - 다만, 일반적인 자료구조로 사용되는 큐는 데이터를 가져가면(Pop) 삭제되지만, 카프카에서는 삭제하지 않음 : 파티션의 레코드는 컨슈머가 가져가는 것과 별개로 관리
3. 이러한 특징 때문에 토픽의 레코드는 다양한 목적을 가진 여러 컨슈머 그룹들이 토픽의 데이터를 여러번 가져갈 수 있음

-----
### 토픽 생성 시 파티션이 배치되는 방법
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/1ba78f2b-a0e7-4f87-8245-649c2ffc08d4" />
</div>

1. 파티션이 5개인 토픽을 생성했을 경우 그림과 같이 0번 브로커에서부터 시작하여 Round-Robin 방식으로 리터 파티션들이 생성
2. 카프카 클라이언트는 리더 파티션이 있는 브로커와 통신하여 데이터를 주고 받으므로 여러 브로커에 골고루에 네트워크 통신을 하게 됨
3. 이를 통해, 데이터가 특정 서버(여기서는 브로커)와 통신이 집중되는(Hot Spot) 현상을 막고 선형 확장(Linear Scale-Out)을 하여 데이터가 많아지더라도 자연스럽게 대응할 수 있게 됨
<div align="center">
<img src="https://github.com/user-attachments/assets/1d745758-85db-42c3-90cf-59b29216c72b" />
</div>

4. 특정 브로커에 파티션이 쏠린 현상
<div align="center">
<img src="https://github.com/user-attachments/assets/4946b900-53b5-4867-bd2d-0274b26900b5" />
<img src="https://github.com/user-attachments/assets/73ed70ea-13dd-4fb5-98b1-1d0039ba22cb" />
</div>

   - 특정 브로커에 파티션이 몰리는 경우 kafka-reassign-partition.sh 명령으로 파티션 재분배 할 수 있음

-----
### 파티션 개수와 컨슈머 개수와 처리량
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/3738e92b-345e-4536-b21e-6b1ccee7e36b" />
</div>

1. 기본적으로 파티션과 컨슈머의 관계는 1:1 관계 (최대 파티션은 최대 1개의 컨슈머만 가질 수 있음)
2. 파티션은 카프카의 병렬처리의 핵심으로써 그룹으로 묶인 컨슈머들이 레코드를 병렬로 처리할 수 있도록 매칭
3. 컨슈머의 처리량이 한정된 상황에서 많은 레코드를 병렬로 처리하는 가장 좋은 방법은 컨슈머의 개수를 늘려 Scale-Out하는 것
4. 컨슈머 개수를 늘림과 동시에 파티션 개수도 늘리면 처리량이 증가하는 효과를 볼 수 있음
5. 하지만 파티션 개수를 줄이는 것은 불가능
<div align="center">
<img src="https://github.com/user-attachments/assets/96b29b95-ba6b-44c4-9037-b2c385a7fcaa" />
</div>

   - 카프카에서 파티션 개수를 줄이는 것은 지원하지 않음
   - 그러므로 파티션을 늘리는 작업을 할 때는 신중히 파티션 개수를 정해야 함
   - 한 번 늘리면 줄이는 것은 불가능하므로 토픽을 삭제하고 재생성하는 방법 외에는 없음
   - 카프카에서는 파티션의 데이터를 세그먼트로 저장하고 있으며, 만에 하나 지원을 한다고 하더라도 여러 브로커에 저장된 데이터를 취합하고 정렬해야하는 복잡한 과정을 거쳐야 하므로 클러스터에 큰 영향을 미침
   - KIP-694에서 파티션 개수를 줄이는 것을 논의했지만, 더 이상 진행되고 있지 않음

