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

