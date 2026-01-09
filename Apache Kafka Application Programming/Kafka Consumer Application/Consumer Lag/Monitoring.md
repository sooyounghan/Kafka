-----
### 컨슈머 랙을 확인하는 방법
-----
1. 카프카 명령어 사용
   - kafka-consumer.groups.sh/bat 명령어 사용 : 컨슈머 랙을 포함한 특정 컨슈머 그룹의 상태를 확인할 수 있음
   - 컨슈머 랙을 확인하기 위한 가장 기초적인 방법으로 다음과 같은 명령어 사용
```
./bin/windows/kafka-consumer-groups.bat --bootstrap-server my-kafka:9092 --group my-group --subscribe
```
   - 카프카 명령어를 통해 컨슈머 랙을 확인하는 방법은 일회성에 그치고 지표를 지속적으로 기록하고 모니터링하기에 부족
   - 그렇기 때문에, kafka-consumer-groups.sh/bat를 통해 컨슈머 랙을 확인하는 것은 테스트용 카프카에서 주로 사용
<div align="center">
<img src="https://github.com/user-attachments/assets/20b53042-fe5b-4d71-9e9e-bb6dbe583a8d" />
</div>

2. metrics() 메서드 사용
   - 컨슈머 애플리케이션에서 KafkaConsumer 인스턴스의 metrics() 메서드를 활용하면 컨슈머 랙 지표를 확인할 수 있음
   - 컨슈머 인스턴스가 제공하는 컨슈머 랙 관련 모니터링 지표는 3가지로, records-lag-max, records-lag, records-lag-avg
```java
for (Map.Entry<MetricName, ? extends Metric> entry : kafkaConsumer.metrics().entrySet()) {
        if ("records-lag-max".equals(entry.getKey().name()) | "records-lag".equals(entry.getKey().name()) | "records-lag-avg".equals(entry.getKey().name())) {
          Metric metric = entry.getValue();
          logger.info("{}:{}", entry.getKey().name(), metric.metricValue());
    }
}
```

   - 사용 이슈
     + 컨슈머가 정상 동작할 경우에만 확인 가능 : metrics() 메서드는 컨슈머가 정상적으로 실행될 경우에만 호출되므로, 만약 컨슈머 애플리케이션이 비정상적으로 종료되면 더는 컨슈머 랙을 모니터링 할 수 없음
     + 모든 컨슈머 애플리케이션에 컨슈머 랙 모니터링 코드를 중복해서 작성해야 함 : 컨슈머 애플리케이션을 여러 종류로 운영할 경우 각기 다른 컨슈머 애플리케이션에 metrics() 메서드를 호출해 컨슈머 랙을 수집하는 로직을 중복해서 넣어야 하는데, 특정 컨슈머 그룹에 해당하는 애플리케이션이 수집하는 컨슈머 랙은 자기 자신 컨슈머 그룹에 대한 컨슈머 랙만 한정되기 때문임
     + 컨슈머 랙을 모니터링하는 코드는 추가할 수 없는 카프카 서드 파티(Third-Party) 애플리케이션의 컨슈머 랙 모니터링이 불가능

3. 외부 모니터링 툴 사용
   - 컨슈머 랙을 모니터링하는 가장 최선의 방법
   - 데이터 독(Datadog), 컨플루언트 컨트롤 센터(Confluent Control Center)와 같은 카프카 클러스터 종합 모니터링 툴을 사용하면, 카프카 운영에 필요한 다양한 지표를 모니터링 할 수 있음
   - 모니터링 지표에는 컨슈머 랙도 포함되어 있으므로, 클러스터 모니터링과 컨슈머 랙을 함께 모니터링 하기에 적합
   - 컨슈머 랙 모니터링만을 위한 툴로 오픈소스로 공개되어 있는 버로우(Burrow)가 존재
