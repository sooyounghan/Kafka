-----
### Assignor
-----
1. 컨슈머와 파티션 할당 정책은 컨슈머의 Assignor에 의해 결정
2. 카프카에서는 RangeAssginor, RoundRobinAssignor, StickyAssignor를 제공
3. 카프카 2.5.0는 RangeAssignor가 기본값으로 설정
   - RangeAssginor : 각 토픽에서 파티션을 숫자로 정렬, 컨슈머를 사전 순서로 정렬하여 할당
   - RoundRobinAssignor : 모든 파티션을 컨슈머에서 번갈아가면서 할당
   - StickyAssignor : 최대한 파티션을 균등하게 배분하면서 할당
