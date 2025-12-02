-----
### 컨슈머 랙 (Consumer Lag)
-----
1. 카프카를 운영함에 있어 아주 중요한 모니터링 지표 중 하나
2. 파티션에 데이터가 저장되면, 각 데이터는 Offset이라는 숫자가 붙게 됨
   - 파티션이 1개인 토픽에 프로듀서가 데이터를 넣으면, 0부터 차례대로 숫자가 매겨짐
   - 프로듀서는 계속해서 데이터를 넣게 되고, 컨슈머는 데이터를 가져감
<div align="center">
<img src="https://github.com/user-attachments/assets/f437b4ce-f10d-410d-a1d9-6dcb39e99f81">
</div>

   - 만약, 프로듀서가 데이터를 넣는 속도가 컨슈머가 가져가는 속도보다 빠를 경우
     + 프로듀서가 넣은 데이터의 오프셋, 컨슈머가 가져간 데이터의 오프셋 간의 차이가 발생 하는데 이것이 바로 Consumer Lag
     + Lag은 적을 수도 있고, 많을 수도 있는데, 이 숫자를 통해 현재 해당 토픽에 대해 파이프라인으로 연계되어 있는 프로듀서와 컨슈머의 상태 유추 가능 : 주로 컨슈머 상태에 대해 볼 때 사용
     + Lag은 각 파티션을 기준으로 프로듀서가 넣은 데이터 오프셋과 컨슈머가 가져가는 데이터의 오프셋의 차이를 기반으로 함
     + 따라서, 토픽에 여러 파티션이 존재할 경우, Lag은 여러 개 존재 가능
<div align="center">
<img src="https://github.com/user-attachments/assets/c87b9c41-9d47-44c9-b286-7d013a1f7530">
</div>

  - 만약, 컨슈머 그룹이 1개이고, 파티션이 2개인 토픽에서 데이터를 가져가면 Lag은 2개 측정될 수 있음
  - 이처럼 한개의 토픽과 컨슈머 그룹에 대한 Lag이 여러 개 존재할 수 있을 때, 그 중 가장 높은 숫자의 Lag : records-lag-max
<div align="center">
<img src="https://github.com/user-attachments/assets/117d1a9e-c420-4c9c-9997-f0dbb21aac4f">
</div>

