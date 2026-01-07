-----
### 정리
-----
1. 카프카를 실행하려면 바이너리 파일을 다운로드 받아야 함
2. 카프카 브로커와 커맨드 라인 툴 버전을 맞춰 실행해야 함
3. kafka-topics.sh/bat를 사용하면 토픽을 생성, 수정, 삭제할 수 있음
4. kafka-console-producer.sh/bat를 사용하면 토픽에 데이터를 보낼 수 있음
5. kafka-console-consumer.sh/bat를 사용하면 토픽의 데이터를 확인할 수 있음
6. kafka-consumer.groups.sh/bat를 사용하면 컨슈머 그룹을 조회, 수정할 수 있음

-----
### 퀴즈
-----
1. 토픽을 생성하기 위해서는 반드시 kafka-topics.sh를 사용해야 함 (X)

2. kafka-topic.sh를 사용하여 토픽을 생성하면 복제 개수는 항상 1로 설정 (X)
   - replicas-factor 또는 브로커 옵션으로 개수 설정 가능

3. kafka-console-producer.sh를 사용하여 메세지 키와 메세지 값이 담긴 레코드를 전송할 수 있음 (O)

4. kafka-console.consumer.sh를 사용하면 오프셋을 확인할 수 있음 (X)
   - Message Key와 Value 값 확인 가능
