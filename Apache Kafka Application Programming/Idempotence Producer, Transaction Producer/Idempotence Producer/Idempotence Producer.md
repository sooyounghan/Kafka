------
### 전달 신뢰성
------
1. 멱등성 : 여러 번 연산을 수행하더라도 동일한 결과를 나타내는 것을 뜻함
2. 멱등성 프로듀서 : 동일한 데이터를 여러 번 전송하더라도 카프카 클러스터에 단 한 번만 저장됨을 의미
3. 기본 프로듀서의 동작 방식은 적어도 한 번 전달(At Least Once Delivery)을 지원
   - 적어도 한 번 전달 : 프로듀서가 클러스터에 데이터를 전송하여 저장할 때, 한 번 이상 데이터를 적재할 수 있고 데이터가 유실되지 않음을 뜻함
   - 다만 두 번 이상 적재할 가능성이 있으므로 데이터 중복이 발생할 수 있음
     + At Least Once : 적어도 한 번 이상 전달
     + At Most Once : 최대 한 번 전달
     + Exactly Once : 정확히 한 번 전달

-----
### 멱등성 프로듀서
-----
1. 프로듀서가 보내는 데이터의 중복 적재를 막기 위해 0.11.0 이후 버전부터는 프로듀서에 eanble.idempotence 옵션을 사용해 정확히 한 번 전달(Exactly Once Delivery)을 지원
   - 기본값은 false이며, 정확히 한 번 전달을 위해서는 true로 옵션값을 설정해 멱등성 프로듀서로 동작하도록 만들면 됨
```java
Properties configs = new Properties();
configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
configs.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true); // 2.5.0 버전은 기본값이 false이므로 true로 변경

KafkaProducer<String, String> producer = new KafkaProducer<>(configs);
```

2. 카프카 3.0.0 부터는 enable.idempotence 옵션값의 기본값을 true(acks=all)로 변경되므로 신규 버전에서 프로듀서의 동작에 유의해서 사용하도록 해야함

3. 동작
   - 멱등성 프로듀서는 기본 프로듀서와 달리 데이터를 브로커로 전달할 때 프로듀서 PID(Producer Unique ID)와 시퀀스 넘버(Sequence Number)를 함께 전달
     + PID(Producer Unique ID) : 프로듀서의 고유한 ID
     + SID(Sequence ID) : 레코드의 전달 번호 ID
   - 그러면 브로커는 프로듀서의 PID와 시퀀스 넘버를 확인하여 동일한 메세지의 적재 요청이 오더라도 단 한 번만 데이터를 적재함으로써 프로듀서의 데이터는 정확히 한 번 브로커에 적재되도록 동작

4. 멱등성 프로듀서가 아닌 경우
<div align="center">
<img width="760" height="621" alt="image" src="https://github.com/user-attachments/assets/13a64d8e-2d96-4087-b553-21b39b720249" />
</div>

5. 멱등성 프로듀서인 경우
<div align="center">
<img width="756" height="631" alt="image" src="https://github.com/user-attachments/assets/b36ffbdc-1f28-4142-aef1-5fe37b999515" />
</div>

6. 한계
   - 멱등성 프로듀서는 동일한 세션에서만 정확히 한 번 전달을 보장
     + 여기서 말하는 동일한 세션이란 PID의 생명주기를 뜻함
   - 만약 멱등성 프로듀서로 동작하는 프로듀서 애플리케이션에 이슈가 발생하여 종료되고 애플리케이션을 재시작하면 PID가 달라짐
     + 동일한 데이터를 보내더라도 PID가 달라지면 브로커 입장에서 다른 애플리케이션이 다른 데이터를 보냈다고 판단하므로 멱등성 프로듀서는 장애가 발생하지 않을 경우에만 정확히 한 번 적재하는 것을 보장한다는 점을 고려

7. 멱등성 프로듀서로 설정할 경우 옵션
   - 멱등성 프로듀서를 사용하기 위해 enable.idempotence를 true로 설정하면 정확히 한 번 적재하는 로직이 성립되기 위해 프로듀서의 일부 옵션들이 강제로 설정
   - 프로듀서의 데이터 재전송 횟수를 정하는 retries는 기본값으로 Integer_MAX_VALUE로 설정되고 acks 옵션은 all로 설정
     + 이렇게 설정되는 이유는 프로듀서가 적어도 한 번 이상 브로커에 데이터를 보냄으로써 브로커에 단 한 번만 데이터가 적재되는 것을 보장하기 위함
   - 멱등성 프로듀서는 정확히 한 번 브로커에 데이터를 적재하기 위해 한 번 전송하는 것이 아닌, 상황에 따라 프로듀서가 여러 번 전송하되 브로커가 여러 번 전송된 데이터를 확인하고, 중복된 데이터는 적재하지 않는 것

8. 멱등성 프로듀서 사용 시 오류 확인
<div align="center">
<img src="https://github.com/user-attachments/assets/5ef3c07c-b9e3-4d81-ab2a-5ec9c4721c9e" />
</div>

   - 멱등성 프로듀서의 시퀀스 넘버는 0부터 시작하여 1씩 더한 것이 전달
   - 브로커에서 멱등성 프로듀서가 전송한 데이터의 PID와 시퀀스 넘버를 확인하는 과정에서 시퀀스 넘버가 일정하지 않은 경우에는 OutOfOrderSeqeucneException이 발생할 수 있음
     + 이 오류는 브로커가 예상한 시퀀스 넘버와 다른 번호의 데이터 적재 요청이 왔을 때 발생
     + OutOfSequenceException이 발생했을 경우에는 시퀀스 넘버의 역전 현상이 발생할 수 있으므로, 순서가 중요한 데이터를 전송하는 프로듀서는 해당 Exception이 발생했을 경우 대응하는 방안을 고려해야 함
  
