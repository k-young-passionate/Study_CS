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
- 여러 프로듀서를 대응하기 위한 파티션 필요
- 많은 파티션의 단점
    - 파일 핸들러 낭비
    - 장애 복구 시간 증가
- producer, consumer의 시간당 생산/소비량을 고려해서 결정

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


## Kafka with Zookeeper
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

## config 파일
- `server.properties`
