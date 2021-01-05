# Introduction

## Operating System Overview
* Computer Hardware를 관리
* Application Program들의 기반 제공
* Computer User와 Computer Hardware의 중계자 역할
* 이러한 작업들을 각각의 OS는 다양한 방법으로 수행함
    + PC의 OS: 복잡한 게임, 비즈니스 도구, 등등
    + Mobile Computer의 OS: User가 쉽게 접근할 수 있는 프로그램 실행 환경 제공
    + 편리함 또는 효율성, 그 중간을 추구함

## Operating System이 하는 일
* Computer System은 hardware, operating system, application program, user로 나뉨
    + Hardware: System의 기본 computing 자원들 (CPU, Memory, I/O Device)
    + Application Program: User의 연산 문제를 풀어주기 위한 Hardware 사용 방법을 정의한 것 (Word Processor, Spreadsheet, Compiler, Web browser)
    + Operating System: Hardware를 조종하여 Application Program 사이의 자원 사용을 조정

1. User의 시각
    * Interface에 따라 달라진다.
        + PC: 한 사람이 모든 자원 독점, 업무/놀이 성능 최대화가 목표 => 사용의 용이성에 목표
        + Mainframe 혹은 minicomputer로 연결된 terminal: 여러 사람이 사용 => 자원 활용에 목표
        + Nerwork로 연결된 서버에 접속하는 workstation: 각 user에 맞춘 사용성과 자원 활용
        + Smartphone: 개인화된 용도, network로 연결 => 상호작용에 목표
        + Embedded Computer: 기능 수행에 목표, 사용자가 알 수 없게 동작

1. System의 시각
    * OS는 Hardware에 연계된 프로그램
    * Resource Allocator의 역할: CPU time, Memory 공간, file-storage 공간, I/O Devices, 등등 분배
    * Control Program: user program의 실행에서 나오는 에러를 막고 computer의 부적절한 사용을 막는 역할, I/O Device 관리

1. OS 정의
    * 많은 역할과 기능을 담당하고 있다.
    * 토스터기, 차, 배, 우주선, 집, 회사 등에 존재
    * Moore's Law: 18개월에 회로 집적도 두배 예측
    * Hardware는 사용이 쉽지 않기 때문에 Application program이 개발되었고, 공통된 기능(자원 분배, I/O device 조종)을 필요로 하며, 이를 하나의 software로 합친 것이 OS 이다.
    * 확대된 의미로는 kernel이라고 불리는, computer에서 계속 실행되는 프로그램이다.
    * Microsoft는 OS에 너무 많은 기능을 넣어 application 판매자들의 경쟁을 막는다고하여 독과점법의 유죄를 받은 적이 있다.(1998)
    * 하지만 요즘 mobile computer를 보면 kernel 뿐만이 아닌 software framework의 집합인 middleware도 포함되어 있다. 이들은 database, multimedia, graphics 등을 지원한다.

## Computer-System Organization
1. Computer-System Operation
    * 현대의 일반적 목적의 computer system은 하나 이상의 CPU와 device controller들이 bus에 연결되어 shared memory에 접근이 가능하다. 이들은 병렬적으로 실행되며, memory cycle을 두고 경쟁하고, 이를 정렬하기 위해 memory controller가 관여한다.
    * Bootstrap program
        + Computer 시작 시 실행
        + ROM/EEPROM 안의 firmware라는 곳에 저장
        + OS kernel을 memory에 load 하고 시작시킴
    * System Process/System Daemon
        + kernel 밖에서 실행되는 서비스 실행
        + boot time에 memory에 올라와 kernel이 실행되는 동안 실행
        + UNIX에서 첫 system process는 init이라고 불리고, 다른 daemon을 실행
        + 모두 load 되면 fully booted 된 것이고, event 발생을 기다림
    * Interrupt
        + hardware/software에서 발생하는 event
        + Software는 보통 system call(monitor call)을 통해 intterupt 발생
        + Interrupt 발생 시, 하던 일을 멈추고 interrupt handle 명령이 저장된 특정 주소 (보통 시작 주소)를 실행
        + Interrupt 처리가 끝나면 하던 명령 다시 실행
        + Interrupt Vector에 각 interrupt 별 handler 위치 저장, 보통 low memory에 존재

1. Storage Structure
    * Main Memory
        * program들이 Main Memory (RAM, DRAM) 위에서 돌아감
        * Von Neumann 아키텍처의 instruction-execution cycle
            1. Instruction 을 memory로 부터 fetch 후 Instruction Register에 저장
            1. decode 후 operand fetch 해 register에 저장
            1. Operand 실행
            1. 결과를 Memory에 저장
        * 모든 프로그램을 영구히 담기에는 너무 작은 용량
        * power가 꺼졌을 때, 내용물이 사라지는 휘발성
        * Main Memory 특징 때문에 Secondary Storage를 제공
    * Solid-state disks
        + Magnetic disk보다는 빠른 영구 저장소
        
