------
### KStream
------
<div align="center">
<img src="https://github.com/user-attachments/assets/ca58f4ff-bb91-4b09-ab99-f1a9443acd08" />
</div>

1. 레코드의 흐름을 표현한 것으로, 메세지 키와 메세지 값으로 구성
2. KStream으로 데이터를 조회하면 토픽에 존재하는(또는 KStream으로 존재하는) 모든 레코드가 출력
3. KStream은 컨슈머로 토픽을 구독하는 것과 동일한 선상에서 사용하는 것이라고 볼 수 있음

-----
### KTable
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/a0065e12-42ca-4b52-b65b-ff70abaf5411" />
</div>

1. KStream과 다르게 메세지 키를 기준으로 묶어서 사용
2. KStream은 토픽의 모든 레코드를 조회할 수 있지만, KTable은 유니크한 메세지 키를 기준으로 가장 최신 레코드를 사용
3. 그러므로 KTable로 데이터를 조회하면, 메세지 키를 기준으로 가장 최신에 추가된 레코드의 데이터가 출력됨
4. 새로 데이터를 적재할 때, 동일한 메세지 키가 있을 경우 데이터가 업데이트 되었다고 볼 수 있음 : 메세지 키의 가장 최신 레코드가 추가되었기 때문임

-----
### 코파티셔닝
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/b6162b5c-2641-4c0f-a3c0-4c626f08a294" />
</div>

1. KStream과 KTable 데이터가 조인한다고 가정 : 둘이 조인하려면 반드시 코파티셔닝(Co-Patitioning)되어 있어야 함
2. 코파티셔닝이란 조인을 하는 2개 데이터의 파티션 개수가 동일하고 파티셔닝 전략(Partitioning Strategy)을 동일하게 맞추는 작업
3. 파티션 개수가 동일하고 파티셔닝 전략이 같은 경우에는 동일한 메세지 키를 가진 데이터가 동일한 태스크에 들어가는 것을 보장
4. 이를 통해 각 태스크는 KStream의 레코드와 KTable 메세지 키가 동일할 경우 조인을 수행할 수 있음
5. 코파티셔닝되지 않은 2개 토픽의 이슈
<div align="center">
<img src="https://github.com/user-attachments/assets/cc42a1ed-3615-47c5-8e30-52fb698aba83" />
</div>

  - 문제는 조인을 수행하려는 토픽들이 코파티셔닝되어 있음을 보장할 수 없다는 것
  - KStream과 KTable로 사용하는 2개의 토픽이 파티션 개수가 다를 수도 있고 파티션 전략이 다를 수 있음
  - 이런 경우 조인을 수행할 수 없으며, 코파티셔닝이 되지 않은 2개의 토픽을 조인하는 로직이 담긴 스트림즈 애플리케이션을 실행하면 TopologyException이 발생

-----
### GlobalKTable
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/9b166b39-3361-49a8-ae40-5168a1dc5aae" />
</div>

1. 코파티셔닝되지 않은 KStream와 KTable을 조인해서 사용하고 싶다면, KTable을 GlobalKTable로 선언하여 사용하면 됨
2. GlobalKTable은 코파티셔닝 되지 않은 KStream과 데이터 조인을 할 수 있음
   - KTable과 다르게 GlobalKTable로 정의된 데이터는 스트림즈 애플리케이션의 모든 태스크에 동일하게 공유되어 사용되기 떄문임
