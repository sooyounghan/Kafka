-----
### 커넥트를 실행하는 방법
-----
1. 단일 모드 커넥트(Standalone Mode Kafka Connect) : 단일 애플리케이션으로 실행
<div align="center">
<img width="893" height="405" alt="image" src="https://github.com/user-attachments/assets/209b5159-29a0-4962-b708-4b5258b84554" />
</div>


   - 커넥트를 정의하는 파일을 작성하고, 해당 파일을 참조하는 단일 모드 커넥트를 실행함으로써 파이프라인 생성 
   - 1개 프로세스만 실행되는 점이 특징 : 단일 프로세스로 고정되므로 고가용성 구성이 되지 않아 단일 장애점(SPOF, Single Point Of Failure)이 될 수 있음
   - 그러므로 단일 모드 커넥트 파이프라인은 주로 개발 환경이나 중요도가 낮은 파이프라인을 운영할 때 사용

2. 분산 모드 커넥트(Distributed Mode Kafka Connect)
<div align="center">
<img width="860" height="393" alt="image" src="https://github.com/user-attachments/assets/cba2291e-81d1-4cca-80f9-f2af6c726945" />
</div>

   - 2대 이상의 서버에서 클러스터 형태로 운영함으로써 단일 모드 커넥트 대비 안전하게 운영할 수 있다는 장점이 존재
   - 2개 이상 커넥트가 클러스터로 묶이면 1개의 커넥트가 이슈 발생으로 중단되더라도 남은 1개의 커넥트가 파이프라인을 지속적으로 처리할 수 있음
   - 분산 모드 커넥트는 데이터 처리량의 변화에도 유연하게 대응할 수 있음 : 커넥트가 실행되는 서버 개수를 늘림으로써 무중단으로 스케일 아웃하여 처리량을 늘릴 수 있음
   - 이러한 장점으로 상용환경에서 커넥트를 운영한다면 분산 모드 커넥트를 2대 이상 구성하고 설정하는 것이 좋음

-----
### 커넥트 REST API 인터페이스
-----
1. REST API를 사용하면 현재 실행 중인 커넥트의 커넥트 플러그인 종류, 태스크 상태, 커넥터 상태 조회 가능
2. 커넥트는 8083 포트로 호출할 수 있으며, HTTP 메서드 기반 API 제공
<div align="center">
<img src="https://github.com/user-attachments/assets/db2ea39e-5532-44dd-8b84-1c20a97969d7" />
</div>