1. I/O Structure
    * Device controller는 여러 device를 담당
    * Device Controller는 local buffer와 special-purpose register를 운영
    * OS는 각 device controller를 이해하고 소통하기 위해 device controller를 보유
    * I/O device 동작
        1. Device Driver loads 시 device driver가 적절한 device controller의 register를 불러옴
        1. Device Controller가 local buffer에 data 전송
        1. 전송 완료 후 동작 끝냈다고 interrupt
    * 위의 방식은 많은 양의 data를 옮기는 데에는 적절하지 않아 DMA (Direct Memory Access) 사용
        + Block 단위로 Memory에 옮기고 Interrupt


## Computer-System Architecture
1. Single-Processor Systems
    * 보통 특수한 목적의 processor를 따로 가짐

1. Multiprocessor Systems
    * 장점
        + 늘어난 throughput: 적은 시간에 더 많은 작업 수행 가능, 하지만 Overhead로 인해 processor 배수만큼 빨라지지는 않음
        + 경제적: 여러 single-processor system들을 모은 것 보다 싸다.
        + 늘어난 안정성: 한 개의 processor가 실패해도 system이 종료되지 않음
            - 어느정도의 안정성을 가진 능력을 graceful degradation이라고 하고, 그보다 더 좋은 system을 fault tolerant라고 하기도 한다.
    * Processing 방식에 따른 type
        + Asymmetric Multiprocessing
            - 각 process는 다른 작업
            - boss processor가 system 조종, schedule & allocate work
        + Symmetric Multiprocessing (SMP)
            - 대부분이 채택
            - 모든 processor가 peer
            - 각자 register와 cache를 가지고 공유된 memory를 읽음
            - 동시에 많은 process가 돌아갈 수 있음
            - 일부 processor가 idle 상태임을 막기 위해 작업을 dynamic하게 배치하고, 이를 위해 특정 자료구조를 공유
    * Memory 공유에 따른 type
        + Uniform Memory Access(UMA): Memory 접근 시간 동일
        + Non-Uniform Memory Access(NUMA): Memory 접근 시간 다름, resource management를 통해 penalty를 줄이고자 함
    * Multicore
        + Single chip 내에 Core를 늘림
        + Multiprocessor의 일종
        + On-chip communication이 빨라 multiple single-core chip 보다 유리
        + 전력 소모도 적음
    * Blade Server
        + 여러 multiprocessor system들이 한 장치에 존재
        + 각자의 OS로 돌아감

1. Clustered Systems
    * Multiprocessor system의 일종
    * loosely coupled
    * 높은 가용성: 한 시스템이 죽어도 실행
    * 운영 방식에 따른 type
        + Asymmetric Clustering: hot-standby server는 active server 관찰하다가 active server fail 시 active server로 전환
        + Symmetric Clustering: 각자 active server 이면서 서로를 관찰, 서버 모두를 사용해 효용성이 높지만 각 시스템이 하나 이상의 process를 실행해야 함
    * 네트워크로 연결 된 여러 컴퓨터 system으로 높은 성능의 computing 환경을 제공
    * Parallelization (병렬화) 기법으로 프로그램을 여러 components 로 나누어 실행하여 매우 빨라짐
    * shared access 에서 발생하는 문제를 해결하기 위해 Distributed Lock Manager (DLM)을 사용
    * Storage-Area Networks (SANs)를 통해 어떤 host에서도 똑같은 program을 실행할 수 있음
    

## Operating-System Structure
* OS는 프로그램이 실행될 환경을 제공
* Multiprogramming을 기본으로 탑재 함
    - CPU Utilization을 높임
* Memory 용량 문제로 초반 job들은 disk의 job pool에서 존재
* Memory 위의 job도 job pool의 subset으로 존재
* Time sharing (multitasking)을 통해 동시에 user가 접근할 수 있게 함
* Process: Memory 위에 올라가 실행되는 Program
* job scheduling을 통해 job들의 실행 순서 결정
* Swapping과 Virtual Memory를 통해 적절한 response time 보장
* Virtual Memroy 장점
    + Physical Memory보다 더 많은 프로그램 실행 가능
* Time Sharing System은 file system에서도 제공된다.

## Operating-System Operations
* 현대 OS는 Interrupt driven이다.
* Trap(Exception): Software generated interrupt, error 혹은 system에 요청을 할 때 발생
1. Dual-Mode and Multimode Operation
    * OS의 정상 동작을 위해 OS 실행 코드와 user-defined code를 분리해야 함
    * Operation Mode: mode bit를 통해 관리
        + User Mode
            - OS 실행 코드 동작 불가
            - System call시 trap이 동작하고, kernel mode로 바뀜
        + Kernel Mode
            - OS 실행 코드(Privileged instructions) 동작 가능
    * Virtual Machine Manager (VMM)을 위한 mode 존재
        + user mode와 kernel mode 사이의 권한
        + vm 생성 및 관리, cpu state 변경 등에 활용
    * System Call
        + user program이 os 코드를 동작하도록 요청하는 도구
        + 실행 시 software interrupt로 처리


