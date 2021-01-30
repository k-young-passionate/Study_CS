# I/O Systems

## Overview
* Computer에 연결된 device의 조종은 OS 설계자의 중요 고민거리였음
    * I/O devices는 기능과 속도에 있어 많이 다르고, 조종하기 위해 다양한 method 필요하기 때문

* 트렌드에 역행하는 I/O-device 기술의 두 가지
    * software와 hardware interface의 표준화
    * 점점 더 다양해지는 I/O devices

* device drivers를 통해 각각 다른 device들을 encapsulate하여 해결

## I/O Hardware
* `port`: connection point를 통한 device communication을 이루어줌
* `bus`: 여러 devices가 공유하는 wire
    * `PCI bus`: processor-memory subsystem을 fast devices에 연결
    * `expansion bus`: 상대적으로 느린 devices들을 연결 (키보드, serial port, USB port)
    * `Smallo Computer System Interface` (`SCSI`): 주변 기기를 연결할 때 직렬로 연결하기 위한 표준
    * `PCI Express`(`PCIe`): 16GB/sec
    * `HyperTransport`: 25GB/sec
* `controller`: port, bus, device들을 작동시킬 수 있는 전자장비의 집합
    * `SCSI`: `Serial Advanced Technology Attachment` (`SATA`)
* ![A typical PC bus structure](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/images/Chapter13/13_01_TypicalBus.jpg)

* Processor가 controller에게 I/O 전송을 완료하기 위해 명령과 데이터를 보내는 법
    * 각 controller는 하나 이상의 register 보유하고, Processor는 이것들을 읽고 쓰면서 통신
        * Special I/O instruction을 만들어 직접 접근을 구현
        * Memory-mapped I/O를 이용해 register에 기록
        * 일부는 위의 두 방식 모두 채택
* I/O Port의 register 구성
    * data-in register: host에 의해 읽힘
    * data-out register: host에 의해 쓰임
    * status register: host에 의해 읽힘, 현재 명령어의 수행 여부 등 기록
    * control register: host에 의해 쓰임, device mode 변경 혹은 command 시작 기록

### 1. Polling
* Busy waiting이라 불림
* 동작 과정
    1. host는 반복적으로 `status register`의 `busy bit`가 clear될 때까지 확인
    1. host가 `command register`의 `write bit`를 set한 뒤 data-out register에 byte 기록
    1. host가 `command-ready bit`을 set함 
    1. controller가 `command-read bit`이 set 된 것을 알면, controller가 `busy bit`을 set
    1. controller가 `command register`를 읽어 `write command`를 확인, 이후 `data-out register`를 읽어 I/O 시작
    1. controller가 `command-ready bit`, `status register`의 `error bit`, `busy bit` 3개를 clear 하여 I/O 종료를 알림
* 효율적이나 ready 상태인 device 찾기가 어려우면 비효율적이 됨

### 2. Interrupts
* CPU는 CPU가 매 instruction이 실행 후 감지하는 interrupt-request line이 존재
* interrupt-request line을 통한 signal 감지 시, interrupt handler routine으로 이동
* CPU가 asynchronous event에 반응할 수 있게 해줌
* Interrupt-controller가 제공하는 Interrupt handling 특징
    * 중요한 연산중에는 interrupt handling을 미뤄야 함
    * interrupt가 일어난 device를 보기 위해 모든 장치를 polling하지 않고 interrupt hanlder로 보내야 함
    * 우선순위를 매겨 적절한 대응을 하기 위해 multilevel interrupts 필요
* Interrupt-request line 종류
    * nonmaskable interrupt: 회복 불가능한 memory error같은 event를 위해 지정
    * maskable interrupt: critical instruction 실행 전까지 꺼놓을 수 있음
* `interrupt vector`
    * interrupt-handling이 저장된 주소를 offset에 저장하는 table
    * `interrupt chaining`: interrupt vector의 요소들이 각각 interrupt handler들의 list의 head를 가리킴, 하나씩 돌아보고 맞는 것을 찾음 (interrupt 종류마다 device에 따라 여러 handler 존재)
* `interrupt priority levels`: 높은 우선순위의 interrupt가 낮은 것을 무시하고 먼저 진행 가능
* `exceptions` 처리에도 쓰임
* `software interrupt` (`trap`)
    * 요구되는 kernel service를 파악하는 operand 보유
    * system library call 시 호출
    * trap instruction 실행 시, interrrupt hardware가 상태 저장 후 kernel mode로 switch 해서 요구되는 service를 적용한 kernel routine으로 보냄
    * device interrupt들보다는 상대적으로 low priority

