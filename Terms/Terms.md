# Terms

## D

### Data warehouse
- 데이터를 추출, 변환, 요약하여 능동적으로 사용자에게 제공할 수 있는 데이터의 집합체

## F
### First-Class Object
- 일급 객체
- 다른 객체들에 일반적으로 적용되는 연산을 모두 지원하는 객체
- 보통 함수에 매개변수로 넘기기, 수정하기, 변수에 대입하기와 같은 연산을 지원할 때 일급 객체라 함
- ref: [wiki](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4)

## H

### Hadoop
- 분산 처리를 위한 프레임워크
- [HDFS](#HDFS)와 [MapReduce](#MapReduce) 구현

### HBase
- 비관계형 분산 데이터베이스
- [HDFS](#HDFS)위에서 동작

### HDFS
- [Hadoop](#Hadoop) 분산 파일 시스템
- [Hadoop](#Hadoop) 프레임워크를 위해 여러 기계에 대용량 파일을 나눠서 저장

### Hive
- [Hadoop](#Hadoop)에서 동작하는 [Data warehouse](#Data-warehouse)

### Hue
- [Hadoop](#Hadoop) User Experience
- 웹 기반 [Hadoop](#Hadoop) 인터페이스
- [Hive](#Hive) 작업 및 [Spark](#Spark) 작업 등을 실행 가능


## J

### JPA
- Java Persistent API
- RDB의 관리를 표현한 Java API
- Hibernate 등이 지원

### jQuery
- [HTML](#HTML)의 조작을 단순화한 [javascript](#Javascript) [library](#Library)
- [DOM](#DOM), event, 특수 효과, [ajax](#Ajax) 함수 포함
```HTML
<script type="text/javascript" src="path/to/jQuery.js"></script> // 웹페이지에 포함
```

## K

### Kafka
- pub-sub 모델의 [Message Queue](#Message-queue)
- publisher와 subscriber 모두 topic만 바라보는 구조


## M

### MapReduce
- Map: 흩어진 데이터를 key-value 값으로 묶어줌
- Reduce: key를 중심으로 필터링/정렬

### Method
- Class에서 생성된 Instance와 관련된 동작 정의
- 데이터와 멤버 변수에 대한 접근 권한 가짐

## O

### ORM
- Object Relation Mapping
- 가상 객체 데이터베이스 구축
- 데이터를 객체로 변환


## P

### Presto
- 분산된 [SQL](#SQL) query 엔진
- [Hive](#Hive)와 [MapReduce](#MapReduce)에 비해 훨씬 빠름


## R

### RDD
- `Resilient Distributed DataSet`
- 여러 노드에 걸쳐 저장되는, 변경 불가능한 데이터 집합
- 하나의 RDD는 여러 개의 파티션으로 분리
- 연산
    - Transformation: 기존 RDD로 filter/map 등을 이용해 새 RDD 생성
    - Action: RDD 기반으로 계산해서 결과 생성 (count 등)

### Referential Transparency
- 참조 투명성
- 함수가 함수 외부에 영향을 받지 않는 것
- 함수의 결과는 파라미터에만 의존
- 참조 투명성을 지닌 코드는 다음과 같은 특징을 가짐
    - self-contained: 자기 충족적, 외부에 의존하는 코드가 없음
    - deterministic: 결정론적, 동일한 parameter에 동일한 결과
    - no exception: out of memory, stack overflow 외의 exception 발생 X
    - 다른 코드가 예기치 않는 실패하지 않게 함 (매개변수 변경 및 외부 데이터 변경 X)
    - db, fs, network 등의 외부기기로 인한 동작이 멈추지 않음
- ref: [Jinwoo's Blog](https://jinwooe.wordpress.com/2017/12/21/%EB%B6%80%EC%88%98-%ED%9A%A8%EA%B3%BC-side-effect-%EC%B0%B8%EC%A1%B0-%ED%88%AC%EB%AA%85%EC%84%B1-referential-transparency/)

## S

### Spark
- 빅데이터를 분석하는 애플리케이션의 성능을 향상시키는 오픈소스 병렬 처리 [Framework](#Framework)
- 너무 크거나 복잡한 데이터 처리

### SQL Aggregation
- sql의 집계함수
- GROUP BY 절과 함께 사용
- APPROX_COUNT_DISTINCT, AVG, COUNT, GROUPING, MAX, MIN, SUM, 등

## T

### Template engine
- template 양식에 데이터 입력 자료를 합성하여 결과 문서 출력하는 소프트웨어
- jsp, thymeleaf 등


## V

### Vagrant
- portable한 virtual 소프트웨어 개발 환경의 생성 및 유지보수를 위한 open source software
- 명령어
    - `vagrant up` : 설치된 virtual machine 시작


## Z

### Zeppelin
- web 기반 notebook style의 데이터 분석 툴
- 코드 작성 - 실행 - 결과 확인 - 코드 수정 가능

### [Znode](../Tool/kafka.md#Znode)

### ZooKeeper
- 분산 시스템을 위한 네이밍 레지스트리 제공
- 정보 공유, 상태 체크, 동기화 처리 프레임워크
