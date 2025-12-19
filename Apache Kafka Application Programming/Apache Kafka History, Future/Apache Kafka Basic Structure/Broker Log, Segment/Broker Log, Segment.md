-----
### 브로커 로그와 세그먼트
-----
1. 데이터의 저장
   - 카프카를 실행할 때 config/server.properties의 log.dir 옵션에 정의한 디렉토리에 데이터를 저장 : 토픽 이름과 파티션 번호의 조합으로 하위 디렉토티를 생성하여 데이터를 저장
<div align="center">
<img src="https://github.com/user-attachments/assets/d405fc35-6a31-46ff-b5a6-4d724361978f" />
</div>

   - hello.kakfa 토픽의 0번 파티션에 저장하는 데이터를 확인할 수 있음
     + log에는 메세지와 메타데이터를 저장
     + index는 메세지의 오프셋을 인덱싱한 정보를 담은 파일
     + timeindex 파일에는 메세지(프로듀서가 보낼 레코드 하나, 즉, 레코드 하나의 데이터가 메세지라고 부르며, 카프카에서는 공식적으로 레코드라고 부름)에 포함된 timestamp 값을 기준으로 인덱싱한 정보가 담겨있음
       * 레코드에는 데이터, 메세지 키, 메시지 값과 같은 데이터가 포함

2. 로그와 세그먼트
<div align="center">
<img src="https://github.com/user-attachments/assets/00e293c2-6a80-42a5-a237-0b12b0c0c7f1" />
<img src="https://github.com/user-attachments/assets/02b65d90-61ca-441c-afcc-dc70e4c20735" />
</div>

   - log.segment.bytes : 바이트 단위의 최대 세그먼트 크기 지정 (기본값 1GB)
   - log.roll.ms(hours) : 세그먼트가 신규 생성된 이후 다음 파일로 넘어가는 시간 주기 (기본값 7일) / 즉, 1GB가 넘거나 도달하지 않더라도 세그먼트가 종료되고, 다음 세그먼트로 넘어가도록 설정하는 것
   - 000000.log, 000010.log, 000020.log : 번호는 offset 번호로, 레코드의 고유한 번호로, 프로듀서가 레코드를 생성해 브로커로 데이터를 보내게 되면, 특정 파티션 중 하나의 데이터에 저장 
   - 위 그림에서는 총 22개 레코드가 저장되어 있음
   - 프로듀서가 레코드를 전송하게 되면, 액티브 세그먼트라고하는 가장 최신의 세그먼트 로그에 추가로 데이터가 쓰여짐
   - 마지막 번호에서 다음 번호, 즉 파일에 만들어지는 가장 최초의 offset 번호가 파일 이름이 됨
         
3. 액티브 세그먼트
   - 가장 마지막 세그먼트 파일(쓰기가 일어나고 있는 파일)을 액티브 세그먼트라고 부름 (나머지 세그먼트는 일반 세그먼트)
   - 액티브 세그먼트는 브로커 삭제 대상에서 포함되지 않음
   - 액티브 세그먼트가 아닌 세그먼트는 retention 옵션에 따라 삭제 대상으로 지정

4. 세그먼트와 삭제 주기 (cleanup.policy=delete)
   - retetion.ms(minuts, hours) : 세그먼트를 보유할 최대 기간 (기본값 7일)
   - retention.bytes : 파티션 당 로그 적재 바이트 값 (기본값 -1 (지정하지 않음))
   - log.retention.check.interval.ms : 세그먼트가 삭제 영역에 들어왔는지 확인하는 간격 (기본값 5분)
   
