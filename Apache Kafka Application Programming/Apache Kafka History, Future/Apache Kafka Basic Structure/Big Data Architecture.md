-----
### 빅데이터 아키텍쳐 종류
-----
1. 초기 빅데이터 플랫폼
<div align="center">
<img src="https://github.com/user-attachments/assets/f51991ef-35b3-43ac-85ef-bc6a41484142" />
</div>

   - End-To-End로 각 서비스 애플리케이션으로부터 데이터를 배치로 모음
   - 데이터를 배치로 모으는 구조는 유연하지 못했으며, 실시간으로 생성되는 데이터들에 대한 인사이트를 서비스 애플리케이션에 빠르게 전달하지 못하는 단점 존재
   - 또한, 원천 데이터로부터 파생된 데이터의 히스토리를 파악하기 어려웠고, 계속되는 데이터의 가공으로 인해 데이터가 파편화되면서 데이터 거버넌스(Data Governance : 데이터 표준 및 정책)를 지키기 어려웠음

2. Lambda Architecture
<div align="center">
<img src="https://github.com/user-attachments/assets/c6856394-6e6a-4d8c-a4f1-9082b94fd9e5" />
</div>

   - 3가지 레이어로 구성
   - 배치 레이어 : 배치 데이터를 모아서 특정 시간, 타이밍 마다 일괄 처리
   - 서빙 레이어 : 가공된 데이터를 데이터 사용자, 서비스 애플리케이션이 사용할 수 있도록 데이터가 저장된 공간
   - 스피드 레이어 : 서비스에서 생성되는 원천 데이터를 실시간으로 분석하는 용도로 사용
   - 배치 데이터에 비해 낮은 지연으로 분석이 필요한 경우에 스피드 레이어를 통해 데이터 분석
<div align="center">
<img src="https://github.com/user-attachments/assets/e620c9ee-340f-4968-8ad6-a3c431ac9805" />
</div>

   - 한계
     + 데이터를 배치 처리하는 레이어와 실시간 처리하는 레이어를 분리한 람다 아키텍쳐는 데이터 처리 방식을 명확히 나눌 수 있었지만, 레이어가 2개로 나뉘기 때문에 생기는 단점 발생
     + 데이터를 분석, 처리하는 데 필요한 로직이 2개로 각각 레이어에 따로 존재해야 한다는 점
     + 배치 데이터와 실시간 데이터를 융합하여 처리할 때는 다소 유연하지 못한 파이프라인을 생성해야 한다는 점
     + 1개의 로직을 추상화하여 배치 레이어와 스피드 레이어에 적용하는 형태를 고민한 서밍버드가 있었지만, 완벽히 해결되지 못함

3. Kafa Architecture
<div align="center">
<img src="https://github.com/user-attachments/assets/b0defaf0-12ba-4523-8090-145d25ede86b" />
</div>

   - 람다 아키텍쳐의 단점을 해소하기 위해 제이 크랩스(Jay Kreps : 카프카를 최초로 고안한 개발자로, 전 Linked-In 팀장, 현 Confluent CEO)는 카파 아키텍쳐 제안
   - 람다 아키텍쳐에서 단점으로 부각되었던 로직 파편화, 디버깅, 배포, 운영 분리에 대한 이슈를 제거하기 위해 배치 레이어를 제거
   - 스피드 레이어에서 데이터를 모두 처리할 수 있으므로 엔지니어들은 더욱 효율적으로 개발과 운영에 임할 수 있게 됨
   - 활용
<div align="center">
<img src="https://github.com/user-attachments/assets/c39a8db2-e5d8-4f45-b7d8-8f309392f0f6" />
</div>

   - 로그는 배치 데이터를 스트림으로 표현하기에 적합
   - 일반적으로 데이터 플랫폼에서 배치 데이터를 표현할 때는 각 시점(시간별, 일자별 등)의 전체 데이터를 백업한 스냅샷 데이터를 뜻함
   - 그러나 배치 데이터를 로그로 표현할 때는 각 시점의 배치 데이터의 변환 기록(Change Log)을 시간 순서대로 기록함으로써 각 시점의 모든 스냅샷 데이터를 저장하지 않고도 배치 데이터를 표현할 수 있게 되었음

4. 배치 데이터와 스트림 데이터
<div align="center">
<img src="https://github.com/user-attachments/assets/f92254f8-5710-4418-8a85-e00530184023" />
</div>

   - 배치 데이터를 처리하는 방법 (Hadoop)
<div align="center">
<img src="https://github.com/user-attachments/assets/4610188d-66ac-4e2c-b9f3-9575dc8eda9a" />
</div>

   - 배치 데이터를 처리하는 방법 (Kafka)
     + 스트림 데이터를 배치 데이터로 사용하는 방법은 로그에 시간을 남기는 것
     + 로그에 남겨진 시간을 기준으로 데이터를 처리하면 스트림으로 적재된 데이터도 배치로 처리할 수 있게 됨
     + 예) 2021년 신입생 목록을 배치 데이터로 가져오기 위해 스트림 데이터로 적재된 1월 1일부터 12월 31일까지 데이터를 구체화된 뷰로 가져온다면 배치로 처리 가능
     + 카프카는 로그에 시간(Timestamp)을 남기므로 이런 방식 처리가 가능
<div align="center">
<img src="https://github.com/user-attachments/assets/325b9f7c-5d23-4808-86c1-3bc5edd267e1" />
</div>

5. Streaming Data Lake
<div align="center">
<img src="https://github.com/user-attachments/assets/0eb6ae4d-f112-42a6-985d-ace4301c3281" />
</div>

   - 2020년 카프카 서밋에서 제이 크랩스는 카파 아키텍쳐에서 서빙 레이어를 제거한 아키텍쳐인 스트리밍 데이터 레이크(Streaming Data Lake)를 제안
   - 카파 아키텍쳐는 데이터를 사용하는 고객을 위해 스트림 데이터를 서빙 레이어에 저장
   - 스피드 레이어로 사용되는 카프카에 분석과 프로세싱을 완료한 거대한 용량의 데이터를 오랜 기간 저장하고 사용할 수 있다면 서빙 레이어도 제거되도 됨
   - 즉, 오히려 서빙 레이어와 스피드 레이어가 이중으로 관리되는 운영 리소스를 줄일 수 있는 것

<div align="center">
<img src="https://github.com/user-attachments/assets/552b66bd-2913-421f-b2ab-4af18c11f478" />
</div>

   - 다만, 아직 카프카를 스트리밍 데이터 레이크로 사용하기 위해 개선해야 하는 부분 존재
     + 우선 자주 접근하지 않는 데이터를 굳이 비싼 자원 (브로커의 메모리, 디스크)에 유지할 필요가 없음
     + 카프카 클러스터에서 자주 접근하지 않는 데이터는 Object Storage와 같이 저렴하면서도 안전한 저장소에 옮겨 저장하고 자주 사용하는 데이터만 브로커에서 사용하는 구분 작업 필요

