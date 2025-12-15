-----
### 카프카 생태계
-----
<div align="center">
<img src="https://github.com/user-attachments/assets/47990580-4abd-46da-96b3-ed700692dbd4" />
</div>

1. Producer, Consumer, Stream, Connect는 모두 Open Source Apache Kafka에 포함된 툴로, 기본적으로 자바 라이브러리로 제공
2. Connect : Cluster로 운영하며, 파이프라인을 지속적으로 템플릿 형태로 반복적으로 여러 번 생성 및 함께 운영 
   - Source Connector : 프로듀서 역할, 특정 데이터베이스나 소스 애플리케이션으로부터 데이터를 가져와서 토픽에 데이터를 넣는 역할
   - Sync Connector : 컨슈머 역할, 타겟 애플리케이션으로 데이터를 보내는 역할