1. Timer
    * 일정 시간 소요 시 interrupt
    * 무한 루프에 빠진 프로그램 구제
    * Variable timer는 fixed-rate clock과 counter로 구현

## Process Management
* Process: 실행 중인 프로그램
    * Resource 필요: CPU time, memory, files, I/O devices 등이 실행되며 배치
* Single-threaded process는 하나의 Program Counter 사용
* Multi-threaded process는 각자 pc를 사용
* OS의 임무
    * CPU의 process와 thread 스케쥴
    * user/system process를 생성/삭제
    * Process 정지/재개
    * process 동기화 방법 제공
    * process 통신 방법 제공

## Memory Management
* user와의 반응성을 높이기 위해 최대한 많은 프로그램을 memory에 남기려고 함
* OS의 역할
    * Memory의 각 부분의 사용 여부와 점유자 확인
    * Memory에 들어올 data와 나갈 data 결정
    * Memory 공간 할당 및 반환

## Storage Management
1. File-System Management
    * file
        * 연관된 정보의 모임, 보통 데이터(숫자, 글자, binary)와 프로그램을 표현함
        * disk 등에 저장됨
        * user의 접근 권한 관리 (read, write, append 등)
        * directory를 통해 사용 용이화
    * OS의 file 관리
        * 파일 생성/삭제
        * directory 생성/삭제
        * 파일과 directory 조작을 위한 기본 기능 지원
        * secondary storage에 mapping
        * 안정적인 storage에 백업
1. Mass-Storage Management
    * Main Memory의 휘발성과 적은 용량으로 백업을 위해 secondary storage를 사용한다.
    * OS의 disk management
        * 빈 공간 관리
        * 저장공간 할당
        * disk scheduling
    * 컴퓨터의 전체 속도는 disk의 하위 시스템과 그의 알고리즘에 달려있다.
    * tertiary storage: Magnetic tape, CD, DVD 등
        * secondary storage에 용량이 부족할 경우 tertiary storage에 저장
        * WORM (Write once, Read many times), RW(Read Write) 형식
        * mount, unmount를 통해 OS가 관리
1. Caching
    * Main Memory에서 사용할 것들을 Cache에 저장해 둠
    * Cache 용량이 매우 한정되어있기에 cache 관리가 중요
    * Register > Cache > Main Memory > SSD > MD 순으로 빠름
    * Cache Coherency: 같은 데이터가 여러 곳에 있을 수 있는데, Multiprocessor 환경에서는 한 값이 바뀌면 다른 곳에도 적용 되어야 한다.
1. I/O Systems
    * OS는 특정 하드웨어의 특수성을 유저로 부터 숨겨야 하는 목적이 있음
    * I/O Subsystem
        * buffering, caching, spooling을 통한 memory 관리
        * device-driver 인터페이스
        * 특정 하드웨어 디바이스의 driver

## Protection and Security
* Computer system은 여러 user를 가지고 동시에 여러 process를 실행시켜야 하며, 이 때문에 data에 대한 access는 규제되어야 한다.
* Protection
    * process 혹은 user의 resource 접근 통제 방법
* Security
    * 내외부의 공격으로부터 system 보호
* 사용자 구분
    * user IDs
    * Security ID (SID): Windows
* 사용자 그룹을 통한 공통 권한 부여
    * group IDs 부여
* Effective UID
    * 특수한 파일 사용을 위해 잠시 권한 상승이 필요할 때 해당 파일의 소유자 uid 를 통해 접근
## Kernel Data Structures
1. Lists, Stacks, and Queues
    * Array: 다양한 size의 data 관리 및 지속적 삭제시 불리
    * List: 특정 순서대로 data 접근, 삽입/삭제에 유리, item 검색에 불리
    * Stack: LIFO 구조, push/pop으로 데이터 관리
    * Queue: FIFO 구조
1. Trees
    * data를 계층화하여 나타내는 자료구조
    * general tree: 수많은 자식 가짐
    * binary tree: 자식이 좌, 우 하나씩
    * binary search tree: left child보다 right child가 더 커야함, worst의 경우 O(n)
    * balanced binary search tree: bst의 worst case 해결, O(lg n)
1. Hash Functions and Maps
    * Hash Function: data를 받아 특정 연산 후 numeric 값으로 반환하는 함수, 여기서 나온 값을 index로 사용
    * Hash collision: 두 개의 서로 다른 input에서 같은 output이 나오는 경우
    * hash map: key-value 쌍을 hash function을 이용해 결합
