-----
### 프로듀서
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/399688b7-1368-4a2e-a461-41d0dd12bdbc" />
</div>

1. 카프카에서 데이터의 시작점으로, 프로듀서 애플리케이션은 카프카에 필요한 데이터를 선언하고 브로커의 특정 토픽의 파티션에 전송
2. 💡 프로듀서는 데이터를 전송할 때, 리더 파티션을 가지고 있는 카프카 브로커와 직접 통신
3. 프로듀서는 카프카 브로커로 데이터를 전송할 때 내부적으로 파티셔너, 배치 생성 단계를 거침

-----
### 프로듀서 내부 구조
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/d17d3466-cf4e-40c2-8437-d81667f9114a" />
</div>

1. ProducerRecord : 프로듀서에서 생성하는 레코드 (오프셋은 미포함)
2. send() : 레코드를 전송 요청하는 메서드
3. Partitioner : 어느 파티션으로 전송할지 지정하는 파티셔너 (기본값으로 DefaultPartitioner로 설정)
4. Accumulator : 배치로 묶어 전송할 데이터를 모으는 버퍼
