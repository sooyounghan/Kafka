-----
### 세그먼트와 삭제 주기
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/00e293c2-6a80-42a5-a237-0b12b0c0c7f1" />
<img src="https://github.com/user-attachments/assets/02b65d90-61ca-441c-afcc-dc70e4c20735" />
</div>

1. 세그먼트와 삭제 (cleanup.policy=delete)
   - retetion.ms(minutes, hours) : 세그먼트를 보유할 최대 기간 (기본값 7일 / 일반적으로 3일로 설정함)
   - retention.bytes : 파티션 당 로그 적재 바이트 값 (기본값 -1 (지정하지 않음))
   - log.retention.check.interval.ms : 세그먼트가 삭제 영역에 들어왔는지 확인하는 간격 (기본값 5분)
   - 카프카에서 데이터는 세그먼트 단위로 삭제가 발생하므로 로그 단위(레코드 단위)로 개별 삭제는 불가능
   - 또한, 로그(레코드)의 메세지 키, 메세지 값, 오프셋, 헤더 등 이미 적재된 데이터에 대해 수정 또한 불가능하므로 데이터를 적재할 때(프로듀서) 또는 데이터를 사용할 때(컨슈머) 데이터를 검증하는 것이 좋음

2. cleanup.policy=compact
<div align="center">
<img src="https://github.com/user-attachments/assets/bda586e6-375c-4927-ad3e-b6fd67ade694" />
</div>

   - 토픽 압축 정책은 일반적으로 생각하는 zip과 같은 압축(Compression)과는 다른 개념
   - 여기서 압축이란 메세지 키 별로 해당 메세지 키의 레코드 중 오래된 데이터를 삭제하는 정책
   - 그렇기 때문에, 삭제(Delete) 정책과 다르게 일부 레코드만 삭제될 수 있음
   - 압축은 액티브 세그먼트를 제외한 데이터가 대상

3. 테일 / 헤드 영역, 클린 / 더티 로그
<div align="center">
<img src="https://github.com/user-attachments/assets/7b6a58df-7016-461f-9ff1-96801891b033" />
</div>

   - 테일 영역 : 압축 정책에 의해 압축이 완료된 레코드들로, 클린(Clean) 로그라고도 부름 (중복 메세지 키가 없음)
   - 헤드 영역 : 압축 정책이 되기 전 레코드들로, 더티(Dirty) 로그라고도 부름 (중복 메세지 키가 존재)

4. min.cleanable.dirty.ratio
<div align="center">
<img src="https://github.com/user-attachments/assets/5cb325d9-6edd-4e12-91da-12669ffe78e9" />
</div>

   - 데이터의 압축 시작 시점은 min.cleanable.dirty.ratio 옵션값을 따름
   - 💡 min.cleanable.dirty.ratio 옵션 값은 액티브 세그먼트를 제외한 세그먼트에 남아 있는 테일 영역 레코드 개수와 헤드 영역의 레코드 개수의 비율을 뜻함
     + 예를 들어, 0.5로 설정한다면 테일 영역의 레코드 개수와 헤드 영역의 레코드 개수와 동일한 경우 압축이 실행
     + 0.9와 같이 크게 설정하면 한 번 압축을 할 때, 많은 데이터가 줄어들게 되므로 압축 효과가 좋음
       * 그러나 0.9 비율이 될 때까지 용량을 차지하므로 용량 효율이 좋지 않음
       * 반면, 0.1과 같이 작게 설정하면 압축이 자주 일어나서 가장 최신의 데이터만 유지할 수 있지만, 압축이 자주 발생하므로 브로커에 부담이 될 수 있음
