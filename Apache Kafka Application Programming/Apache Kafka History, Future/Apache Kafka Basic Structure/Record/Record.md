-----
### 레코드
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/a9c23543-5bdf-4f30-9e43-d225213dae17" />
</div>

1. 레코드는 타임스탬프, 헤더, 메시지 키, 메세지 값, 오프셋으로 구성
2. 프로듀서가 생성한 레코드가 브로커로 전송되면, 오프셋과 타임스탬프가 지정되어 저장
3. 브로커에 한 번 적재된 레코드는 수정할 수 없고, Retetion 기간 또는 용량에 따라서만 삭제됨
4. 레코드 - 타임스탬프
<div align="center">
<img src="https://github.com/user-attachments/assets/0c1a6484-c34f-4318-8b24-5c0b35213584" />
</div>

   - 레코드의 타임스탬프는 스트림 프로세싱에서 활용하기 위한 시간을 저장하는 용도로 사용
   - 카프카 0.10.0.0 이후 버전부터 추가된 타임스탬프는 Unix timestamp가 포함되며, 프로듀서가 따로 설정하지 않으면, 기본값으로 ProducerRecord 생성 시간(CreateTime)이 들어감
   - 또는 브로커 적재 시간(LogAppendTime)으로 설정할 수 있음
   - 해당 옵션은 토픽 단위로 설정 가능하며, message.timestamp.type 사용

5. 레코드 - 오프셋
<div align="center">
<img src="https://github.com/user-attachments/assets/0f3b2349-daa5-4a61-ace9-1d2b0251af3f" />
</div>

  - 레코드의 오프셋은 프로듀서가 생성한 레코드에는 존재하지 않음
  - 프로듀서가 전송한 레코드가 브로커에 적재될 때 오프셋이 지정
  - 오프셋은 0부터 시작되고 1씩 증가
  - 컨슈머는 오프셋을 기반으로 처리가 완료된 데이터와 앞으로 처리해야 할 데이터를 구분
  - 각 메세지는 파티션별로 고유한 오프셋을 가지므로 컨슈머에서 중복 처리를 방지하기 위한 목적으로도 사용

6. 레코드 - 헤더
<div align="center">
<img src="https://github.com/user-attachments/assets/143db5dd-86ed-4e5e-a011-7bd83bd2a3bd" />
</div>

   - 레코드의 헤더는 0.11부터 제공된 기능
   - key/value 데이터를 추가할 수 있으며, 레코드와 스키마 버전이나 포맷과 같이 데이터 프로세싱에 참고할만한 정보를 담아서 사용할 수 있음

7. 레코드 - 메세지 키
<div align="center">
<img src="https://github.com/user-attachments/assets/e08a8995-3e47-42a4-9b2a-c8fd4f490dd2" />
</div>

   - 메세지 키는 처리하고자 하는 메세지 값을 분류하기 위한 용도로 사용 : 이를 파티셔닝이라고 부름
   - 파티셔닝에 사용하는 메세지 키는 파티셔너(Partitioner)에 따라 토픽의 파티션 번호가 정해짐
   - 메세지 키는 필수 값이 아니며, 지정하지 않으면 null로 설정됨 : 메세지 키가 null인 레코드는 특정 토픽의 파티션에 라운드 로빈으로 전달됨
   - null이 아닌 메세지 키는 해시값에 의해 특정 파티션에 매핑되어 전달됨 (기본 파티셔너의 경우)

8. 레코드 - 메세지 값
<div align="center">
<img src="https://github.com/user-attachments/assets/0af5fe3b-bc79-498e-92e8-2331386d76a6" />
</div>

   - 레코드의 메시지 값은 실질적으로 처리할 데이터가 담기는 공간
   - 메세지 값의 포맷은 제네릭으로 사용자에 의해 지정
   - Float, Byte[], String 등 다양한 형태로 지정 가능하며, 필요에 따라 사용자 지정 포맷으로 직렬화 / 역직렬화 클래스를 만들어 사용할 수도 있음
   - 브로커에 저장된 레코드의 메세지 값은 어떤 포맷으로 직렬화되어 저장되었는지 알 수 없으므로 컨슈머는 미리 역직렬화 포맷을 알고 있어야 함
