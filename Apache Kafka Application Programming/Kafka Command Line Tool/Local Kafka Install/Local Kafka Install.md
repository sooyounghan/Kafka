-----
### 로컬 카프카 설치
-----
1. 예제 코드 다운로드
<div align="center">
<img src="https://github.com/user-attachments/assets/70783b49-3457-4086-8f4d-e8e8ad0ebb25" />
</div>

2. IntelliJ CE 다운로드
<div align="center">
<img src="https://github.com/user-attachments/assets/9c7791d1-c593-49ca-8bbf-3d4259d1c9ca" />
</div>

3. 로컬 카프카 설치 및 실행
   - 카프카 바이너리 파일 다운로드
     + ```https://kafka.apache.org/downloads```
     + Binary Downloads : kafka_2.12-2.5.0.tgz

   - 카프카 바이너리 압축 해제
<div align="center">
<img src="https://github.com/user-attachments/assets/02ad556b-b176-4418-ac75-b48343b1a904" />
</div>

   - server.properties
     + log.dirs : 파일 시스템을 지정하는 부분 (프로듀서가 데이터를 보내게 되면, 프로듀서에 있는 데이터는 카프카 브로커로 가게 되고, 카프카 브로커에 있는 데이터는 파일 시스템에 저장)
     + log.dirs=C:/kafka_2.12-2.5.0/data로 변경 (C:/kafka_2.12-2.5.0 내 data 디렉토리 생성)
     + num.partitions : 카프카 토픽을 만들 때, 기본적으로 만들 파티션 개수
<div align="center">
<img src="https://github.com/user-attachments/assets/00ea9466-e74f-4efe-b511-ead482f187de" />
</div>

   - 주키퍼 실행
     + 상용 환경에서는 앙상블로 따로 설치하여 진행되는게 일반적임 : 주키퍼에서는 내부적으로 어떤 투표 과정을 거쳐 가장 알맞은 데이터를 고르는 과정이 존재
     + 로컬에서 테스트하는 환경에서는 로컬에 한 대만 띄워서도 활용할 수 있도록 카프카 바이너리에도 포함되어 있으므로, 카프카 브로커를 실행하기 전 다음과 같이 설정 값을 통해 주키퍼를 실행
     + 윈도우 환경 : 윈도우 터미널 실행 - 파워 쉘 실행 (주키퍼 실행 명령어 : ```.\bin\windows\zookeeper-server-start.bat config\zookeeper.properties```)
<div align="center">
<img src="https://github.com/user-attachments/assets/a025e656-0d03-4830-9a84-dbc12a0f7f2e" />
</div>

   - 카프카 브로커 실행 : kafka-server-start.sh와 함께 config/server.properties하는 파일의 내용을 참조받아서 사용
     + 이 파일의 내용을 바탕으로 카프카 서버, 즉 카프카 서버는 브로커 임을 알 수 있음
     + started라는 로그가 나오게 되면, 카프카 브로커를 정상적으로 실행시켰음을 확인 가능
     + 카프카 실행 명령어 : ```.\bin\windows\kafka-server-start.bat ./config\server.properties```
       * 수정 사항 : \bin\windows\kafka-server-start.bat 내용 변경 (아래 내용 삭제 후, 내용 삽입)
```bat
IF ["%KAFKA_HEAP_OPTS%"] EQU [""] (
    rem detect OS architecture
    wmic os get osarchitecture | find /i "32-bit" >nul 2>&1
    IF NOT ERRORLEVEL 1 (
        rem 32-bit OS
        set KAFKA_HEAP_OPTS=-Xmx512M -Xms512M
    ) ELSE (
        rem 64-bit OS
        set KAFKA_HEAP_OPTS=-Xmx1G -Xms1G
    )
)
```
```bat
IF ["%KAFKA_HEAP_OPTS%"] EQU [""] (
    rem force 64-bit heap settings (wmic removed in modern Windows)
    set KAFKA_HEAP_OPTS=-Xmx1G -Xms1G
)
```
   - C:\kafka_2.12-2.5.0\bin\windows\kafka-run-class.bat : wmic 검색 → 전부 삭제 → 아래로 교체 (없을 시 생략)
```bat
set OS_ARCH=64-bit
```

<div align="center">
<img src="https://github.com/user-attachments/assets/4cfbe83b-8343-42d2-a681-d12d4e851959" />
</div>

   - 카프카 정상 실행 여부 확인
      +  bin/kakfa-broker-api-versions.sh라는 쉘 스크립트를 사용하면 로컬 호스트에 띄워져 있는 카프카 브로커에 대해 정상적으로 띄워졌는지 통신을 통해 각종 옵션에 대한 정보 조회 가능
      +  명령어 : ```./bin/windows/kafka-broker-api-versions.bat --bootstrap-server localhost:9092```
      +  명령어 : ```./bin/windows/kafka-topics.bat --bootstrap-server localhost:9092 --list```
<div align="center">
<img src="https://github.com/user-attachments/assets/6cdc69d1-2829-4b64-b659-e2df8b1deb76" />
</div>

   - 테스트 편의를 위한 hosts 설정
     + 명령어 : ```notepad C:\Windows\System32\drivers\etc\hosts```
     + 파일 맨 아래에 ```127.0.0.1    my-kafka``` 입
<div align="center">
<img src="https://github.com/user-attachments/assets/0e9e85de-9070-413d-b47d-c12c20a84f90" />
</div>

   - 카프카 바이너리 실행

