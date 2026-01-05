-----
### kafka-console-producer
-----
1. hello.kafka 토픽에 데이터를 넣을 수 있는 kafka-console.producer.bat 실행
   - 키보드로 문자를 작성하고 엔터 키를 누르면, 별다른 응답 없이 메세지 값 전송
   - 명령어
```
./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic hello.kafka
```
<div align="center">
<img src="https://github.com/user-attachments/assets/03bd83f9-c444-4300-ac1f-87e847343e5d" />
</div>

2. 메세지 키를 가지는 레코드를 전송
   - 메세지 키를 가지는 레코드를 전송하기 위해서는 몇가지 추가 옵션을 작성해야 함
   - key.separator를 선언하지 않으면, 기본 설정은 Tab Delimiter(\t)이므로 key.separator를 선언하지 않고, 메세지를 보내려면 메세지 키를 작성하고 탭 키를 누른 뒤, 메세지 값을 작성하고 엔터를 누름
   - 여기서는 명시적으로 확인하기 위해 콜론(:)을 구분자로 선언
   - 명령어
```
./bin/windows/kafka-console-producer.bat --bootstrap-server my-kafka:9092 --topic hello.kafka --property "parse.key=true" --property "key.separator=:"
```
<div align="center">
<img src="https://github.com/user-attachments/assets/32b1069f-6cba-481a-a9d0-d2286d8fd352" />
</div>

3. 메세지 키와 메세지 값이 포함된 레코드가 파티션에 전송됨
<div align="center">
<img src="https://github.com/user-attachments/assets/c671a21a-b749-4cf5-a4dd-c9479f48ae41" />
</div>

   - 메세지 키와 메세지 값을 함께 전송한 레코드는 토픽의 파티션에 저장
   - 메세지 키가 null인 경우에는 프로듀서가 파티션으로 전송할 때 레코드 배치 단위(레코드 전송 묶음)로 라운드 로빈으로 전송
   - 메세지 키가 존재하는 경우에는 키의 해시값을 작성하여 존재하는 파티션 중 한개에 할당
   - 이로 인해 메세지 키가 동일한 경우에는 동일 파티션으로 전송
     + 동일한 메세지 키가 있는 레코드들에 대해 순서를 지킬 수 있는 것이 핵심
     + 컨슈머 입장에서는 파티션을 보통 1:1 관게로 데이터를 가져가게 되는데, 이 경우 컨슈머는 0번부터 데이터를 가져감
     + 즉, 특정 데이터에 대해 순서를 지켜서 데이터를 처리하고 싶을 때는, 메세지 키를 넣어서 데이터를 보내게 되는 것 (프로듀서에 메세지 키를 지정하는 것)
  
