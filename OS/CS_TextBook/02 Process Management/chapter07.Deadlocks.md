# Deadlocks
* 요청한 resource가 다른 process에 의해 점유되어있는 상황에서 계속 멈춰있는 상황

## System Model
* Process의 resource 사용 과정
    1. Request: 자원 요청, 얻기 위해 기다려야 함
    1. Use: 자원 사용
    1. Release: 자원 반납
## Deadlock Characterization
### 1. Necessary Conditions
* Deadlock은 다음과 같은 상황에서 발생
    1. Mutual Exclusion: 한 번에 하나씩 접근
    1. Hold and wait: 획득할 수 있는 자원은 쥐고 남은 필요 자원 기다림
    1. No preemption: 자원 반납은 process 자의적으로 반납
    1. Circular wait: 여러 process가 각각 다른 process가 가진 자원을 필요로 함, hold and wait 이 전제된 상황에서 벌어짐
### 2. Resource-Allocation Graph
* 각 resource와 process 관계 표시
* request edge: Pi -> Rj (Pi가 Rj를 요구)
* assignment edge: Rj -> Pi (Pi가 Rj 점유)
* 여기서 cycle이 생기면 circular wait

## Methods for Hnadling Deadlocks
* 해결 방법
    * deadlock을 미리 방지하거나 회피하는 protocol을 사용
    * system이 탐지해 회복
    * deadlock이 일어나지 않는다고 가정: 대부분의 OS가 선택

* Deadlock prevention: Deadlock 조건 중 적어도 하나를 갖지 못하는 것을 보장하는 method 제공
* Deadlock avoidance: OS가 resource와 process 관계 정보를 받는 상황 필요

## Deadlock Prevention
### 1. Mutual Exclusion
* 무조건 지켜야하는 조건
### 2. Hold and Wait:
* resource requesting 시 다른 resource를 갖고 있으면 안 되는 것을 보장 해야 함
* 방법
    1. 실행 직전 모든 resource를 불러오는 방법: resource가 memory에 배치 후 사용하지 않는 기간이 늘어나 utilization이 떨어짐
    1. disk에서만 resource를 불러오는 방법: 인기있는 resource들을 요구 시 이미 다른 process에 배치되어 있을 수 있어 starvation이 가능함
### 3. No Preemption
* 점유 후 대기 중인 Process의 Resource를 뺏고, 다시 가져오면 재시작
* Mutex lock과 semaphore 때문에 적용이 잘 안되고 있음
### 4. Circular Wait
* 각 process가 번호가 더 높은 순서대로 resource를 요구하는 방식으로 해결
* FreeBSD에서는 witness 사용: Lock ordering
* Dynamic하게 lock이 요구되는 상황에서 deadlock prevention이 보장되어지지 않음

## Deadlock Avoidance
* Deadlock Prevention은 성능 등에서 side effect 존재
### 1. Safe State
* safe sequence만 존재하는 상황
    * safe sequence: 요청되는 resource가 할당 가능한 상태 (추후 release 되는 resource도 고려)
* safe state는 deadlock이 아닌 상태이지만, 모든 unsafe state가 deadlock 상태는 아니다.
### 2. Resource-Allocation-Graph Algorithm
* 직선 화살표: 자원 점유/요청 상태
* 점선 화살표: 필요하지만 아직 요청하지 않은 상태
### 3. Banker's Algorithm
* 용어 정리
    * Available: 현재 사용가능한 resource 갯수
    * Max: 각 Process의 최대 resource 요청량
    * Allocation: 현재 process에 배정된 resource 갯수
    * Need: process에게 더 필요한 resource 갯수, ```Need[i][j] = Max[i][j] - Allocation[i][j]```
1. Safety Algorithm
    1. ```Work = Available; Finish[i] = false;```로 설정
    1. ```Finish[i] == false``` 이고 ```Need[i] <= Work``` 인 process 찾기
    1. ```Work=Work+Allocation[i]```로 사용 가능한 자원 release 후 ```Finish[i]=true```
    1. 모든 process의 ```Finish[i]=true```이면 system은 safe state
1. Resource-Request Algorithm
    1. ```Request[i] <= Need``` 이면 step2로, 아니면 error 발생
    1. ```Request[i] <= Available```이면 step3로, 아니면 wait
    1. ```
        Available=Available-Request
        Allocation[i]=Allocation[i]+Request[i];
        Need[i]=Need[i]-Request[i];
        ```
        위 처럼 자원 할당 실행

## Deadlock Detection
* system이 deadlock-prevention과 deadlock-avoidance 알고리즘 미 채택 시, deadlock 발생 가능
### 1. Single Instance of Each Resource Type
* 각 resource가 하나의 instance만 가질 때, wait-for graph라 불리는 resource 할당 그래프의 변형된 것을 detection에 사용할 수 있다.
* wait-for graph가 cycle을 보이면 deadlock 상황
* wait-for graph를 유지하여 주기적으로 해당 algorithm을 호출해 cycle을 찾아야 함 (O(n^2))
### 2. Several Instances of a Resource Type
* 각 resource type에 여러 instance가 존재할 때, wait-for graph 사용 불가
* Banker's algorithm과 비슷한 자료구조 사용
    * Available: 각 type의 사용 가능한 instance 갯수 저장 vector
    * Allocation: 현재 process당 배정 된 resource instance 갯수를 저장하는 n*m matrix
    * Request: 현재 process당 더 필요로하는 resource instance 갯수를 저장한 n*m matrix
* 방법
    1. ```Work=Available``` 후 자원 배정이 되어있지 않은 process에 대해서는 ```Finish[i]=true``` 나머지는 ```Finish[i]=false```
    1. ```Finish[i]==false && Request[i] <= Work```인 process ```i```를 찾는다. 없으면 step 4로
    1. ```
        Work=Work+Allocation[i]
        Finish[i]=true
        ```
        자원 release 후 process 종료 및 step 2로 이동
    1. 어떠한 ```Finish[i]==false```를 만족시키는 ```i``` 존재 시, Pi는 deadlock 상태
### 3. Detection-Algorithm Usage
* detection 알고리즘을 사용 시기 결정 조건
    1. 얼마나 deadlock이 자주 일어나는가
    1. deadlock이 벌어지면 얼마나 많은 process가 영향을 받는가
* deadlock이 자주 발생하면 잦은 detection 필요
* process의 resource에 대한 request마다 detection 하면 deadlock의 원인을 찾을 수 있음 but 많은 overhead => 적당한 주기로 부르면 해결 가능하지만 deadlock의 원인 찾기 힘듬

## Recovery from Deadlock
### 1. Process Termination
* Abort all deadlocked processes: deadlock cycle을 부수는 것을 보장하지만 비싼 비용
* Abort one process at a time until the deadlock cycle is eliminated: 하나 부수었을 때 못 찾으면 detection algorithm 호출해야 해서 비쌈
* minimum cost로 하기 위해 고려해야할 점
    1. process의 priority
    1. process가 연산된 시간 및 더 필요한 시간
    1. 사용한 resource의 type 및 갯수
    1. 더 필요한 resource 갯수
    1. 종료해야하는 process 갯수
    1. process가 interactive/batch 인지 여부
### 2. Resource Preemption
* 해결해야 할 issue들
    1. Selecting a victim
        * Cost 고려해야 함
    1. Rollback
        * running process의 state를 보존해주어야 함
    1. Starvation
        * 계속 victim이 되면 starvation 될 수 있으므로 고려 필요
## Summary