# Memory-Management Strategy
## Background
* Memory
    * modern computer system의 작동의 중심
    * 주소를 가진 a large array of bytes로 구성
    * CPU는 PC에 담긴 주소를 참조해 instruction을 memory에서 가져옴
1. Basic Hardware 
    * Main memory와 processor에 설치된 register는 CPU가 직접 접근할 수 있는 범용도 storage
    * Register는 CPU one Cycle 안에 접근 가능
    * Main memory는 memory bus를 타고 오기에 여러 CPU cycle이 소요된다.
    * stall: main memory에서 data를 기다리는 동안 CPU clock이 낭비되는 상태
    * cache를 통해 stall 해결 노력
    * base register에는 유효한 memory 시작 주소, limit register에는 범위를 저장
    * 유효한 memory 주소를 넘어가면 trap 발생

1. Address Binding
    * Program은 binary executable file이고, 실행되기 위해서는 process로 memory에 올라가야 함
    * memory에 올라가기를 기다리는 disk 속 process들은 input queue의 형태를 이룬다.
    * User process는 memory 아무 곳에서 시작해도 됨 (0x00000에서 시작할 필요 X)
    * Source program의 address들은 symbolic 하고 (변수로 표현) compiler는 이를 재위치 가능한 주소로 bind 해준다. linkage editor 또는 loader는 relocatable address를 절대 주소로 bind 해준다.
    * instruction 및 data가 memory 주소에 binding이 될 수 있는 step
        * Compile Time: absolute code 생성
        * Load Time: process가 compile time에 memory 어디에 자리잡을지 결정되지 않았다면, compiler는 relocatable code를 생성해야한다. 이 경우 최종 binding은 load time까지 연기된다. 시작 주소가 바뀌면 user code를 다시 올려야 한다.
        * Execution Time: 실행 중인 process의 memory segment 가 바뀔 수 있다면, binding은 runtime으로 미뤄져야 한다.
        ![Multistep processing of a user program](https://media.geeksforgeeks.org/wp-content/uploads/20200531135539/1406-5.png)

1. Logical Versus Physical Address Space
    * CPU에 의해 생성된 주소는 logical address라고 부르고, memory-address register에 올라오는 memory unit에 보이는 주소는 physical address라 부른다.
    * compile time과 load time의 address binding method는 똑같은 logical/physical 주소를 생성하나, execution time에는 서로 다른 logical/physical 주소를 생성한다. execution time에 생성되는 logical address를 virtual address라 한다.
    * logical address space: 프로그램에 의해 생성되는 모든 logical/virtual address set
    * physical address space: logical address space에 해당하는 physical address set
    * MMU(memory-management unit): virtual to physical address로 run-time에 mapping 해주는 역할 수행하는 hardware
    * base register == relocation register
    * user program은 physical address를 볼 일 없음

1. Dynamic Loading
    * 모든 process를 memory에 올려야하지만, 더 나은 memory-space utilization을 위해 필요할 때마다 불러오는 dynamic loading 사용
    * 필요할 때만 올라와서 자주 발생하지 않는 code를 다룰 때 좋다.
    * OS의 특별한 도움을 요구하지 않는다.

1. Dynamic Linking and Shared Libraries
    * Dynamically linked libraries: run-time에 프로그램과 link 되는 system library
    * static linking: system library가 object module 취급되어 loader에 의해 binary program image에 결합되는 linking, 안쓰이는 것들도 main memory에 올라가 공간 낭비
    * dynamic linking: execution time까지 linking을 미룸, 필요한 것만 불러옴
    * stub: 필요한 library의 routine이 memory에 없을 때 어떻게 load해야하는지 설정하는 code
    * shared library: 여러 버전이 사용 됨
    * OS의 도움을 필요로 함, 다른 process의 메모리 주소에 접근하여 필요한 routine이 있는지 확인하여 여러 process가 접근할 수 있게 해줘야 함 

## Swapping
1. Standard Swapping
1. Swapping on Mobile Systems

## Contiguous Memory Allocation
1. Memory Protection
1. Memory Allocation
1. Fragmentation

## Segmentation
1. Basic Method
1. Segmentation Hardware

## Paging
1. Basic Method
1. Hardware Support
1. Protection
1. Shared Pages

## Structure of the Page Table
1. Hierarchical Paging
1. Hashed Page Tables
1. Inverted Page Tables

## Example: Intel 32 and 64-bit Architectures
1. IA-32 Architecture
1. x86-64

## Example: ARM Architecture

## Summary