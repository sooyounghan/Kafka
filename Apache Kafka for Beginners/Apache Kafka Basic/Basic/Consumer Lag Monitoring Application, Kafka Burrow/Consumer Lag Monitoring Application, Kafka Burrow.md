-----
### 컨슈머 랙 모니터링 애플리케이션, 카프카 버로우 (Consumer Lag Monitoring Application, Kafka Burrow)
-----
1. Consumer Lag을 모니터링 하기 위한 오픈 소스 : Burrow
2. Kafka-client 라이브러리를 사용해 자바와 Scalar와 같은 언어를 통해 카프카 컨슈머를 구현할 수 있음
   - 이 때 구현한 KafkaConsumer 객체를 통해 현재 Lag 정보를 가져올 수 있음
   - 만약 Lag을 실시간 모니터링하고 싶다면, 데이터를 ElasticSearch나 InfulxDB와 같은 저장소에 넣은 뒤, Grafana 대시보드를 통해 확인 가능
<div align="center">
<img src="https://github.com/user-attachments/assets/aa4c7e7b-791d-43a7-9dc0-237cc21ba700">
</div>

3. 하지만, Consumer 단위에서 Lag을 모니터링하는 것은 위험하고 운영 요소가 많이 발생
   - 컨슈머 로직단에서 Lag을 수집하는 것은 컨슈머 상태에 Dependency가 발생 : 즉, 컨슈머가 비정상적으로 종료되게 된다면 더 이상 컨슈머는 Lag 정보를 보낼 수 없으므로, 측정할 수 없고, 추가적으로 컨슈머가 개발될 때마다 해당 컨슈머에 Lag 정보를 특정 저장소에 저장할 수 있도록 개발해야 함
   - 만약, Consumer Lag을 수집할 수 없는 컨슈머라면, Lag을 모니터링 할 수 없으므로 까다로워짐
   - 따라서, Linked-In에서 이를 위해 카프카의 Consumer Lag을 효과적으로 모니터링할 수 있도록 Burrow를 개발

4. Burrow
   - Golang으로 작성된 오픈소스
   - Conumser Lag 모니터링을 도와주는 독립적인 애플리케이션
   - 3가지 특징
     + 멀티 카프카 클러스터 지원 : 카프카 클러스터가 여러 개더라도 Burrow Application 1개만 실행해서 연동한다면, 카프카 클러스터들에 있는 컨슈머의 랙을 모두 모니터링 가능
     + Sliding Window를 통한 Consumer의 Status 확인 : Sliding Window를 통해서 Consumer의 Status를 ERROR, WARNING, OK로 표현할 수 있음
       * 만약, 일시적으로 데이터 양이 많아지면서 Consumer Offest이 증가되고 있으면, WARNING으로 지정
       * 만약, 데이터 양은 많아지지만 Conumser가 데이터를 가져가지 않으면 ERROR로 정의 : 실제로 컨슈머가 문제가 있는 여부 파악 가능

     + HTTP API 제공 : 이를 호출해 Response 받은 데이터를 Application에 활용 가능
