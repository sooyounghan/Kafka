-----
### 프로듀서 주요 옵션
-----
1. 필수 옵션 (default 값이 없으므로, 값을 반드시 지정해야 함 [설정하지 않으면 실행 불가])
   - bootstrap.servers : 프로듀서가 데이터를 전송할 대상 카프카 클러스터에 속한 브로커의 호스트 이름:포트를 1개 이상 작성
     + 2개 이상 브로커 정보를 입력하여 일부 브로커에 이슈가 발생하더라도 접속하는 데 이슈가 없도록 설정이 가능
   - key.serializer : 레코드의 메세지 키를 직렬화하는 클래스를 지정
   - value.serializer : 레코드의 메세지 값을 직렬화하는 클래스를 지정

2. 선택 옵션
   - acks : 프로듀서가 전송한 데이터가 브로커들에 정상적으로 저장되었는지 전송 성공 여부를 확인하는 데 사용하는 옵션
     + 0, 1, -1(all) 중 하나로 설정할 수 있음
     + 기본값은 1
   - linger.ms : 배치를 전송하기 전까지 기다리는 최소 시간 (기본값은 0)
   - retries : 브로커로부터 에러를 받고 난 뒤 재전송을 시도하는 횟수 지정 (기본값은 2147483647)
   - max.in.flight.requests.per.connection : 한 번에 요청하는 최대 커넥션 개수로, 설정된 값 만큼 동시에 전달 요청 수행 (기본값은 5)
   - partitioner.class : 레코드를 파티션에 전송할 때 적용하는 파티셔너 클래스를 지정 (기본값은 org.apache.kafka.clients.producer.internals.DefaultPartitioner)
   - enable.idempotence : 멱등성 프로듀서로 동작할지 여부 설정 (기본값은 false)
   - transactional.id : 프로듀서가 레코드를 전송할 때 레코드를 트랜잭션 단위로 묶을지 여부 설정 (기본값은 null)
