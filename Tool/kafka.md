# [apache kafka](../Terms/Terms.md#Kafka)
- pub/sub 구조의 Message Queue
- Topic 단위로 pub/sub 이루어짐

## 특징
### 1. 분산 시스템
- 유동적으로 서버 추가/삭제 가능
- 부하 분산

### 2. 페이지 캐시
- 처리 속도 증가

### 3. 배치 전송 처리
- 작은 I/O는 묶어서 처리하도록 배치 작업 처리


## 주요 옵션
- `broker.id`: 브로커 구분하기 위한 ID
- `delete.topic.enable`: 토픽 삭제 기능을 on/off
- `default.replication.factor`: replication factor 주지 않았을 경우 기본값
- `min.insync.replicas`: 최소 replication factor
- `auto.create.topics.enable`: 존재하지 않는 토픽으로 pub 할 때, 자동 topic 생성 여부
- `offsets.topic.num.partitions`: offsets 토픽의 파티션 수, deploy 이후에는 수정 불가
- `offsets.topic.replication.factor`: offsets 토픽의 replication factor
- `message.max.bytes`: 카프카에서 허용하는 가장 큰 메시지 크기
- `zookeeper.connect`: 주키퍼 접속 정보
- `zookeeper.session.timeout.ms`: 브로커-주키퍼 timeout 시간
- `uncleanleader.election.enable`: ISR 그룹에 포함되지 않은 마지막 리플리카를 리더로 인정 여부


## 용어 정리
### Topic (토픽)
- 데이터를 구분하기 위한 단위
- 249자 미만 영문, 숫자, '.', '_', '-' 로 구성

### Parition (파티션)
- 여러 프로듀서를 대응하기 위한 Partition 필요
- 많은 파티션의 단점
    - 파일 핸들러 낭비
    - 장애 복구 시간 증가
- producer, consumer의 시간당 생산/소비량을 고려해서 결정
- 같은 Partition내에서는 메시지의 순서 보장
- Partion이 다른 메시지의 경우 순서가 보장되어지지 않음

### Offset
- 파티션마다 메시지가 저장되는 위치
- 64bit 정수

### Replication
- Topic을 이루는 각 Partition을 Replication
- 서버 장애를 대비해 복제해놓은 것
- 원본/복제본 구분: Leader, Follwer
    - Leader: RW 일어남
    - Follower: Leader를 replicate하여 동일한 offset 및 Message 가짐, Leader 장애 시 Leader로 교체

### ISR (In Sync Replicas)
- ISR에 속해있는 구성원만 Leader가 될 수 있음
- Follower가 Leader에 sync되는 것을 확인하고, timeout시 방출
- 복구 방식 (`uncleanleader.election.enable` 이용)
    - `true`: 마지막 leader의 복구를 기다려 메시지 손실을 줄이는 방법
    - `false`: ISR 밖의 Replica를 Leader로 삼아 빠른 서비스를 제공하는 방법


## Kafka with [Zookeeper](../Terms/Terms.md#ZooKeeper)
### Znode
- Zookeeper에서 Status 정보들을 key-value 쌍으로 저장

### Kafka Znode
- `/controller`: broker의 failure를 감지해 이에 영향받는 Partition Leader 변경 책임
- `/brokers`: broker 관련 정보 저장, 하위에 `/topics` znode에서 topic 정보 확인 가능
- `/consumers`: 마지막으로 읽은 offset 정보 저장
- `/config`: topic의 상세 설정 정보 확인


## Producer
### 코드
#### Java
```Java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class KafkaBookProducer {
    public static void main(String[] args){
        Properties props = new Properties();
        props.put("bootstrap.servers", "peter-kafka001:9092,pter-kafka002:9092,peter-kafka003:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        Producer<String, String> producer = new KafkaProducer<>(props);

        producer.send(new ProducerRecord<String, String>("peter-topic", "Message")); // 확인하지 않고 전송
        producer.send(new ProducerRecord<String, String>("peter-topic", "key", "value")); // 확인하지 않고 전송


        RecordMetadata metadata = producer.send(new ProducerRecord<String, String>("peter-topic", "Message")).get(); // 동기 전송

        producer.send(new ProducerRecord<String, String>("peter-topic", "Message"), new PeterCallback()); // 비동기 전송

        producer.close();
    }
}

import org.apache.kafka.clients.producer.Callback;

public PeterCallback implements Callback {
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        // 비동기 전송 후 코드 작성
    }
}
```

