# Virtual-Memory Management

## Background
* 실제로 프로그램 전체가 메모리에 올라와 있을 필요가 없음
    * 잘 발생하지 않는 error code는 실행될 일이 거의 없다
    * Array, Lists, Tables는 실제 필요한 것보다 많이 할당
    * 특정 옵션이나 특징은 거의 쓰이지 않음
    * 전체의 코드가 쓰여도 동시에 필요할 일은 없음
* program 일부만 memory에 올라와있는 경우 장점
    * Physical memory 용량 제한을 받지 않음
    * 동시에 memory에 올릴 수 있는 process가 많아지면서, CPU util 증가
    * I/O의 필요가 줄어들어 더 빠른 이용 가능
* Virtual Memory
    * physical memory와 분리되어 user에게 인지되는 logical memory
    * 실제 physical memory보다 훨씬 커짐
* Virtual Address Space
    * process가 memory에 어떻게 저장되어 있는지에 대한 logical view
    * address 0 부터 시작

    ![virtual address space](https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/Virtual_address_space_and_physical_address_space_relationship.svg/1200px-Virtual_address_space_and_physical_address_space_relationship.svg.png)

    * 가운데 비어있는 부분은 stack 혹은 heap에 의해 채워지게 되었을 때 physical page가 필요하게 됨
    * 가운데가 비어있는 virtual address space를 sparse address space라고 함
        * stack, heap으로 채울 수 있거나 dynamic하게 library를 link할 수 있음
* logical memory에서는 file과 memory를 두개 이상의 process에게 page sharing을 통해 나누어 줌
    * system library가 share 가능
    * process끼리 memory를 share할 수 있음
    * fork call 동안 page를 share하여 process creation을 좀 더 빠르게 할 수 있음

## Demand Paging
* 필요할 경우에만 page를 main memory에 load 하는 것
* 사용이 안 될 경우 load 되지 않음
* paging system에 swapping을 적용한 것과 비슷
* lazy swapper를 이용해 page가 필요할 때만 swap
    * swapper는 모든 process를 관장하기 때문에, pager라는 이름이 더 적합
1. Basic Concepts
    * memory/disk에 있는 page 구분하기 위한 hardware support 필요: valid-invalid bit 방법 사용
        * valid: page가 legal하고 memory에 있을 때
        * invalid: page가 valid 하지 않거나, valid한데 disk에 없을 때
    * memory resident: 메모리에 올라와 있는 상태
    * Page fault: invalid marked된 page에 접근하면 발생
        * 처리
            1. internal table 확인해 valid 여부 확인
            1. invalid일 경우 process 종료, valid이나 올라오지 않은 page의 경우 memory에 page를 load
            1. 빈 frame 찾음
            1. 새롭게 allocated 된 frame에 필요한 page를 읽는 disk operation 실행
            1. disk를 다 읽으면 memory에 새로 올라온 page를 가리키기 위해 page table update
            1. interrupted된 instruction을 재시작
    * pure demand paging: 필요해지기 전까지는 절대 page를 memory로 가져오지 않음
    * locality of reference로 인해 계속된 page fault가 발생하지 않음, 참조했던 page 또 참조하는 확률 높음
    * demand paging을 지원하는 hardware
        * Page table: 각 page마다 valid 여부 확인
        * Secondary memory: main memory 위에 없는 page를 swap space에 보관, swap device라고도 불림
    * PCB에 state가 저장되기 때문에 page fault 이후 실행 종료된 instruction부터 실행 가능
    * 힘든 상황: instruction이 여러 다른 위치를 수정하는 경우 => 수많은 page fault
        * 해결책
            1. 접근하려는 block의 처음부터 끝까지를 page fault 한 번에 모두 메모리에 load 해 연산
            1. overwritten 되기 전의 memory 값을 temporary register에 저장한 뒤, page fault가 발생하면 trap 작동 전 원래 값으로 돌려놓는 방식

1. Performance of Demand Paging
    * Demand paging's effective access time = (1 - `p`) * `memory access time` + `p` * `page fault time`
        * `p`: page fault rate 
        * page fault rate에 의해 effective access time 결정
    * page fault 발생 시 처리해야하는 것
        1. page fault interrupt
        1. page 읽기
        1. process 재시작


## Copy-on-Write
    * fork() 시에 parent와 child가 초반에 같은 page를 공유하는 것
    * copy on write로 marked 됨
    * 한 process의 page 변경 시도 시, free page의 pool에서 copy된 page를 생성하고 여기서 수정 작업
    * zero-fill-on-demand: free page 할당 전에 0으로 채움
