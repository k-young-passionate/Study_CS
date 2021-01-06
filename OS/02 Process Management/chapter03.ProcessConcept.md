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
* process간 협력해야 할 이유 == Interprocess Communication
    * Information sharing: 같은 정보에 관심있어 할 수 있기 때문
    * Computation speedup: subtask로 나누어 parallel하게 처리
    * Modularity: system function을 여러 process/thread로 나누는 방식으로 system을 구축할 때 용이
    * Convenience: 사용자는 동시에 많은 작업을 진행할 수 있음
1. Shared-Memory Systems
    * 한 process에서 shared memory segment를 만들고, 통신하고자 하는 process가 해당 shared memory segment를 자신의 address space에 추가해야 함
    * 같은 장소에 안쓰기로 하는 규약 필요
    * Producer-Consumer Problem
        * producer: 정보 생산, server와 같은 역할
        * consumer: 정보 소비, client와 같은 역할
        * buffer를 사용해 producer가 채우고 consumer가 지워야 함
    * Buffer 종류
        * unbounded buffer: buffer size 제한 없음, producer는 계속 생산 가능
        * bounded buffer: buffer size 제한, producer는 buffer가 차면 빌 때까지 기다림
1. Message-Passing Systems
    * 공용 공간 없이 협력 가능
    * send()와 receive() 함수 사용
        * direct / indirect
        * synchronous / asynchronous
        * automatic / explicit buffering
    1. Naming
        * direct: 상대 name 명시 필요
            * addressing 방식
                * symmetry: sender와 receiver 모두 상대 name 명시
                * asymmetry: sender만 상대 name 명시
            * 특성
                * pair 마다 성립 가능
                * 두 process만 허용
                * 두 process간 딱 하나의 link
        * indirect: mailbox나 port를 통해 전송/수신
            * OS의 mailbox 관리
                * mailbox 생성
                * messagebox를 통한 송수신
                * mailbox 삭제
            * 특성
                * shared mailbox가 있어야 link 성립
                * 두 개 이상의 process에 동시에 link 성립 가능
                * 두 process 사이에 여러 link 가능
    1. Synchronization
        * Blocking send: 상대가 보낸 것을 받을 때 까지 기다림
        * Nonblocking send: 상대에게 보내고 하던 일 함
        * Blocking receive: 자신에게 수신될 때까지 기다림
        * Nonblocking receive: 수신을 확인하고 비어있으면 null을 받고 하던 일 함
    1. Buffering
        * Zero capacity: 용량이 없을 경우 (== 1일 경우) receiver가 받기까지 sender는 block
        * Bounded capacity: 용량이 제한되어 있을 경우, 차있지 않으면 전송, 차있으면 sender는 queue가 빌 때까지 block
        * Unbounded capacity: 용량 제한이 없어서 sender는 block되지 않음
## Examples of IPC Systems
1. POSIX Shared Memory
1. Mach
1. Windows

## Communication in Client-Server Systems
1. Sockets
    * network 상에서 communication을 위한 endpoint로 정의됨
    * IP address와 port 번호로 구분
    * Java의 socket
        * TCP (Connection Oriented): Socket class
        * UDP (Connectionless): DatagramSocket class
        * Multicast(여러 명에게 보냄): MulticastSocket class (DatagramSocket의 subclass)
1. Remote Procedure Calls
    * network connection을 통한 system끼리의 호출
    * IPC랑 비슷한 서로 다른 system 간의 message 기반 통신
    * 각 message는 remote system의 RPC daemon으로 보내지고, 특정 함수를 실행시키는 identifier와 해당 함수에 넘기려는 parameter를 보낸다. 함수가 실행 되면 결과가 requester에게 message로 보내진다.
    * System의 endian과 같이 표현 방식이 다를 수 있기에 이에 독립적인 external data representation (XDR) 사용
    * Network 환경에 불안정성
        * at most once: server가 여러 번 받아도 단 한 번만 실행
            * at most once 전략 사용 시, server는 history를 보유하다가 이미 실행된 message는 무시하여 client가 여러번 보내도 단 한 번 실행된다.
        *  exactly once: server가 단 한 번만 받을 수 있도록 실행, 더 선호되는 전략
            * exactly once 전략 사용 시, 서버가 request를 놓치는 상황을 만들면 안 된다. 이를 달성하기 위해서는 at most once protocol을 사용해야 하고, client에게 상황을 알려주기 위해 ACK을 이용한다.
    * RPC call의 경우 port를 통해 procedure call의 name을 정의해야 하는데 port를 미리 알 수 없음
        * binding information을 고정된 port 번호로 미리 정의해놓고 알려준다.
        * rendezvous(matchmaker) daemon을 고정된 RPC port로 정의하여 요청 시에 원하는 port 번호로 안내해준다.
    * 분산 file system 적용에 유리

1. Pipes
    * 두 process 간의 배관 역할
    * 설계 시 고려되어야 할 점
        1. 양방향(bidirection) or 단방향(unidirection) communication
        1. 양방향 communication이 가능하면 한 번에 한 방향(half-duplex) or 동시에 두 방향(full-duplex)
        1. communication 하는 process 사이 관계가 존재해야 하는가
        1. network를 통해 communication이 가능해야 하는가
    1. Ordinary Pipes (Anonymous Pipes)
        * producer-consumer 방식 차용: producer가 pipe 반대쪽에 write
        * 단방향, half-duplex, 관계가 존재해야 함
    1. Named Pipes (FIFO)
        * 양방향, half-duplex (Window 에선 full-duplex), 관계 없어도 됨, 여러 process가 사용 가능
        * 생성 시 file로 존재
        * 외부 machine과 연동 시, socket 필요
## Summary