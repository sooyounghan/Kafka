-----
### AWS에 카프카 클러스터 설치, 실행
-----
1. ZooKeeper : 카프카 관련 정보를 저장하는 역할
2. 카프카 클러스터를 최소로 구축하기 위해 3대의 EC2 인스턴스를 발급 (테스트 목적의 머신이므로 Amazon Linux 2 AMI(Amazon Machine Image)를 t2.micro로 발급 : t2.micro는 1 CPU, 1 RAM 으로 이루어져 있음)
3. ooKeeper 설치
```bash
$ wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz
```
```bash
$ tar xvf zookeeper-3.4.12.tar.gz
```
  - 방화벽 설정 및 /etc/hosts 설정
    + ZooKeeper와 카프카 클러스터가 각각 통신을 하기 위해서는 아래와 같이 In-Bound 규칙을 추가해야함
    + 기본적으로 AWS EC2를 발급받은 뒤 Security Group의 기본설정은 Out-Bound에 대해 Anywhere로 open되어 있으므로 In-Bound만 추가
    + 추가해야할 Port : 각 Port는 Anywhere기준으로 열도록 함
    + Zookeeper는 2181, 2888, 3888 Port를 사용하므로 3개 포트에 대해 Anywhre 조건으로 Open하며, 카프카 통신을 위해 9092 Port도 열어주는 것이 좋음
<div align="center">
<img src="https://github.com/user-attachments/assets/601c766a-5918-46ad-a166-d70b13a77d2">
</div>

   - 3개의 인스턴스의 이름은 test-broker01, 02, 03로 지정
      + 이를 위해 각 host별로 /etc/hosts를 설정
      + 자기자신의 Host는 0.0.0.0으로 설정하고 나머지 Host는 IP로 할당되록 설정
```bash
// 만약 test-broker01인 경우
0.0.0.0 test-broker01
14.252.123.4 test-broker02
55.231.124.1 test-broker03
```

   - ZooKeeper의 configuration을 설정 : ZooKeeper 폴더내부의 conf 폴더에 zoo.cfg 파일을 생성하여 아래와 같이 configuration을 넣어줌
```bash
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
initLimit=20
syncLimit=5
server.1=test-broker01:2888:3888
server.2=test-broker02:2888:3888
server.3=test-broker03:2888:3888
```
   - ZooKeeper 앙상블을 만들기 위해 각 ZooKeeper마다 myid 라는 파일을 만들어줘야함 : myid의 위치는 /var/lib/zookeeper/myid 이고, 해당 파일에는 숫자를 하나 넣으면 됩니다. test-broker01은 1, test-broker02는 2, test-broker03은 3으로 지정
```bash
// 만약 test-broker01에서 실행한 경우 1이 나와야함
$ cat /var/lib/zookeeper/myid
```
   - ZooKeeper 실행
```bash
$ ./bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/ec2-user/zookeeper-3.4.12/bin/../conf/zoo.cfg
Strating zookeeper ... STARTED
```

4. Kafka 설치
```bash
$ wget https://archive.apache.org/dist/kafka/2.1.0/kafka_2.11-2.1.0.tgz
```
   - 압축 해제
```bash
$ tar xvf kafka_2.11-2.1.0.tgz
```

   - Kafka실행을 위해서 broker.id 설정, ZooKeeper에 대한 설정과 Listener설정을 아래와 같이 설정 : 대상 파일은 Kafka 폴더 내부에 config/server.properties 입니다.
```bash
// test-broker01인 경우 아래와 같이 설정합니다.
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://test-broker01:9092
zookeeper.connect=test-broker01:2181,test-broker02:2181,test-broker03/test
```
   - 참고로 ZooKeeper 설정 시 마지막에 /test 와 같이 route를 넣는 것을 추천 : ZooKeeper의 Root Node가 아닌 Child Node에 카프카 정보를 저장하게 되므로 유지보수에 이득이 있음
   - Kafka 실행
```bash
$ ./bin/kafka-server-start.sh ../config/server.properties
```

5. Console-Producer, Console-Consumer 테스트
   - 실제로 kafka 클러스터가 동작하는지는 외부망에서 정상적으로 접근이 되는지 테스트
   - 테스트 전에 먼저 topic을 아래와 같이 생성
```bash
$ ./bin/kafka-topics.sh --create --zookeeper test-broker01:2181,test-broker02:2181,test-broker03:2181/test \
  --replication-factor 3 --partitions 1 --topic test
```
   - 이제 Console-Producer, Console-Consumer를 동시에 켜고 Topic에 데이터가 정상적으로 처리되는지 확인
```bash
$ ./bin/kafka-console-producer.sh --broker-list test-broker01:9092,test-broker02:9092,test-broker03:9092 \
  --topic test
> This is a message
> This is another message
```
```bash
$ ./bin/kafka-console-consumer.sh --bootstrap-server test-broker01:9092,test-broker02:9092,test-broker03:9092 \
  --topic test --from-beginning
This is a message
This is another message
```
