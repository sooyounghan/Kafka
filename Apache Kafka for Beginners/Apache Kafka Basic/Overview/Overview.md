-----
### 아파치 카프카 개요 및 설명
-----
1. Kafka 이전
<div align="center">
<img src="https://github.com/user-attachments/assets/9ce42ac8-1d08-445a-8e91-57cc0ac87b7f">
</div>

   - 데이터를 전송하는 소스 애플리케이션과 데이터를 받는 타켓 애플리케이션이 존재
   - 단방향 통신을 수행
   - 소스 애플리케이션과 타겟 애플리케이션이 많아지면서 데이터를 전송하는 것이 매우 복잡해지면서, 데이터를 전송하는 라인이 많아지는 문제 발생 : 배포와 장애에 대응하기 어려워짐

<div align="center">
<img src="https://github.com/user-attachments/assets/1a32bde5-91d1-43fd-b138-6a9fe43a50c4">
</div>

   - 더불어, 데이터를 전송할 때 프로토콜 포맷 파편화가 많아지는 심각한 문제 발생
   - 추후 데이터 전송 시, 데이터 포맷의 변경사항이 발생하면, 유지보수가 매우 어려워지는 문제 발생

2. Kafka : 위의 문제점을 해결하기 위해 Linked-In에서 내부적으로 개발하였고, 오픈 소스로 현재 제
<div align="center">
<img src="https://github.com/user-attachments/assets/231f81c8-1020-4f28-a754-e91e20cf3880">
</div>

   - 소스 애플리케이션과 타겟 애플리케이션의 결합도를 약하게 하기 위해 등장
<div align="center">
<img src="https://github.com/user-attachments/assets/6432520f-2e7a-40d4-979b-af443a7ba033">
</div>

   - 소스 애플리케이션은 아파치 카프카에게 데이터를 전송, 타겟 애플리케이션은 아파치 카프카에서 데이터를 가져오면 됨
   - 소스 애플리케이션에서 보낼 수 있는 데이터 포맷은 거의 제한이 없음

3. Kafka의 특징
<div align="center">
<img src="https://github.com/user-attachments/assets/3f4ede39-d980-4534-b21a-f8bca4046521">
</div>

   - 카프카는 각종 데이터를 저장하는 토픽(Topic)이라는 개념이 존재
      + 큐의 데이터를 넣는 것과 비슷함
      + 데이터를 넣는 것 : Kafka Producer
      + 데이터를 가져감 : Kafka Consumer
   - Producer와 Consumer는 라이브러리로 구성되어 있어 애플리케이션에서 구현 가능
   - 즉, 카프카는 아주 유연한 큐 역할을 하며, Falut Tolerant (고가용성)으로, 서버 이슈가 생기거나, 전원이 나가는 등의 상황에서도 데이터를 손실 없이 복구 가능
   - 또한, 낮은 지연(Delay)과 높은 처리량(Throughput)을 통해서 아주 효과적으로 데이터를 대량 처리 가능
