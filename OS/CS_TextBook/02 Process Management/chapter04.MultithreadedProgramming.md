# Multithreaded Programming
## Overview
* thread: CPU utilization의 기본 단위
    * thread ID, PC, register set, stack으로 구성
    * code section, data section 및 OS resource (open file, signals) 등은 process 내에서 공유

### 1. Motivation
* 하나의 application이 서버에서 request 처리
    * request 마다 하나의 process 생성 후 배정: 시간 오래걸리고 resource가 많이 필요, 같은 역할을 하는 process가 여러 개 발생
    * thread로 역할 구분: 효율적
* RPC system에 핵심적 역할
* OS kernel도 multithreaded
    * Solaris: interrupt handling, memory 관리, device 관리
    * Linux: 빈 메모리 관리
### 2. Benefits
* Responsiveness (반응성): 일부 thread가 block 되어도 나머지 thread 동작
* Resource sharing: 자원 공유 시, 프로세스 메모리를 공유하기에 추가 메모리가 필요하지 않음
* Economy: process creation 시에 사용되는 memory와 resource 소모가 큼
* Scalability: 동시에 여러 processor에서 돌 수 있음 

## Multicore Programming
* multicore/multiprocessor system: 여러 CPU chip 또는 core를 쓰는 system
* parallel system: 여러 작업을 동시에 수행할 수 있음
* concurrent system: 여러 작업을 동시에 진행을 허용함
### 1. Programming Challenges
* multicore system에서의 프로그래밍 시 고려할 점
    * Identifying tasks
    * Balance: 각 core가 같은 양의 일을 해야 유리
    * Data splitting: 나눠진 작업에 맞는 data 분리
    * Data dependency: task A가 task B의 결과 데이터를 이용할 경우 synchronization 확인 필요
    * Testing and debugging
 
### 2. Types of Parallelism
* Data parllelism: 같은 데이터의 부분 집합을 여러 core에 나누어서 각 core에서 같은 성능을 내도록 하는 것
    * 1~n까지 계산은 A core가, n+1~2n까지의 계산은 B core가
* Task parallelism: 데이터가 아닌 업무를 나누는 것, 서로 다른 업무를 해야 함

## Multithreading Models
* thread 관리 방법
    * user threads: kernel 상위 레벨에서 thread 관리
    * kernel threads: kernel이 thread 관리, 대부분의 OS에서 지원
* user threads와 kernel threads의 관계가 중요
### 1. Many-to-One Model
* 하나의 kernel thread가 모든 user thread 관리
* thread 하나가 blocking 되면 전체 process block
* 한 번에 한 thread만 kernel에 접근
* multicore의 이점을 취할 수 없음
### 2. One-to-One Model
* 여러 kernel이 각각 한 개의 user thread 관리
* user thread 생성 시 kernel thread가 만들어져 performance에 부담
* 위의 이유로 thread 갯수 제한
* Linux, Windows에서 사용
### 3. Many-to-Many Model
* kernel threads 갯수가 user threads 갯수 이하
* user thread가 blocking 시, 대체 kernel thread가 가동
* two-level model
    * 일부 tuser thread는 kernel thread에 1:1 바인딩 가능하게 허용

## Thread Libraries
* thread library 적용하는 방법
    * user space에만 제공하는 것: system call을 부르지 않음
    * kernel-level library를 적용하는 것: system call을 부름
### 1. Pthreads
* user-level, kernel-level library 채택
* POSIX standard API
* thread 행위 적용한 것이 아닌 명시한 것
### 2. Windows Threads
* kernel-level library 채택

### 3. Java Threads
* host OS 방식을 따름
* Runnable class  사용
* start() method
    * JVM에서 memory 할당 및 새로운 thread 초기화
    * run() 메소드 대신 호출
## Implicit Threading
* Programmer 입장에서 thread 사용이 어려움
    * implicit threading: thread 생성과 관리를 compiler에게 맡김
### 1. Thread Pools
* web server에서 thread 문제점
    * 생성 시에 시간이 필요 & 다 사용하면 사라짐
    * 동시에 실행되는 thread 갯수 제한이 설정되어있지 않아 CPU time 혹은 memory issue가 생길 수 있음
    * 이를 해결하기 위한 방법 중 하나가 thread pool
* Thread Pool
    * process 시작 시 일정 갯수의 thread를 미리 생성해 둠
    * request 받으면 한 thread를 awaken 시킴
    * thread 종료 시 pool로 돌아가 await 상태
    * thread 부족 시, 서버는 request를 더 받지 않고 대기 상태
