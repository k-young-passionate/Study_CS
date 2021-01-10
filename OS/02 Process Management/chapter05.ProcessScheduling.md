# Process Scheduling
## Basic Concepts
* Single Processor system에서는 한 번에 한 process만 돌아가지만, multiprogramming의 목적은 CPU utilization을 최대화 하기 위해 process를 상시 돌려야 함
* I/O request 같은 wait 전까지 계속 실행시키고, wait 동안의 idle은 CPU util을 낮추기 때문에 wait 시간 동안 다른 process를 CPU에 넣음
1. CPU-I/O Burst Cycle
    * CPU scheduling의 성공은 CPU 실행과 I/O wait의 cycle의 조합에 따라 달라짐
    * Process는 CPU burst와 I/O burst의 반복 후 CPU burst로 끝남
1. CPU Scheduler
    * short-term scheduler가 CPU에서 실행 될 process 선택
1. Preemptive Scheduling
    * CPU scheduling 결정이 일어나는 상황
        1. process 하나가 waiting state가 되었을 때
        1. process 하나가 ready state가 되었을 때
        1. process 하나가 waiting에서 ready state가 되었을 때
        1. process 하나가 끝났을 때
    * 1, 4번의 경우는 nonpreemptive/cooperative 상황: 새 process에게 자발적으로 CPU 자원을 내어 줌
    * 2, 3번의 경우는 preemptive 상황: 실행되는 process의 CPU 자원을 뺏어야 함
    * preemptive scheduling이 필요하지만, 다음과 같은 상황에서는 race condition에 처하게 된다.
        * process A가 X 정보를 updating 중에 process B로 switch 되어지고 X를 접근하는 상황에서는 inconsistent 상황에 처하게 됨
        * system call이 끝나거나 I/O block이 발생하면 switch하는 kernel-execution model이 있지만, interrupt는 언제든 일어날 수 있기에 real-time computing에 적합하지 않음
1. Dispatcher
    * short-term scheduler에 의해 선택된 process에게 CPU 통제권을 주는 module
    * 역할 상세
        * context switch
        * user mode로 switch
        * program을 재시작 할 수 있도록 user program에 적절한 위치로 jump
    * dispatch latency: dispatcher에 의해 한 process가 끝나고 다음 process가 시작할 때 걸리는 시간

## Scheduing Criteria
* CPU utilization: CPU의 바쁜 정도, 실제 system에서 40~90% 사이로 운용됨
* Throughput: 특정 시간 동안 할 수 있는 일의 양
* Turnaround time: 총 소요 시간 (memory load, ready queue 대기, CPU 실행, I/O 실행)
* Waiting time: ready queue에서 대기하는 시간의 합
* Response time: 요청 후 첫 응답이 오는데까지 걸리는 시간, interactive system에서 turnaround time의 중요도가 낮아져서 대신 함

## Scheduling Algorithm
1. First-Come, First-Served Scheduling (FCFS)
    * 먼저 요청된 process를 실행
    * nonpreemptive: terminating 혹은 I/O wait까지 CPU 자원 점유
    * 평균 waiting time이 김
1. Shortest-Job-First-Scheduling (SJF)
    * 가장 적은 CPU burst time을 갖는 process 먼저 실행
    * nonpreemptive
    * CPU burst time을 예상하지 못하는 문제점
    * long-term scheduling에 사용
    * preemptive 버젼은 shortest-remaining-time-first (SRTF)
1. Priority Scheduling
    * Priority가 예상되는 CPU burst의 역으로 매겨지고, priority 순으로 실행
    * nonpreemptive, preemptive 방식 모두로 운영 가능
        * nonpreemptive: 실행 중인 process의 CPU 자원 점유 해제 시, 우선순위가 가장 높은 process가 점유
        * preemptive: 실행 중인 process를 포함하여 우선순위가 가장 높은 process가 점유
    * 특정 process가 자원을 점유하지 못하는 starvation 혹은 indefinite blocking이 발생할 수 있음
        * waiting이 길어질 수록 priority를 높여주는 aging을 통해 해결할 수 있음 

1. Round-Robin Scheduling (RR)
    * FCFS와 비슷하지만, time quantum/time slice로 잘라 공평하게 번갈아 CPU 자원 분배
    * preemptive
    * time-sharing system을 위한 설계
    * time quantum에 따라 성능이 바뀜
        * time quantum이 커지면 FCFS와 같은 상황
        * time quantum이 작아지면 수많은 context switch 발생하여 느려짐
        * 현대 system은 10~100 milliseconds 정도의 time quantum 설정

1. Multilevel Queue Scheduling
    * 각기 다른 알고리즘을 가진 여러 queue로 나누고 각 queue에게 우선순위를 매기는 방식
        * 예시: system > interactive > interactive editing > batch
    * process는 한 번 배치된 queue를 바꿀 수 없음 (priority 고정)
    * foreground (interactive) process와 background(batch) process group이 구분 될 때 유용
        * (예시) foreground는 RR, background는 FCFS 방식
    * 여러 방식으로 운영 가능
        * priority를 철저히 지키는 방식
        * priority에 따른 time slice의 차등을 두는 방식

1. Multilevel Feedback Queue Scheduling
    * Multilevel Queue Scheduling에서, process가 배치된 queue를 바꿀 수 있는 여지를 줌
    * CPU time을 많이 소모할 수록 낮은 priority, I/O-bound 이거나 interactive process의 경우에는 높은 priority 부여, aging을 통해 starvation 방지
    * 구성
        * queue의 갯수
        * 각 queue의 scheduling algorithm
        * higher/lower queue으로 재배정을 판별하는 method
        * process 요청 시 들어갈 queue 선정할 method
