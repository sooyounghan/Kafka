-----
### 카프카 컨슈머 애플리케이션 (Kafka Consumer Application)
-----
1. Consumer는 파티션에 저장된 데이터를 가져오는데, 데이터를 가져오는 것을 폴링(Polling)이라고 
2. 역할
   - 토픽 내부 파티션으로부터 데이터를 가져옴 (Polling) : 메세지를 가져와서 특정 데이터베이스에 저장하거나 또 다른 파이프라인에 전달
   - Partition(파티션에 있는 데이터 번호) Offset 위치 기록(Commit)
   - Consumer Group을 통한 병렬 처리

3. 카프카 컨슈머 애플리케이션 개발
   - 라이브러리 추가 
<div align="center">
<img src="https://github.com/user-attachments/assets/212d8ede-bc1b-461b-9ad4-2fe2caf2c69a">
</div>

   - 카프카는 브로커와의 버전 차이로 인해 정상적으로 동작하지 않을 수 있으므로, 반드시 호환 가능한 버전인지 확인 후 사용하는 것 권장
   - Consumer
<div align="center">
<img src="https://github.com/user-attachments/assets/df5418ab-cde7-4432-b6cf-c65b60a76689">
</div>

   - 자바 프로퍼티 설정을 통해 기본적인 Consumer 옵션을 지정 가능
   - bootstrap.servers 옵션을 통해 카프카 브로커 설정 : 카프카 브로커 중 한 개에 이슈가 생기면, 다른 브로커가 붙을 수 있도록 여러 개 브로커 설정 권장
   - 그룹 아이디 (Consumer Group) 지정 : 컨슈머들의 묶음
   - Key와 Value에 대한 직렬화 설정
   - KafkaConsumer 클래스를 통해 선언한 설정을 매개변수로하여 Consumer 인스턴스 생성 : 데이터 읽고 처리 가능
     + Consumer 그룹을 정하고, 어느 카프카 브로커에서 데이터를 가져올지 선언했으므로, 어느 토픽을 대상으로 데이터를 가져올지 선언 : Consumer의 subscribe() 메서드를 통해 설정 가능
     + 특정 토픽의 전체 파티션이 아니라 일부 파티션의 데이터만 가져오고 싶다 assign() 메서드 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/49879c51-0e4e-4a77-89ef-5284db524fee">
</div>

   - 만약 Key가 존재하는 데이터라면, 이 방식을 통해 데이터 순서를 보장하는 데이터 처리 가능
   - 데이터를 실질적으로 가져오는 폴링 루프 구문 : 컨슈머 API의 핵심 로직
     + 카프카 컨슈머에서 폴링 루프는 poll() 메서드가 포함된 무한 루프를 의미
     + 컨슈머 API의 핵심은 브로커로부터 연속적으로 컨슈머가 허락하는 한 많은 데이터를 읽는 것
     + 컨슈머는 poll() 메서드를 통해 데이터를 가져오며, poll() 메서드에서 설정한 시간 동안 데이터를 기다림
       * 0.5초 동안 데이터가 도착하기를 기다리고, 이후 코드 실행
       * 만약 0.5초 동안 데이터가 들어오지않으면 빈 값의 records 변수를 반환
       * 데이터가 있다면 데이터가 존재하는 records 값 반환 : records 변수는 데이터 배치로서 레코드 묶음 리스트
       * 실제 데이터를 카프카에서 처리할 때는 가장 작은 단위인 reocrd로 나누어 처리 : records 변수를 for 루프를 반복 처리하면서, 실질적으로 처리하는 데이터를 가져오게 함
       * for 구문 내부에서 record 변수 value() 메서드 반환 값이 이전 프로듀서가 전송한 데이터

4. 프로듀서에서 컨슈머로 메세지 전달
   - Offset은 토픽별로 그리고 파티션 별로 별개로 지정하며, 컨슈머가 데이터를 어느 지점까지 읽었는지 확인하는 용도로 사용
   - 컨슈머가 데이터를 읽기 시작하면 Offset을 Commit하게 되는데, 가져간 내용에 대한 정보는 카프카의 __consumer_offset 토픽에 offset 정보 저장 : 컨슈머를 재실행하면 중지된 시점을 알고 있으므로, 시작 위치부터 다시 복구하여 데이터 처리 가
<div align="center">
<img src="">
</div>

<img width="1902" height="637" alt="image" src="https://github.com/user-attachments/assets/28d64c59-b916-49c1-a6ae-b76ecb924171" />

   - 컨슈머에 이슈가 발생하더라도 데이터 처리 시점을 복구할 수 있는 고가용성 특징을 가짐

5. Multiple Consumer
    - 한 개의 Consumer일 때는, 두 개의 Partition에서 값을 가졈
      + 각 컨슈머가 각각의 파티션을 할당하여 데이터를 가져와 처리
    - 세 개의 Consumer일 때는, 이미 파티션들이 각 컨슈머에 할당이 되므로 더 이상 할당될 파티션이 없어서 동작하지 않음
    - 즉, 여러 파티션을 가진 토픽에 대해 컨슈머를 병렬처리 하고 싶다면, Consumer 개수는 Partition 개수보다 적거나같아야 함

6. Different Group
   - 각기 다른 컨슈머 그룹에 속한 컨슈머들은 다른 컨슈머 그룹에 영향을 미치지 않음
   - __consumer_offset 토픽은 컨슈머 그룹별로, 토픽별로 offset을 나누어 저장함
