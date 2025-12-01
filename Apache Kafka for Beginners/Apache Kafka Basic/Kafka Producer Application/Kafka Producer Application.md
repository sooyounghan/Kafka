-----
### Kafka Producer Application
-----
1. Producer : 데이터를 Producing, 즉 생산하는 역할 (즉, 데이터를 카프카 토픽에 생성하는 것을 의미)
2. 역할
   - 토픽에 전송할 메세지를 생성
   - 특정 토픽으로 데이터를 Publish(전송)
   - 처리 실패 및 재시도 : 카프카 브로커로 데이터를 전송할 때, 전송 성공 여부 및 실패할 경우 재시도할 수 있음

3. 프로듀서와 컨슈머를 사용하기 위해서는 아파치 카프카 라이브러리를 추가해야 함
<div align="center">
<img src="https://github.com/user-attachments/assets/283a41e7-42b1-4401-8c90-84a1b29b374f">
</div>

  - 카프카는 브로커 버전과 클라이언트 버전의 하위 호환성이 완벽하게 모든 버전에 대해 지원하지 않음
  - 일부 카프카 버전은 특정 카프카 클라이언트 버전을 지원하지 않을 수 있음

4. 예제 코드
<div align="center">
<img src="https://github.com/user-attachments/assets/33073669-d77e-4e1f-b2cb-67410adf4b1a">
</div>

   - 프로듀서를 위한 설정 : 자바 프로퍼티 객체를 통해 프로듀서 설정 정의
     + 부트스트랩 서버 설정을 로컬 호스트의 카프카를 바라보도록 설정
     + 카프카 브로커의 주소 목록은 2개 이상의 IP와 Port를 설정하도록 권장 : 둘 중 한개가 비정상일 경우, 다른 정상적인 브로커에 연결되어 사용 가능하기 때문임
     + 따라서, 2개 이상 브로커 정보를 넣는 것을 권장
     + Key(메세지를 보내면, 토픽의 파티션이 지정될 때 사용)와 Value에 대해서는 StringSerializer로 설정 : Serializer는 Key 또는 Value를 직렬화하기 위해 사용
     + 카프카 프로듀스 인스턴스를 생성

   - 카프카 클라이언트에서는 ProducerRecord라는 클래스를 제공
     + 이 인스턴스를 생성할 때 어느 토픽에 넣을 것인지 Key와 Value를 지정 가능
     + 만약 Key를 포함하여 전송하고 싶다면, ```ProducerRecord record = new ProducerRecord<String, String> ("click_log", "1", "login");```으로 지정

   - send() 메서드의 파라미터로 ProducerRecord를 담아 전송하면 전송
   - 전송이 완료되면 close() 메서드를 통해 프로듀서를 종료

5. 프로듀서가 전송한 데이터의 흐름
   - Key가 null인 데이터를 파티션이 1개인 토픽에 보내면 그림과 같이 차례대로 쌓임

<div align="center">
<img src="https://github.com/user-attachments/assets/a714966c-661c-4a01-a0c3-dca30c77e8a1">
</div>

   - 파티션이 1개 더 늘어나게 되면, Key는 null이므로, 라운드 로빈에 의해 2개의 파티션에 쌓이게 됨
<div align="center">
<img src="https://github.com/user-attachments/assets/afd39c03-52b4-4314-8ec2-1f2b6cde38e1">
</div>

   - 반면에 Key가 존재하는 데이터를 토픽에 보낼 경우 : Key를 특정한 해시값으로 변경하므로 파티션과 1:1 매칭
<div align="center">
<img src="https://github.com/user-attachments/assets/073555be-1c7b-4f30-8508-0bf92de1d04e">
</div>

   - 파티션을 한 개 더 추가할 경우 : 새로운 파티션이 추가되는 순간 Key와 파티션 매칭이 깨지므로, 둘의 연결이 보장되지 않음 (따라서, 파티션 개수와 키의 개수의 주의하며, 키를 설정했다면 추후에 파티션을 생성하는 것을 지양) 
<div align="center">
<img src="https://github.com/user-attachments/assets/4688d0d3-2e53-400a-9edc-99c359356e9c">
</div>
