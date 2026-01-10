-----
### 카프카 커넥트
-----
1. Kafka Connect : 카프카 기반 데이터 파이프라인을 반복적으로 생성할 때 사용하는 애플리케이션
2. Soruce Connector : 특정 파일 또는 소스 애플리케이션으로부터, 데이터를 토픽으로 보내는 Producer
3. Sink Connector : 토픽에서 특정 파일 또는 소스 애플리케이션으로 데이터를 보내는 Consumer
4. Open-Source Connector : Source / Sink Connector를 사용할 수 있도록 jar 형태로 배포하는 Connector
5. Task : Connector에서 데이터를 처리하는 최소 로직 단위
6. 단일 모드 : 디버깅 / 테스트 용으로 적합한 One-Process 단위 Connect
7. 분산 모드 : 상용환경 운영에 적합한 Multi-Process 단위 Connect
