-----
### 프로듀서 파티셔너
-----
1. 프로듀서의 기본 파티셔너
   - 프로듀서 API를 사용하면 'UniformStickyPartitioner'와 'RoundRobinPartitioner' 2개 파티셔너를 제공
   - 카프카 클라이언트 라이브러리 2.5.0 버전에서 파티셔너를 지정하지 않은 경우 UniformStickyPartitioner가 파티셔너로 기본 설정
   - 메세지 키가 있을 경우 동작
     + UniformStickyPartitioner와 RoundRobinPartitioner 둘 다 메세지 키가 있을 때는 메세지 키의 해시값과 파티션을 매칭하여 레코드를 전송
     + 동일한 메세지 키가 존재하는 레코드는 동일한 파티션 번호에 전달됨
     + 만약 파티션 개수가 변경될 경우, 메세지 키와 파티션 번호 매칭은 꺠지게 됨

   - 메세지 키가 없을 때 동작
     + 메세지 키가 없을 때 파티션에 최대한 동일하게 분배하는 로직이 있는데, UniformStickyPartitioner는 RoundRobinPartitioner의 단점을 개선하였다는 점이 다름

   - RoundRobinPartitioner
     + ProducerRecord가 들어오는 대로 파티션을 순회하면서 전송
     + Accumulator에서 묶이는 정도가 적으므로 전송 성능이 낮음

   - UniformStickyPartitioner
     + Accumulator에서 레코드들이 배치로 묶일 때까지 기다렸다가 전송
     + 배치로 묶일 뿐, 결국 파티션을 순회하면서 보내기 때문에 모든 파티션에 분배되어 전송
     + RoundRobinPartitioner에 비해 향상된 성능을 가짐
      
2. 프로듀서의 커스텀 파티셔너
   - 카프카 클라이언트 라이브러리에서는 사용자 지정 파티셔너를 생성하기 위한 Partitioner 인터페이스를 제공
   - Partitioner 인터페이스를 상속받은 사용자 정의 클래스에서 메세지 키 또는 메세지 값에 따른 파티션 지정 로직을 적용할 수 있음
   - 파티셔너를 통해 파티션이 지정된 데이터는 Accumulator에 버퍼로 쌓임
   - Sender(센더) 스레드는 Accumulator에 쌓인 배치 데이터를 가져가 카프카 브로커로 전송
