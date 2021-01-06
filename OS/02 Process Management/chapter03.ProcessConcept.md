# Process Concept

## Process Concept
* 각 OS가 호출하는 것
    * Batch System: Job
    * Time Shared System: User program, Tasks
    * 결국 이들은 OS의 memory allocation 등의 관리를 받아야 하기에 process라고 부르기로 한다.

1. The Process
    * Program in Execution
    * Process의 구성
        * Text Section: program code
        * Data Section: 전역 변수 보관
        * Heap: dynamic하게 할당되는 memory
        * Stack: 임시 data (함수의 인자, return 주소, 지역 변수)
        * Current Activity: Program Counter 및 기타 register 값들
    * Process 자체도 다른 code의 실행 환경이 될 수 있음
        * 사례: JVM

1. Process State
    * New: 막 생성된 상태
    * Running: 실행중인 상태
    * Waiting: 특정 이벤트 발생을 기다리는 상태 (I/O 완료, signal 수신 등)
    * Ready: processor에 할당되기를 기다리는 상태
    * Terminated: 종료한 상태
    * 대부분은 Ready 혹은 Waiting 상태

1. Process Control Block
    * 모든 process는 PCB로 표현
    * PCB 구성
        * Process state: new, running, waiting, ready, terminated 등등
        * Program Counter: 다음 instruction의 위치
        * CPU registers: register에 저장된 값들
        * CPU-scheduling information: process priority, scheduling queue를 가리키는 pointer 등의 값들
        * Memory-management information: base, limit 레지스터 값, page/segment table 값 등 저장
        * Accounting information: CPU 사용 시간, process 번호 등
        * I/O status information: Process에 할당된 I/O device, file 등의 정보
    * Processor에 할당 시 PCB load, context switch시 PCB store

1. Threads
    * 한 Process에서 병렬적으로 돌아가는 다양한 작업

## Process Scheduling
1. Scheduling Queues
    * Queue의 종류
        * job queue: Process가 system에 들어오면 들어감
        * ready queue: Process가 실행 준비가 되면 들어감
            * Ready Queue의 header는 처음과 끝 PCB를 가리킴
            * 각 PCB는 다음 PCB를 가리키는 포인터 가짐
        * device queue: I/O를 요청하면 들어감
    * CPU에서 방출되는 경우
        * I/O request 요청 시
        * fork 후 child 종료까지 기다림
        * interrupt에 의해 CPU에서 강제로 꺼내지는 경우

1. Schedulers
    * Scheduler 종류
        * Long-term scheduler (job scheduler)
            * pool에서 process를 골라 memory에 올리는 역할
            * 덜 자주 일함
            * degree of multiprogramming 결정 (memory 위의 process 수 조절)
            * I/O bound process와 CPU bound process 수를 잘 맞춰줘야 함
        * Short-term scheduler (CPU scheduler)
            * 실행 준비가 된 process를 골라 CPU에 할당하는 역할
            * 자주 일을 함
        * Medium-term scheduler
            * 잠깐 degree of multiprogramming을 줄여야할 때, swap out/in을 관리해 줌

1. Context Switch
    * Interrupt가 발생하면 실행 중인 process의 context를 PCB로 저장했다가 (state save) 돌아올 때 되면 불러와야 함 (state restore)
    * CPU의 process를 바꾸는 것을 context switching이라 함

## Operations on Processes
1. Process Creation
    * 만들어주는 process는 parent process, 만들어지는 process는 child process
    * tree 형태로 process가 만들어짐 (init process 가 root)
    * process identifier(pid)로 process 구분
    * child process의 자원 획득 방식
        * resource를 os로 부터 직접 얻기
        * parent process의 일부만 가지기: process overloading 방지
    * 새 process 생성시 기존 process
        * child와 동시에 실행
        * child 종료까지 기다림
    * 새 process의 주소
        * parent process 복사 (같은 프로그램 같은 data): fork() 만 호출
        * 새로운 program load: fork() 후 exec()
    
1. Process Termination
    * 종료 방법
        * exit()을 통한 종료
        * parent process의 kill
            * 할당된 resource 이용을 초과했을 때
            * child가 맡은 task가 더 필요 없어졌을 때
            * parent가 종료되었을 때 (cascading termination)
        * user의 kill
    * 종료 후 parent가 wait()을 호출하지 않으면 zombie process가 됨
    * parent가 wait()을 호출하지 않고 종료가 되면 orphans 상태가 됨
        * UNIX, LINUX에서 orphan의 parent로 init을 설정
        
## Interprocess Communication
1. Shared-Memory Systems
1. Message-Passing Systems

## Examples of IPC Systems
1. POSIX Shared Memory
1. Mach
1. Windows

## Communication in Client-Server Systems
1. Sockets
1. Remote Procedure Calls
1. Pipes

## Summary