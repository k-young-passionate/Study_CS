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
* Process는 memory에 있어야 하나 잠깐 swapped되어 backing store로 나갔다가 돌아와서 실행될 수 있음
* 더 많은 주소를 이용할 수 있음 
1. Standard Swapping
    * main memory와 backing store (보통 fast disk) 사이에 process가 이동하는 것을 포함
    * backing store, memory에 있는 실행 준비된 process들로 ready queue 구성
    * context switch가 오래 걸리게 됨
    * idle 상황에서만 swap out이 가능
    * I/O operation 중에는 memory에 data를 불러와야하는 문제
        * never swap a process with pending I/O
        * OS buffer에 I/O 하기 => 추후 swapped in 되었을 때, process의 buffer에 옮겨야 하므로 double buffering이고 overhead가 크게 됨
    * 다른 방식
        * 일정 threshold 이상의 memory 점유 발생 시에서야 swapping을 시도하는 방식
        * process의 일부만 swapping 하는 방식

1. Swapping on Mobile Systems
    * Swapping을 지원하지 않음
        * Hard disk대신 좀 더 좁은 flash memory를 쓰기 때문에, 공간 부족으로 swapping을 하기 힘들다.
        * flash memory가 견딜 수 있는 쓰기 횟수 제한 => 이를 넘어가면 unreliable + poor throughput
    * Swapping 대신 쓰는 기법
        * iOS: Read-only data는 memory에서 없앴다가 필요할 때 다시 불러옴, 메모리 공간 부족 시, OS에 의해 process halted
        * Android: iOS와 비슷한 전략, Android는 process 강제 halt 전 application state를 flash memory에 기억

## Contiguous Memory Allocation
* 메모리 공간은 OS와 user process 공간으로 나누어 짐
    * Interrupt vector에 의해 OS memory 공간을 lowest/highest로 할 것인지 결정, 보통 lowest에 있어서 OS도 lowest로 배치
* Contiguous Memory Allocation: 각 process가 memory의 single section만을 차지, 즉 파편화되어있지 않음 
1. Memory Protection
    * process가 시작되면 dispatcher가 limit 및 relocation register를 불러옴
    * limit register의 주소 값 이상을 참조하는지 체크하고, 유효한 범위면 relocation register 값을 더해 physical address를 참조
    * transient OS code: 필요하면 부르고, 쓰이지 않으면 내보내는 OS 관련 code들 (device driver 등)
1. Memory Allocation
    * Memory 배치 방법
        1. Fixed-sized partition
            * 각 process는 하나의 process 가짐
            * multiple-partion method 라고도 함
            * degree of multiprogramming이 partition 갯수에 의해 결정
        1. Variable partition
            * 이용 가능한 memory와 점유된 memory에 대한 정보를 table로 관리
            * 다 비어있을 때는, 하나의 큰 block으로의 이용가능한 memory, 즉 hole로 여겨짐
            * 다양한 사이즈의 hole로 memory가 구성
    * Dynamic storage-allocation problem: 여러 hole 중 어디에 배치될 것인가
        1. first-fit: 가장 먼저 찾은 가능한 hole에 배치
        1. best-fit: 가장 비슷한 size의 hole에 배치, 조그마한 hole을 남김
        1. worst-fit: 가장 큰 size의 hole에 배치, 남은 hole의 이용가능성이 증가함
        * Simulation을 통해 first-fit과 best-fit이 worst-fit보다 성능이 좋은 것을 보여줌
        * first fit이 보통 빠름
1. Fragmentation
    * 종류
        1. external framgentation
            * 배치된 process에 의해 contiguous 하지 않은 조그만 hole 발생
            * first-fit과 best-fit은 external fragmentation으로 고통받음
            * 빈 공간은 충분하지만 process를 배치할 수 없는 상황 발생 가능
            * 50% rule: 통계적으로 N process를 배치하면, external fragmentation으로 인한 0.5N 개의 배치할 수 없는 memory 공간 발생
        1. internal fragmentation
            * 할당받은 memory 보다 적게 사용해서 memory가 낭비되는 현상
            * multiple-partition allocation 방식에서 많이 발생
            * internal fragmentation으로 인해 발생한 hole을 추적하는 것의 overhead는 그 자체보다 크다
    * 해결책
        1. external fragmentation
            * compaction: memory를 재배치해서 free memory를 한 곳으로 모으는 것, address가 static 하면 불가하고 dynamic하면 가능하지만 overhead가 클 가능성 높음
            * process를 contigous하지 않게 배치하는 것: segmentatoin과 paging으로 달성 가능
        1. internal fragmentation
            * physical memory를 fixed-sized block으로 나눠 memory를 block size에 기반한 unit 단위로 배치하는 것
## Segmentation
1. Basic Method
    * programmer가 보는 memory: 다양한 size의 segment들의 집합
    * segmentation: programmer가 보는 memory를 지원하는 memory 관리 방식
    * 각 segment는 이름과 길이를 가지고, address는 segment name과 offset을 지정
        * programmer는 각 address를 segment name과 offset로 지정
        * 구현 용이화를 위해 segment name 대신 segment number로 표현
        * ```<segment-number, offset>``` 으로 logical address 구성
    * C compiler가 만드는 segment들
        * code
        * global variable
        * heap
        * stack
        * standard C library
1. Segmentation Hardware
    ![segmentation hardware](https://img1.daumcdn.net/thumb/R720x0.q80/?scode=mtistory2&fname=http%3A%2F%2Fcfile2.uf.tistory.com%2Fimage%2F211313505757D40A177146)
    * segment table: segment base와 limit으로 구성
        * segment base: 시작하는 physical address
        * segment limit: segment 길이
    * logical address
        * segment number ```s```: segment table의 index
        * offset ```d```: 0에서 segment limit 사이의 숫자 

## Paging
1. Basic Method
    * frame이라 불리는 physical memory를 나눈 fixed-sized block과 page라 불리는 logical memory를 같은 size로 나눈 것을 이용
    * process 실행 시, page가 frame 위에 load 됨
    * CPU에 의해 생성되는 address는 page number ```p```와 page offset ```d```로 구분
    ![paging hardware](https://lh3.googleusercontent.com/proxy/snx0LJiUfZj4byxGtwBCceRUH979O3cUKcJIvCqscUv9v6IGCMmird8-fVEm542KZEQIkaIYpeTa14e7vwAf74aqRSVtiLi9t9fjrzWKVKH-fkcXsv7yRHH6kFlbhf7Zc5fyi5KhCtiLHFoCgTeiLAqbnK7B2ZmB-OqBawMDcfwesg)
        * page number: page table의 index
        * page table: 각 page의 base address 보관
    * Process size와 page size의 차이로 internal fragmentation이 발생
        * small page size 사용 시, internal fragmenation을 줄일 수 있으나, page가 많아짐으로써 page table이 커지는 overhead 존재
        * I/O 시에 더 많은 데이터를 가져오는 것이 유리
        * 최근 memory, process, data set들이 커짐에 따라 page도 커지고 있음 (4/8KB 수준 혹은 그 이상)
    * Frame에 대한 정보는 OS가 Frame table로 관리
        * Free / Allocated 여부
        * 어떤 process의 몇 번째 page인지에 대한 정보
    
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
