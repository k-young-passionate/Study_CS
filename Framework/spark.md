# [spark](../Terms/Terms.md#Spark)
## 개념
- 인메모리 기반 대용량 데이터 고속 처리
- 범용 분산 클러스터 컴퓨팅 프레임워크

## 특징
- Speed: in-memory 기반이라 빠름
- 쉬운 사용: java, python, Scala, R, SQL 지원
- SQL, Machine Learning 등 지원
- 적용: YARN, Mesos, Kubernetes 등에서 작동, [HBase](../Terms/Terms.md#HBase), [HDFS](../Terms/Terms.md#HDFS), Casandra 등의 파일 포맷 지원

## 컴포넌트 구성
![component](http://cfile25.uf.tistory.com/image/2140BE3C555DFB51305898)
### Core
- 작업 스케줄링, 메모리 관리, 장애 복구
- [RDD](../Terms/Terms.md#RDD), Dataset, DataFrame을 이용한 [Spark](../Terms/Terms.md#Spark) 연산 처리

### Library
1. Spark SQL
    - SQL을 이용한 data 처리
    - [Shark](../Terms/Terms.md#Shark): [Hive](../Terms/Terms.md#Hive)에서 [Spark](../Terms/Terms.md#Spark)작업 처리
1. Spark Streaming
    - 실시간 데이터 스트림 처리
    - 작은 사이즈로 쪼개 [RDD](../Terms/Terms.md#RDD)처럼 처리
1. MLib
    - [Machine Learning](../Terms/Terms.md#MachineLearning) 기능 제공
    - 분류, 회귀, 클러스터링, 협업 필터링 등 지원
1. GraphX
    - Graph 그려줌

### Cluster Manager
- [Spark](../Terms/Terms.md#Spark) 작업을 운용하는 Cluster 관리자
- Mesos, YARN, Kubernetes 등의 Manager 지원

## 구조
### Driver
- application 실행하는 process
- 실행 시, SparkContext 생성
- 기능
    - 사용자에게 입력을 받아 application에 전달
    - node에 있는 Executer가 실행한 결과 return

#### Executer
- 실제 작업 진행
- 결과를 Driver에 알려줌

#### Task
- Executer에서 실제로 진행되는 작업
- Executer와 Cache 공유

#### 작업의 단위
- Job: Spark Application으로 제출된 작업
- Stage: Job을 작업에 따라 구분
- Task: 실제 작업, 읽거나 필터링 작업

### Cluster Manager
- 클러스터 운용

## Monitoring
- Web UI 제공: 4040 포트
- history server: 18080 포트, REST api 이용


# Java Spark
- 참고: [apache spark docs](https://spark.apache.org/docs/latest/api/java/index.html), [apache spark getting started](https://spark.apache.org/docs/latest/sql-getting-started.html), [apache spark guide](https://spark.apache.org/docs/2.1.1/programming-guide.html)

## class, method 명
### SparkSession
- SparkSession: Spark programming 시작 점
- `builder()`: Spark sessions 생성
- `appName(param)`: cluster에서 돌아갈 application 이름, spark web UI에 표출
- `master(param)`: cluster의 url, "local" 입력 시 로컬모드, "local[4]"는 4 core 사용
- `getOrCreate()`: Spark session이 존재하면 가져오고 없으면 생성
- `createDataset(data, Encoding)`: data(list, RDD, seq 등)를 받아서 encoding 형태에 맞춰 dataset 생성
- `createDataFrame(data, schema)`: data(list, RDD 등)를 받아서 type 형태에 맞춰 dataframe 생성

### JavaSparkContext
- JavaSparkContext: load system properties, Java Collection으로 사용할 수 있는 JavaRDDs로 값 반환
- `parallelize(data, numSlices)`: local Scala collection을 분배하여 RDD 형성 (numSlices만큼 분배)

### Encoders
- `Encoder`: JVM object의 type을 Spark SQL 형태로 변환, 다 static 해야 함
- `Encoders`: `Encoder` 만드는 method 집합
- `bean(class)`: class type의 Java Bean에 대한 `Encoder` 생성

### JavaRDD
- JavaRDD: Java에서 사용하는 RDD
- `foreach(() -> {})`: 각 element 마다 foreach 안의 함수 실행
- `map(() -> {})`: 각 element 마다 map 안의 함수에서 생성하는 값으로 새 RDD 생성
- `collect()`: RDD 안에 있는 모든 elements를 포함한 array 반환
- `glom()`: 각 partition의 elements를 합쳐 하나의 array로 만들고, 각 array들을 모아 하나의 RDD 생성

### JavaPairRDD
- `JavaPairRDD<T1, T2>`: 값을 pair로 가지고 있는 RDD
- `collectAsMap()`: map 형태로 return, 모든 데이터가 메모리에 올라가기에 적은 데이터에 사용하는 것을 추천
- `reduceByKey(() -> {})`: 함수를 통해 같은 key 값들 연산해 merge
- `foldByKey(() -> {})`: `reduceByKey()`와 같음, `zerovalue`를 넣고 이 값을 기본으로 연산 시작, 항등원을 넣어줌
- `combineByKey(createCombiner, mergeValue, mergeCombiners)`
    - `createCombiner`: 각 entry 마다 value에 대한 처리
    - `mergeValue`: value를 Combiner에 추가
    - `mergeCombiners`: Combiner끼리 merge
- `mapValues()`: value 하나씩을 돌아가면서 연산해 출력

### Dataset
- `printSchema()`: `tree` 형태로 schema 출력
- `show()`: 20개까지 data 보여줌
- `filter(condition)`: condition 에 해당하는 값들만 출력
- `where(condition)`: `filter`와 일치
- `select(..columns)`: 보여줄 column 선택
- `foreach`로 탐색시 compile error

### Row
- `Row`: Dataset의 한 행을 의미하는 interface

### DataFrame
- `DataFrame`: `Dataset<Row>` 형태
- `foreach`로 탐색시 list로 return

### Streaming
