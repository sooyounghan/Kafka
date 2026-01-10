-----
### 카프카 활용 아키텍쳐 사례 분석
-----
1. 카카오 스마트 메세지 서비스
<div align="center">
<img src="https://github.com/user-attachments/assets/9a0721bb-a4bd-4f7e-8c70-e76149513a88" />
</div>

   - 스마트 메세지 서비스는 소재 최적화와 유저 타겟팅을 통해 적합한 유저에게 광고 메세지를 개인화 전송하는 서비스
   - 사용자에게 흥미가 있는 소재를 최적화하고, 적합한 유저에게 타겟팅하여 광고를 선별 전송
   - 아키텍쳐
<div align="center">
<img src="https://github.com/user-attachments/assets/8c1d4d55-4398-430b-b624-0c852bd7939e" />
</div>

   - 스마트 메세지의 카프카 스트림즈 활용
     + 사용자의 반응 로그(imp, click)을 취합하여 저장하고 유저 데이터를 매핑하여 처리하는 용도로 카프카 스트림즈 활용
     + 카프카 스트림즈의 groupByKey, windowedBy, aggregrate 구문을 통해 1분 로그를 window 취합하여 적재하고, map 구문을 통해 Redis의 유저 데이터와 결합 처리하는 로직 수행
<div align="center">
<img src="https://github.com/user-attachments/assets/d9bf1a69-b51e-481b-80a4-a549218fb0fc" />
</div>


2. 넷플릭스 키스톤 프로젝트
<div align="center">
<img src="https://github.com/user-attachments/assets/0b3398ee-361d-4c50-a5c3-9c550bd08919" />
</div>

   - 첫번째 단계 카프카(Fronting Kafka)는 라우팅(라우터 : 두 개의 카프카 클러스터를 연결) 용도로 모든 데이터 수집
   - Router(자체 스트림 프로세싱, 라우팅 애플리케이션)을 사용하여 두번쨰 카프카(Consumer Kafka) 또는 하둡, 엘라스틱서치와 같은 데이터베이스로 전달

3. 라인 쇼핑 플랫폼 사례
<div align="center">
<img src="https://github.com/user-attachments/assets/46ef331c-6e8e-4123-9f8d-b11d4cbc62cd" />
</div>

   - 시스템 유연성을 개선하고 처리량에 한계를 없애기 위해 카프카를 중앙에 둔 아키텍쳐 적용 (무중단 이벤트 스트림 프로세싱을 달성하기 위함)
   - 카프카 커넥트 도입, MongoDB CDC(Change Data Capture) 커넥터 활용
     + 즉, 어떤 주문 데이터나 특정 데이터에 대해 데이터가 변환되면 CDC 데이터를 기반으로 다시 카프카로 데이터를 재가공해서 가져옴
   - 이벤트 기반 데이터 처리로 상품 처리가 매우 빨라짐

4. 11번가, 주문 / 결제 시스템 적용 사례
<div align="center">
<img src="https://github.com/user-attachments/assets/4e8b4c35-5eee-48ec-833c-ceb8e173e1f8" />
</div>

   - 데이터베이스의 병목현상을 해소하기 위해 도입
   - 카프카를 단일 진실 공급원(Source Of Truth)으로 정의
   - 메세지를 Materialized View로 구축하여 배치 데이터처럼 활용

-----
### 카프카 기술 별 아키텍쳐 적용 방법 정리
-----
1. 카프카 커넥트
   - 반복적인 파이프라인을 만들어야 할 경우 분산 모드 커넥트를 설치하고 운영
   - 되도록 오픈소스 커넥터를 사용하되, 필요시 커스텀 커넥터를 개발하여 운영
   - REST API와 통신하여 동작할 수 있는 웹 화면을 개발하거나 오픈 소스 활용 추천

2. 카프카 스트림즈 : 강력한 Stateful, Stateless 프로세싱 기능이 있으므로 카프카 토픽의 데이터 처리 시 선택
3. 카프카 컨슈머, 프로듀서 : 커넥트와 스트림즈로 구현할 수 없거나 단일성 파이프라인 개발일 경우 컨슈머 또는 프로듀서로 개발
4. 정리
<div align="center">
<img src="https://github.com/user-attachments/assets/c082b07c-a9e1-4f8f-b3f9-cc9ee6e75d1e" />
</div>

   - 프로듀서를 통해 데이터 최초 유입
   - 토픽 기반 데이터 프로세싱은 카프카 스트림즈로 처리
   - 반복된 파이프라인은 카프카 커넥트에서 운영 (단발적인 처리는 컨슈머로 개발)