#### Python3
```python
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='peter-kafka001:9092,pter-kafka002:9092,peter-kafka003:9092')
producer.send('peter-topic', 'Message')
```

### 옵션
- bootstrap.servers: 서버 정보 (`{host name}:{port #}`)
- acks: 메시지 보내고 받아야 하는 ack 수 (`0`, `1`, `all`, 등)
    - 많은 acks 수: 메시지 손실 가능성 낮음
    - 적은 acks 수: 성능 좋음
- buffer.memory: 서버로 보내기 전 대기할 수 있는 메모리 바이트
- compression.type: 데이터 압축 시, type (`none`, `gzip`, `snappy`, `lz4`, 등)
- retries: 전송 실패 데이터 재선송 횟수
- batch.size: 배치 byte 사이즈
- linger.ms: 한꺼번에 보내기 위해 추가 메시지 기다리는 시간, batch.size에 도달하면 즉시 전송
- max.request.size: 최대 메시지 byte 사이즈


## Consumer
### 종류
- Old Consumer: consumer offset을 [Zookeeper Znode](#Znode)에 저장
- New Consumer: consumer offset을 Kafka Topic에 저장

### 코드
#### Java
```Java
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;

public class KafkaBookConsumer {
    public static void main(String[] args){
        Properties props = new Properties();
        props.put("bootstrap.servers", "peter-kafka001:9092,pter-kafka002:9092,peter-kafka003:9092");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("peter-topic"));   // topic 구독

        try {
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(100);   // consume
                for (ConsumerRecords<String, String> record : records)  // 한 번에 온 여러 record 처리
                    // message 처리
            }
        } finally {
            consumer.close();
        }
    }
}
```

#### Python3
```python
from kafka import KafkaConsumer

consumer = KafkaConsumer('peter-topic',group_id='pter-consumer',bootstrap_servers='peter-kafka001:9092,pter-kafka002:9092,peter-kafka003:9092')
for message in consumer:
    # message 처리
```
### Consumer Group
- consumer 각각이 partition 하나씩을 점유
- 같은 topic에 대해서 같은 `group.id` 공유
- partition:consumer
    - `1:1`
    - `n:1`
    - `1:n` 은 불가

#### Rebalnace
- consumer group 내에서 partition 소유권이 이전되는 것

#### Commit & Offset
- Commit: 각 partition에 대한 offset 위치 업데이트
    - 자동 Commit
        - `enable.auto.commit=true`
        - `auto.commit.interval.ms` 옵션을 통한 주기 조정
        - 5초마다 `poll()` 호출 시 마지막 offset commit
    - 수동 Commit
        - `enable.auto.commit=false`
        - 자체 db에 저장 후 `commit`해 혹시 모를 장애에 대해 대비
        - 메시지를 모두 가져와 `consumer.commitSync()` 호출을 통해 수동 commit
- 특정 Partition 할당
    - `consumer.assign(Arrays.asList(new TopicPartition(topic, 0), new TopicPartition(topic, 1)));`
- 특정 Offset 메시지 가져오기
    - `consumer.seek(new TopicPartition(topic, 0), 2);`

### 옵션
- `bootstrap.servers`: 서버 정보 (`{host name}:{port #}`)
- `fetch.min.bytes`: 한번에 가져올 수 있는 최소 사이즈
- `fetch.max.bytes`: 한번에 가져올 수 있는 최대 사이즈
- `fetch.max.wait.ms`: `fetch.min.bytes`에 의해 설정된 데이터보다 적은 경우 요청에 응답을 기다리는 최대시간
- `group.id`: consumer group 식별자
- `enable.auto.commit`: 주기적으로 offset commit (background)
- `auto.commit.interval.ms`: offset commit 주기
- `auto.offset.reset`: current offset이 없는 경우 reset
    - earliest: 가장 초기의 offset
    - latest: 가장 마지막의 offset
    - none: 이전 offset 못 찾을 경우 error
- `request.timeout.ms`: 요청 응답 대기 최대 시간
- `session.timeout.ms`: session timeout 시간 (heartbeat 기준)
- `heartbeat.interval.ms`: heartbeat 보내는 주기, 보통 session timeout의 1/3
- `max.poll.records`: `poll()`에 대한 최대 레고드 수 조정
- `max.poll.interval.ms`: consumer가 `heartbeat`만 보내고 메시지를 가져가지 않을 경우, 주기적으로 poll 호출하지 않으면 장애라고 판단 후 다른 consumer에게 파티션 배정


## config 파일
- `server.properties`
