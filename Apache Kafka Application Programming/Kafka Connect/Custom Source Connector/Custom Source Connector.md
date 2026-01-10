-----
### 커스텀 소스 커넥터
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/beb5761c-fd32-4ebd-bdde-5b32e527ba75" />
</div>

1. 소스 커넥터는 소스 애플리케이션 또는 소스 파일로부터 데이터를 가져와 토픽으로 넣는 역할
   - 오픈소스 소스 커넥터를 사용해도되지만, 라이센스 문제나 로직이 원하는 요구사항이 맞지 않아 직접 개발해야 하는 경우가 존재하는데, 이 때는 카프카 커넥트 라이브러리에서 제공하는 SourceConnector와 SourceTask 클래스를 사용해 직접 소스 커넥터를 구현
   - 직접 구현한 소스 커넥터를 빌드하여 jar 파일로 만들고, 커넥트 실행 시 플러그인으로 추가하여 사용할 수 있음

2. Dependency : 소스 커넥터를 만들 때는 connect-api 라이브러리를 추가
   - conenct-api 라이브러리에는 커넥터를 개발하기 위한 클래스들이 포함
   - build.gradle
```gradle
dependencies {
    implementation 'org.apache.kafka:connect-api:2.5.0'
}
```