## Thread Scheduling
1. Contention Scope
    * Process-Contention Scope (PCS): many-to-one, many-to-many model에서 thread library가 user-level thread를 사용 가능한 LWP에 schedule되도록 결정
        * 같은 process 내에서 thread들끼리의 CPU 경쟁이 벌어짐
        * 보통 user에 의해 부여된 priority에 의해 작동
    * System-Contention Scope (SCS): kernel-level thread가 CPU에 schedule되도록 결정
        * 모든 thread끼리의 CPU 경쟁이 벌어짐
        * one-to-one model(Windows, Linux, Solaris) 에서는 SCS만 사용

1. Pthread Scheduling
    * PTHREAD_SCOPE_PROCESS는 PCS, PTHREAD_SCOPE_SYSTEM은 SCS scheduling 채택

## Multiple-Processor Scheduling
* Multiple Processor는 load sharing이 가능해짐
1. Approaches to Multiple-Processor Scheduling
    * Asymmetric multiprocessing: Master가 scheduling, I/O processing, 등의 활동을 맡고, 나머지 processor가 user code를 실행하는 방식
    * Symmetric multiprocessing(SMP): 각 processor가 private한 ready queue를 갖고 self-scheduling
1. Processor Affinity
    * process가 점유하던 processor가 아닌 다른 processor로 옮겨갈 경우 cache migration의 overhead가 발생하므로 최대한 같은 processor로 배정되려고 하는 경향
    * 종류
        * soft affinity: 같은 processor에서 실행되려고는 하지만 보장은 안 되는 것
        * hard affinity: 같은 processor에서 실행 보장
1. Load Balancing
    * SMP system에서 workload를 processor마다 균등하게 분배하는 것
    * 방법
        * push migration: 각 processor의 load를 주기적으로 확인하여 덜 바쁜 processor에 overloaded 된 processor의 process를 옮김
        * pull migration: idle processor가 busy processor의 process 가져옴
    * processor affinity에 반할 수 있음

1. Multicore Processors
    * memory stall: processor가 memory에 접근 시에 꽤나 많은 시간 waiting
        * cache miss 등으로 인해 발생
        * 한 thread가 memory stall 중일 경우 core는 다른 thread로 교체하는 방식으로 해결
    * processing core에서 multithreading 하는 법
        * coarse-grained multithreading: memory stall이 일어날 때까지 processor에서 실행 후 switch
            * context switching이 상대적으로 적게 발생 
        * fine-grained (interleaved) multithreading: instruction cycle마다 switch 
            * hardware logic을 추가하여 context switch overhead를 최대한 줄임
    # 이 부분 복습 필요
    * 2 level의 scheduling 필요
        * 각 processor에서 어떤 software thread를 고를지 결정하는 level
        * 각 core가 어떤 hardware thread를 실행시킬지 결정

## Real-Time CPU Scheduling
* Real-Time system 구분
    * Soft real-time system: critical한 real-time process가 언제 schedule 될지 보장이 안 됨 (우선 순위는 보장)
    * Hard real-time system: deadline 안에 서비스 되지 않으면 치명적 문제
1. Minimizing Latency
    * event latency: event 발생 시점부터 service 되는 시점까지 걸린 총 시간
        * Interrupt latency: interrupt가 CPU에 도달해서 해당 처리 서비스 진입 시점까지의 시간, hard real-time system에서는 기준에 꼭 맞춰야 함
        * Dispatch latency: dispatcher가 한 process를 끝내고 다른 process를 시잘할 때까지 걸리는 시간
    * dispatch의 conflict phase 구성 요소
        * kernel에 의해 실행 중인 모든 process보다 선점
        * 높은 priority의 process에 의해 낮은 priority process의 자원 점유 해제
1. Priority-Based Scheduling
    * Realtime system에서는 respond time이 중요하기에 priority based한 preemption이 적용 되어야 함
    * soft real-time의 기능만 보장해주고, hard real-time의 경우 추가적인 알고리즘 필요
    * schedule 되어야 하는 process 특징
        * process는 periodic 하다고 여겨져야 함
    * Admission control algorithm: process 들여오고 시간 내에 끝남 보장
1. Rate-Monotonic Scheduling
    * periodic task를 고정된 prioriy로 두는 preemption 방식 scheduling
        * period가 짧으면 priority 높음
1. Earliest-Deadline-First Scheduling (EDF)
    * deadline에 따라 priority를 dynamic하게 배정
        * 더 짧은 deadline 보유 시 priority 높임

# 이 부분 복습 필요
1. Proportional Share Scheduling
    * 모든 application에 대해 T를 share


1. POSIX Real-Time Scheduling

## Operating-System Examples
1. Examples: Linux Scheduling
1. Examples: Windows Scheduling
1. Examples: Solaris Scheduling

## Algorithm Evaluation
* 평가 기준
    * CPU utilization maximize + maximum response time 1 sec
    * throuput maximize + turnaround time이 평균적으로 execution time에 선형 비례
* Analytic evaluation
    * 주어진 algorithm과 system workload로 식이나 점수를 생산해 성능 평가
1. Deterministic Modeling
    * Analytic evaluation의 일종
    * workload를 미리 가정하여 성능 평가
1. Queueing Models
    * process가 도는 방식이 날이 갈수록 달라져 deterministic modeling을 적용하기 힘들어짐
    * CPU 및 I/O burst의 분배로 결정할 수 있음
1. Simulations
    * 직접 simulation 해보면 더 정확
    * trace tapes를 이용해 이벤트별 기록을 남겨 더 정확한 판단 가능
    * expensive, 시간 오래걸림
1. Implementation
    * 실제 OS에 적용해서 동작하는 것을 보는 것이 가장 정확
## Summary