* 장점
    * thread 생성 시간이 안 걸려 빠름
    * thread 갯수 제한으로 과부화 문제 없음
    * 수행할 작업에서 생성 작업을 분리하여 작업 실행을 위한 다른 전략 구사 가능 (task의 주기적 실행 혹은 delayed된 실행)
### 2. OpenMP
* compiler 명령어의 집합이자 parallel computing을 위한 API
* parallel regions을 block으로 지정해 parallel하게 run할 수 있음
### 3. Grand Central Dispatch
* GCD: Mac OS X & iOS의 parallel하게 돌릴 코드를 section으로 분리/구분하는 API이자 run-time library
### 4. Other Approaches
* TBB(Threading Building Blocks): intel에서 만든 library
* java.util.concurrent package

## Threading Issues
### 1. The fork() and exec() System Calls
* fork() 시에 모든 thread도 복사해야하는가?
    * 모든 thread 복사
    * fork() call 한 thread만 복사
* exec() 시에는 어떻게 되는가?
    * fork() 된 모든 thread를 포함한 process가 변경
### 2. Signal Handling
* signal 원칙
    * 특정 event에 의해 발생
    * process에게 전송되어야 함
    * process가 이를 수신하면, 어떠한 방식으로든 처리되어야 함
        * default signal handler: kernel이 처리
        * user-defined signal handler: 처리 방법 지정
    * 종류
        * synchronous signal: process 자신이 발생시킨 signal (illegal memory access, division by zero 등)
        * asynchronous signal: 외부 요인에 의한 signal (Ctrl C, timer interrupt 등)
* multi-thread에서의 signal 처리
    * signal이 적용될 thread에 전달
    * 같은 process의 모든 thread에 전달
    * 같은 process의 특정 thread 들에만 전달
    * 해당 process에 모든 signal을 받는 thread 할당
* Windows에서는 signal을 지원하지 않지만 APC(asynchronous procedure calls)로 emulate 함
### 3. Thread Cancellation
* thread가 끝나기 전 종료시키는 것을 포함
* 발생하는 상황
    * Asynchronous cancellation: 한 thread가 다른 thread를 즉시 종료, 여러 문제 야기 가능
    * Deferred cancellation: 종료 요청 시, 스스로 종료가능한 시점에 종료할 수 있는 권한을 줌 
### 4. Thread-Local Storage
* thread에게 공유되지 않은 자신만의 data copy가 필요할 경우 TLS (thread-local storage) 호출
* static data와 비슷한 느낌이지만 각 thread 별로 unique한 데이터 가짐  
### 5. Scheduler Activations
* many-to-many model에서 kernel thread와 thread library(user thread) 사이의 communication 필요
* Lightweight process (LWP)
    * kernel thread와 thread library 사이의 매개체
    * LWP는 kernel thread에 붙어 있고, OS는 physical processor에서 kernel thread가 동작하도록 schedule
    * user-thread 입장에서 LWP는 동작을 schedule 할 수 있는 virtual processor
    * kernel thread가 block 되면 LWP도 block되고, LWP에 연결된 user thread도 block
    * CPU bounded의 경우 LWP의 갯수가 중요하지 않지만, block이 자주 걸리는 I/O bounded의 경우 LWP 갯수가 user-thread 갯수만큼 있어야 한다.
* scheduler activation
    * user-thread와 kernel thread 사이 communication을 위한 방법/계획
    * kernel이 LWP를 제공하면 application이 사용 가능한 LWP들에 user thread를 schedule 하는 방식으로 운영
    * upcall: kernel이 event를 application에 알림
    * upcall handler: upcall을 처리
    * upcall로 인해 block이 되면, blocking된 thread의 state를 저장하고 blocking thread가 돌아가고있는 LWP를 버리는 새로운 LWP 할당한다. 이후 upcall handler는 나머지 thread들에 대해 새로운 LWP를 할당하여 돌아갈 수 있도록 한다. blocking 작업이 끝나면 kernel은 다시 upcall을 해주고, 이를 담당하는 upcall handler 또한 LWP가 필요하여 새로운 LWP를 할당하거나 user thread 하나를 먼저 차지하여 upcall handler가 LWP에서 돌아갈 수 있도록 한다. 이후 unblocked 된 thread는 사용 가능한 LWP에서 동작하게 된다.

## Operating-System Examples
### 1. Windows Threads
### 1. Linux Threads

## Summary