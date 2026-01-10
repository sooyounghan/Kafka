-----
### 카프카 스트림즈
-----
1. Kafka Streams : 상태 / 비상태 기반 스트림 데이터 처리를 수행하는 애플리케이션
2. Task : Streams Application 내부에서 생성되어 로직을 수행하는 최소 단위
3. Processor : 데이터를 가져오거나, 처리하거나, 내보내는 노드
4. Stream : 프로세서로부터 처리된 데이터를 다른 프로세스로 전달되는 레코드
5. Streams DSL : 추상화되어 Stream Procssing에 필요한 메서드들을 정의한 메서드들의 모음
6. Processor API : Streams DSL에서 구현할 수 없는 로직을 구현할 때 사용하는 API
7. KStream : 레코드의 흐름 (Consumer의 poll()과 유사)
8. KTable : 특정 파티션의 메세지 키를 기준으로 가장 최근의 레코드들의 묶음
9. GlobalKTable : 모든 파티션의 메세지 키를 기준으로 가장 최근의 레코드들의 묶음
10. Co-Paritioning : 동일한 파티션 개수, 동일한 파티셔닝 전략을 통해 레코드가 저장된 서로 다른 2개의 토픽