### 3. Direct Memory Access
* disk drive와 같이 많은 양을 전송하는 device에게는 `PIO`(`Programmed I/O`)라고 불리는 CPU를 이용하여 한 번에 한 byte씩만을 전송하는 것은 너무 비효율적
* PIO로 인한 CPU의 사용량을 줄이기 위해 `direct-memory-access` (`DMA`) 고안 
* 작동 방식
    1. host가 DMA command block을 memory에 기록
        * DMA command block: 전송할 source, destination의 pointer, 전송될 bytes 수 보관
    1. CPU는 DMA command block을 DMA controller에 쓰고 다른 작업을 하러 감
    1. DMA controller가 memory bus를 단독으로 운영해 data 전송 받음
    1. disk controller가 DMA controller에게 byte씩 data 전송
    1. DMA controller는 메모리 주소에 bytes를 기록한 뒤 주소를 늘리고, 전송될 bytes 수를 줄임
    1. 전송 완료 시, interrupt를 보내 끝났음을 알림

* 자세한 설명은 추후 정리

### 4. I/O Hardware Summary
* 종류
    * bus
    * controller
    * I/O port와 I/O port가 가진 register
    * host와 device controller 간의 handshaking
    * polling loop 혹은 interrupt 방식의 handshaking
    * 많은 양의 전송을 위한 DMA controller를 통한 offloading

## Application I/O Interface
* Device hardware의 구현 차이  
    * Character-stream vs block: bytes씩 보내기 vs block 단위로 보내기
    * Sequential or random access: 정해진 순서대로 보내기 vs 잡히는 대로 보내기
    * Synchronous vs asynchronous: system에 맞춰 예상되는 response time에 전송 vs computer event에 맞춰지지 않는 예측 불가한 response time
    * Sharable or dedicated: 여러 process/thread에 의해 동시에 접근 가능 vs 한 번에 하나씩
    * Speed of operation: device 속도
    * Read-write vs read only vs write only
* device의 차이가 있어도 OS가 보는 사용법은 비슷
    * block I/O, character-stream I/O
    * Memory-mapped file access
    * Network sockets
* 대부분 OS는 driver를 통과해 직접 명령할 수 있는 escape(back door)를 가짐
    * ex) ioctl() (UNIX)

### 1. Block and Character Devices
* block-device interface
    * disk drive 혹은 block 지향 device 접근에 필요한 것들을 가짐
    * read(), write(), seek()
* raw I/O
    * block device를 linear array of blocks로 보고 접근하는 것
    * file system이 차지하는 용량의 overhead 불필요
    * OS가 거치지 못하게 함 => direct I/O
* Memory mapped file access
    * block-device drivers의 최상단 layer에 위치
    * read(), write() 제공 대신 main memory의 bytes array를 통해 disk storage 접근
    * demand-page와 같이 data 전달이 이루어짐
* character stream interface: keyboard 같은 것 존재

### 2. Network Devices
* network i/o는 disk i/o와 확연이 차이나기에 read(), write(), seek() 대신 socket interface 사용
    * select(): socket들을 관리, 각 socket의 상태 정보 return

### 3. Clocks and Timers
* computer의 clock/timer의 역할
    * 현재 시간 반환
    * 걸린 시간 반환
    * 특정 시간에 특정 작업을 trigger 하는 timer 설정
* programmable interval timer
    * 걸린 시간 반환 및 trigger timer
    * 특정 시간이 되면 interrupt 생성
    * scheduler에 사용

### 4. Nonblocking and Asynchronous I/O
* blocking system call
    * application이 정지
    * wait queue로 전환되고, system call 끝날 경우 run queue로 돌아옴
    * hardware들은 보통 asynchronous하게 수행하지만, OS는 이해의 용이성 때문에 blocking system call 사용
* nonblocking I/O
    * 일부 user-level process에서 요구: keyboard 입력 등을 받는 경우, video frame을 읽으며 보여주는 경우
    * 무언가 값을 받기 시작했을 때, return
* asynchronous I/O
    * nonblocking I/O의 대체
    * I/O 완료를 기다리지 않고 call 즉시 return

### 5. Vectored I/O
* 하나의 system call로 동시에 여러곳에서 여러 I/O가 가능하게 함
* context-switching 및 system call overhead 피함
* scatter-gather 방식: interruption 없이 atomicity 보장
* vectored I/O를 쓰지 않으면 큰 buffer에 순서대로 담아서 전송하는 비효율적 방식 사용

## Kernel I/O Subsystem
### 1. I/O Scheduling
### 2. Buffering
### 3. Caching
### 4. Spooling and Device Reservation
### 5. Error Handling
### 6. I/O Protection
### 7. Kernel Data Structures

## Transforming I/O Requests to Hardware Operations

## STREAMS

## Performance

## Summary