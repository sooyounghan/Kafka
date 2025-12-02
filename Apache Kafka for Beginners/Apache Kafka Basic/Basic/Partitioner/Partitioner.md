-----
### 파티셔너 (Partitioner)
-----
1. 프로듀서가 토픽으로 데이터를 보내면, 무조건 파티셔너를 통해서 브로커로 데이터가 전송
<div align="center">
<img src="https://github.com/user-attachments/assets/ce26fe93-4e3b-47e3-b797-b70b1e7e6bec">
</div>

   - 즉, 파티셔너는 데이터를 토픽에 어떤 파티션에 넣을지 결정하는 역할을 함
   - 레코드에 포함된 메세지 키 또는 메세지 값에 따라서 파티션의 위치가 결정

2. 파티셔너를 따로 설정하지 않으면, UniformStickyPartitioner로 설정
3. 파티셔너는 메세지 키가 있을 때와 없을 때와 다르게 동작
<div align="center">
<img src="https://github.com/user-attachments/assets/eac960a9-c657-42a0-a634-4d42663a82cc">
</div>

<div align="center">
<img src="https://github.com/user-attachments/assets/bd718467-418d-4965-a9a8-cf185ff5e19e">
</div>

   - 메세지 키가 있는 경우 : 레코드는 파티셔너에 의해 해시값을 기준으로 어느 파티션으로 들어갈지 결정
     + 토픽에 파티션이 2개 있는 경우 : 파티셔너의 해시 로직에 의해 서울은 파티션 0번, 부산은 파티션 1번, 울산은 파티션 0번으로 들어갈 수 있음
     + 동일한 메세지 키를 가진 레코드는 동일한 해시값을 만드므로, 항상 동일한 파티션에 들어가는 것을 보장 : 따라서, 순서를 지켜서 데이터를 처리 가능 (파티션 한 개 내부에서는 큐처럼 동작하므로, 순서를 지킬 수 있음)

  - 메세지 키가 없는 경우 : 라운드 로빈(Round Robin)으로 파티션에 들어가게 됨
    + 전통적인 라운드 로빈 방식이 아닌, UniformStickyPartitioner는 프로듀서에서 배치로 모을 수 있는 최대한의 레코드를 모아서 파티션으로 데이터를 보내게 됨
    + 이렇게 배치 단위로 데이터를 보낼 때, 파티션에 라운드 로빈 방식으로 돌아가면서 데이터를 넣음

4. 직접 개발한 파티셔너도 프로듀서에서 설정 가능 : Custom Partitioner를 만들 수 있도록, Partitioner 인터페이스를 제공하고 있음
   - Custom Partitioner 클래스를 만들면, 메시지 키 또는 값 또는 토픽 이름에 따라 어느 파티션에 데이터를 보낼 것인지 결정 가능
   - 사용 시기 : 예) VIP 고객을 위해 데이터 처리를 좀 더 빠르게 처리하는 로직 (10개의 파티션 중 8개는 VIP 고객, 2개는 일반 고객 등으로 편성)
   
