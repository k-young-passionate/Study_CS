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
1. I/O Structure

## Computer-System Architecture
1. Single-Processor Systems
1. Multiprocessor Systems
1. Clustered Systems

## Operating-System Structure

## Operating-System Operations
1. Dual-Mode and Multimode Operation
1. Timer

## Process Management

## Memory Management

## Storage Management
1. File-System Management
1. Mass-Storage Management
1. Caching
1. I/O Systems

## Protection and Security

## Kernel Data Structures
1. Lists, Stacks, and Queues
1. Trees
1. Hash Functions and Maps
1. Bitmaps

## Computing Environments
1. Traditional Computing
1. Mobile Computing
1. Distributed Systems
1. Client-Server Computing
1. Peer-to-Peer Computing
1. Virtualization
1. Cloud Computing
1. Real-Time Embedded Systems

## Open-Source Operating Systems
1. History
1. Linux
1. BSD UNIX
1. Solaris
1. Open-Source Systems as Learning Tools

## Summary