1. Bitmaps
    * n binary digits로 이루어진 string
    * disk block을 가리키기 좋음
    

## Computing Environments
1. Traditional Computing
    * Web 기술과 WAN(Wide Area Network)으로 전통적인 computing의 영역이 확대되고 있다.
    * 각 회사는 자신의 내부 서버에 접근할 수 있는 Web인 Portal을 구축하고 있다.
    * Network Computer(thin clients)는 기존의 workstation으로 쓸 수 있다.
    * Mobile 컴퓨터는 회사 정보를 이용하기 위해 PC와 동기화 할 수 있다.
    * 무선 네트워크로 연결 가능하다.
    * 보안 위반으로부터 네트워크를 보호하기 위해 firewalls 사용
1. Mobile Computing
    * Smartphone 혹은 tablet computer 의미
    * 가볍고 휴대성이 좋다.
    * IOS와 Android 두 개의 OS 존재
1. Distributed Systems
    * 물리적으로 떨어져있는 computer system의 집합
    * 높은 연산 능력, 저장 능력, 안정성, 높은 기능성
    * Network를 통해 소통, 보통 TCP/IP protocol 사용
    * LAN (Local Area Network): 컴퓨터 끼리의 연결
    * WAN: 빌딩, 도시, 나라 등을 연결
    * Network operating system: network를 통한 파일 공유와 같은 기능을 제공하는 OS
1. Client-Server Computing
    * Client의 요청을 Server에서 처리
    * compute-server system: client의 요청을 받아 처리해 결과를 전달해 줌
    * file-server system: client가 file을 생성/변경/읽기/삭제 가능 (CRUD)
1. Peer-to-Peer Computing
    * Client와 Server가 따로 구분되지 않음
    * 여러 node들이 network에 가입되어 있음
    * network 가입 시 중앙화된 lookup 서비스 혹은 broadcasting을 통해 원하는 서비스를 제공하는 node를 찾을 수 있음
    * Skype는 P2P computing의 대표적 예시, voice over IP (VoIP) 기술을 통해 client 끼리 통신 가능
1. Virtualization
    * OS 내에서 application의 개념으로 또다른 OS를 허용하는 기술
    * Emulation을 포함하는 개념
    * Emulation: source CPU type과 target CPU type이 다를 때 사용
        * Interpretation: native code가 아닌 중간 형태로 바꿔주는 것, emulation 중 하나
1. Cloud Computing
    * network를 통해 연산능력, 저장공간, 어플리케이션 서비스 등을 제공하는 것
    * Public cloud: Internet을 통해 돈을 내고 이용할 수 있는 cloud
    * Private cloud: 회사 자신이 이용하기 위해 운영하는 cloud
    * Hybrid cloud: public/private cloud가 섞인 cloud
    * SaaS: Internet에서 이용할 수 있는 application (word processor, spreadsheet 등)
    * PaaS: Internet을 통해 application을 만들 수 있는 소프트웨어 하위 시스템 (database 등)
    * IaaS: Internet을 통해 사용 가능한 server 혹은 storage (storage 등)
1. Real-Time Embedded Systems
    * 엔진과 로봇부터 DVD와 전자레인지까지 많은 곳에서 찾을 수 있음
    * 특정 기능을 수행함
    * 기본적인 기능만 실행하기에, OS는 한정된 기능만 제공
    * Real-time system: 엄격한 시간 요구사항이 요구되는 경우 사용
    * Real-time Embedded systems: sensor 등을 의미

## Open-Source Operating Systems
* Source code로 제공되는 OS
* Closed source와 반대
1. History
    * Binary code 형태의 OS 제공의 이점
        * 경쟁사로부터의 도용 보호
        * program들의 Copy protection 혹은 DRM 관리 시 유리
    * Richard Stallman이 1983년 GNU project 시작
        * Free Software Foundation(FSF) 설립
        * GNU General Public License (GPL)
        * UNIX 호환 도구 개발
1. Linux
    * 핀란드의 Linus Torvalds란 학생이 1991년 GNU compiler를 통해 UNIX-like kernel 개발
    * 수많은 분기/custom build 등을 야기했다.
    * 주요 분기는 RedHat, SUSE, Fedora, Debian, Slackware, Ubuntu 가 있다.
1. BSD UNIX
    * UC Berkley가 개발해 AT&T의 UNIX로 1978년 시작, 추후 open source로 1994년 등장 
    * Linux 보다 깊은 역사를 지님
    * FreeBSD, NetBSD, OpenBSD, DragonflyBSD 등 존재
    * Mac OS X의 핵심 kernel인 Darwin의 기반
1. Solaris
    * Sun Microsystems에서 개발한 상업 용도의 UNIX 기반의 OS