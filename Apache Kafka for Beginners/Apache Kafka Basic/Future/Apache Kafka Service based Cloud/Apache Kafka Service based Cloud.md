-----
### 클라우드 기반 아파치 카프카 서비스
-----
1. Kafka를 SaaS 형태로 사용할 수 있는 서비스
   - AWS의 MSK(Managed Streaming for Kafka)
   - Confluent의 Cloud Kakfa

2. Confluent의 Cloud Kakfa
   - 클라우드 서비스에서는 고가용성을 위해 Multi Available Zone을 구축하여 사용하는 전략 사용 (AWS MKS에서도 동일하게 사용)
   - Single Available Zone을 사용하게 되면, Available Zone의 장애로 인해 데이터 유실 또는 클러스터를 쓰지 못하는 경우도 존재하므로 Standard 이상 서비스 사용 (유실되는 안 되는 데이터를 다룰 경우)
   - 만약 추가적인 처리량을 가져가고 싶거나 더 많은 스토리지 사용을 원하면 Deplicated 서비스 사용
   - 가격 정책 : 서비스 시간당 사용량 + 서버 사용량
   - 카프카 클러스터를 제어하는 것은 Confluent의 웹 대시보드로 이루어짐
