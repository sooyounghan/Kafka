-----
### 컨슈머
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/8b5dbf51-dfaa-4c81-992c-1f5b5d619fed" />
</div>

1. 프로듀서가 전송한 데이터는 카프카 브로커에 적재
2. 컨슈머는 적재된 데이터를 사용하기 위해 브로커로부터 데이터를 가져와서 필요한 처리를 실시
3. 예를 들어, 마케팅 문자를 고객에게 보내는 기능이 있다면, 컨슈머는 토픽으로부터 고객 데이터를 가져와 문자 발송 처리를 하게 됨

4. 컨슈머 내부 구조
<div align="center">
<img src="https://github.com/user-attachments/assets/ad99c4cc-8de9-488c-a0ff-fece982480a5" />
</div>

   - Fetcher : 리더 파티션으로부터 레코드들을 미리 가져와서 대기
   - poll() : Fetcher에 있는 레코드들을 리턴하는 레코드
   - ConsumerRecords : 처리하고자 하는 레코드들의 모음 (오프셋 포함)