## Page Replacement
1. Basic Page Replacement
    * 방법
        1. 원하는 page를 disk에서 찾음
        1. 빈 frame 찾음
            1. free frame이 있다면 사용
            1. free frame이 없다면 victim frame을 page replacement algorithm을 통해 선정 후 disk에 기록하고 page 및 frame table 교체
        1. 원하는 page를 free frame에 기록하고 page 및 frame talbe 교체
        1. process 다시 시작
    * disk에 기록하는 과정으로 인한 page replacement cost의 두 배 증가
        * dirty bit(modify bit)를 이용해 변경되지 않은 page는 덮어씌움
    * demand paging 적용 시, 해결해야하는 두 가지 문제
        1. frame-allocation algorithm: 각 process에게 어느정도의 frame을 배치할 것인지
        1. page-replacement algorithm
    * reference string을 통해 algorithm 평가
        * 1, 4, 1, 6, 1, 6, 1, 6, ... 

1. FIFO Page Replacement
    * 먼저 읽은 page가 먼저 교체됨
    * queue로 만들어 관리
    * Belady's anomaly: page-fault가 배치된 frame 수 만큼 증가하는 현상
    * Belady's anomaly 발생 가능

1. Optimal Page Replacement
    * Belady's anomaly를 해결하기 위한 알고리즘
    * 가장 오래 사용되어지지 않은 page 교체
    * reference string이 필요해 적용이 쉽지 않음

1. LRU Page Replacement
    * Optimal page replacement에 근접한 알고리즘
    * 최근에 사용된 것을 근접한 미래에도 사용한다는 사실을 이용
    * 적용 방법
        1. Counter를 이용한 방법: reference 마다 counter 증가, 가장 counter가 적은 page 교체, Overflow 고려 필요
        1. Stack을 이용한 방법: page number들에 대한 stack 유지, 참고된 page는 stack에서 제거된 후 push, 가장 안쪽에 있는 page를 교체

1. LRU-Approximation Page Replacement
    * 적은 수의 system만 LRU 지원 가능한 hardware 소유
    1. Additional-Reference-Bits Algorithm
        * 각 page table entry에 8 bit reference 기록
        * timer interrupt마다 right shift 후 msb에 숫자를 기록
        * 참조되었으면 1, 참조되어지지 않았으면 0
        * 가장 작은 reference를 가진 것이 교체
        * 겹치는 reference 발생 시, 모두 내보내거나 FIFO 적용
    1. Second Chance Algorithm
        * FIFO에 기반
        * 1 bit 짜리 reference 사용
        * 시계처럼 돌면서 reference bit가 0이면 교체, 1이면 0으로 줄이고 다음으로 넘어감
        * 사용된 page의 reference bit는 1로 set
        * clock algorithm이라고도 함
    1. Enhanced Second-Chance Algorithm
        * reference bit(1 bit)와 modify bit(1 bit) 이용
        * (r, m) - (0, 0), (0, 1), (1, 0), (1, 1) 순으로 교체

1. Counting-Based Page Replacement
    * Least Frequently Used(LFU): 가장 작은 count의 page가 replace
    * Most Frequently Used(MFU): 가장 count가 적을수록 최근에 들어와서 사용되어지지 않았다고 가정

1. Page-Buffering Algorithms
    * modify가 되어지지 않는 free frame을 두고, 읽기만 할 page는 해당 free frame에 swap-out 없이 교체

1. Applications and Page Replacement
    * Database 같이 스스로 I/O를 관리하는 application은 OS가 관리하는 것보다 나은 성능을 보임
    * Application에서는 오래된 데이터를 읽을 확률이 더 높고, 따라서 LRU보다는 MFU가 더 적합
    * 일부 OS는 file system이 없는 raw disk 제공: 매우 긴 sequential array

## Allocation of Frames
1. Minimum Number of Frames

1. Allocation Algorithms

1. Global versus Local Allocation

1. Non-Uniform Memory Access

## Thrashing
1. Cause of Thrashing

1. Working-Set Model

1. Page-Fault Frequency

1. Concluding Remarks

## Memory-Mapped Files
1. Basic Mechanism

1. Shared Memory in the Windows API

1. Memory-Mapped I/O

## Allocating Kernel Memory
1. Buddy System

1. Slab Allocation

## Other Considerations
1. Prepaging

1. Page Size

1. TLB Reach

1. Inverted Page Tables

1. Program Structure

1. I/O Interlock and Page Locking

## Operating-System Examples
1. Windows

1. Solaris

## Summary