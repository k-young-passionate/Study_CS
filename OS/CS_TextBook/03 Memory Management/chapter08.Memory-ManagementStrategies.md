# Memory-Management Strategy
## Background
* Memory
    * modern computer system의 작동의 중심
    * 주소를 가진 a large array of bytes로 구성
    * CPU는 PC에 담긴 주소를 참조해 instruction을 memory에서 가져옴
### 1. Basic Hardware 
* Main memory와 processor에 설치된 register는 CPU가 직접 접근할 수 있는 범용도 storage
* Register는 CPU one Cycle 안에 접근 가능
* Main memory는 memory bus를 타고 오기에 여러 CPU cycle이 소요된다.
* stall: main memory에서 data를 기다리는 동안 CPU clock이 낭비되는 상태
* cache를 통해 stall 해결 노력
* base register에는 유효한 memory 시작 주소, limit register에는 범위를 저장
* 유효한 memory 주소를 넘어가면 trap 발생

### 2. Address Binding
* Program은 binary executable file이고, 실행되기 위해서는 process로 memory에 올라가야 함
* memory에 올라가기를 기다리는 disk 속 process들은 input queue의 형태를 이룬다.
* User process는 memory 아무 곳에서 시작해도 됨 (0x00000에서 시작할 필요 X)
* Source program의 address들은 symbolic 하고 (변수로 표현) compiler는 이를 재위치 가능한 주소로 bind 해준다. linkage editor 또는 loader는 relocatable address를 절대 주소로 bind 해준다.
* instruction 및 data가 memory 주소에 binding이 될 수 있는 step
    * Compile Time: absolute code 생성
    * Load Time: process가 compile time에 memory 어디에 자리잡을지 결정되지 않았다면, compiler는 relocatable code를 생성해야한다. 이 경우 최종 binding은 load time까지 연기된다. 시작 주소가 바뀌면 user code를 다시 올려야 한다.
    * Execution Time: 실행 중인 process의 memory segment 가 바뀔 수 있다면, binding은 runtime으로 미뤄져야 한다.
    ![Multistep processing of a user program](https://media.geeksforgeeks.org/wp-content/uploads/20200531135539/1406-5.png)

### 3. Logical Versus Physical Address Space
* CPU에 의해 생성된 주소는 logical address라고 부르고, memory-address register에 올라오는 memory unit에 보이는 주소는 physical address라 부른다.
* compile time과 load time의 address binding method는 똑같은 logical/physical 주소를 생성하나, execution time에는 서로 다른 logical/physical 주소를 생성한다. execution time에 생성되는 logical address를 virtual address라 한다.
* logical address space: 프로그램에 의해 생성되는 모든 logical/virtual address set
* physical address space: logical address space에 해당하는 physical address set
* MMU(memory-management unit): virtual to physical address로 run-time에 mapping 해주는 역할 수행하는 hardware
* base register == relocation register
* user program은 physical address를 볼 일 없음

### 4. Dynamic Loading
* 모든 process를 memory에 올려야하지만, 더 나은 memory-space utilization을 위해 필요할 때마다 불러오는 dynamic loading 사용
* 필요할 때만 올라와서 자주 발생하지 않는 code를 다룰 때 좋다.
* OS의 특별한 도움을 요구하지 않는다.

### 5. Dynamic Linking and Shared Libraries
* Dynamically linked libraries: run-time에 프로그램과 link 되는 system library
* static linking: system library가 object module 취급되어 loader에 의해 binary program image에 결합되는 linking, 안쓰이는 것들도 main memory에 올라가 공간 낭비
* dynamic linking: execution time까지 linking을 미룸, 필요한 것만 불러옴
* stub: 필요한 library의 routine이 memory에 없을 때 어떻게 load해야하는지 설정하는 code
* shared library: 여러 버전이 사용 됨
* OS의 도움을 필요로 함, 다른 process의 메모리 주소에 접근하여 필요한 routine이 있는지 확인하여 여러 process가 접근할 수 있게 해줘야 함 


## Swapping
* Process는 memory에 있어야 하나 잠깐 swapped되어 backing store로 나갔다가 돌아와서 실행될 수 있음
* 더 많은 주소를 이용할 수 있음 
### 1. Standard Swapping
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

### 2. Swapping on Mobile Systems
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
### 1. Memory Protection
* process가 시작되면 dispatcher가 limit 및 relocation register를 불러옴
* limit register의 주소 값 이상을 참조하는지 체크하고, 유효한 범위면 relocation register 값을 더해 physical address를 참조
* transient OS code: 필요하면 부르고, 쓰이지 않으면 내보내는 OS 관련 code들 (device driver 등)

### 2. Memory Allocation
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

### 3. Fragmentation
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
### 1. Basic Method
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

### 2. Segmentation Hardware
![segmentation hardware](https://img1.daumcdn.net/thumb/R720x0.q80/?scode=mtistory2&fname=http%3A%2F%2Fcfile2.uf.tistory.com%2Fimage%2F211313505757D40A177146)
* segment table: segment base와 limit으로 구성
    * segment base: 시작하는 physical address
    * segment limit: segment 길이
* logical address
    * segment number ```s```: segment table의 index
    * offset ```d```: 0에서 segment limit 사이의 숫자 

## Paging
### 1. Basic Method
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
    
### 2. Hardware Support
* OS 별 page table 저장법
    * 각 process마다 Page table을 할당하고, PCB에 page table 위치 저장
    * 적은 수의 page table을 제공하여 context switch overhead를 줄임
* Hardware Implementation
    * Simple: 지정된 register set으로 page table 적용, 빠른 접근이 가능하나 page table이 작을 경우만 가능
    * Simple method's Solution: Main memory에 저장하고 Page-Table Base Register(PTBR)가 가리키게 함, page table 교체 시 주소만 update하면 되기에 context switch overhead가 줄어드나 main memory에 접근하는 시간이 오래 걸림
    * Current Standard: Translation Look-aside Buffer(TLB)라는 빠르고 작은 hardware cache에 key-value 쌍으로 저장 
* TLB
    
    ![TLB](https://upload.wikimedia.org/wikipedia/commons/thumb/6/6e/Translation_Lookaside_Buffer.png/373px-Translation_Lookaside_Buffer.png)
    * page-table의 일부만 가짐
    * logical address가 CPU에 의해 계산되면, page number를 TLB에서 찾아 frame number를 얻고, TLB miss가 날 경우 memory에서 가져와서 offset과 physical address 연산
    * 일부 entry는 wired down함 (교체되지 않도록 지정)
    * 일부 TLB는 각 entry마다 address-sapce identifiers(ASIDs)를 보관
    * ASIDs: 각 process를 구분하여 address-space protection, ASIDs match 실패 시, TLB miss로 처리

* 성능
    * hit ratio: TLB에서 발견되는 비율
    * effective memory-access time = TLB hit ratio * TLB access time + TLB miss ratio * Memory access time

### 3. Protection
* 보통 bit로 관리하고 page table에 포함
* 권한 관련 bit
    * 각 frame의 protection bits에 의해 protected
    * read-only / read-write 결정
    * bit를 추가해 execute-only 등도 설정 가능
    * 위반 시 hardware trap 혹은 memory-protection violation 발생
* valid-invalid bit
    * page 접근 권한 통제
    * process가 접근할 수 있는 logical address인지를 확인
* PTLR (page-table length register)
    * 일부 system에서 제공
    * page table의 size를 알려줌
    * process가 접근하려는 주소가 유효한지 확인

### 4. Shared Pages
* paging의 장점으로 common code를 공유할 수 있는 가능성이 있음
* reenterant code (pure code) 일 경우 여러명이 사용 가능
    * reenterant code: non self modifying code, 실행하는 동안 바뀌지 않음

## Structure of the Page Table
### 1. Hierarchical Paging
* 현대 system은 매우 큰 logical address 사용 => page table도 커짐
* page table을 효율적으로 관리하기 위해 two-level paging 사용
    * page table 자체도 paging 하기
    * outer page table에서 inward로 들어오는 방식으로 forward-mapped page table이라고 하기도 함
    ![two-level page table](https://i.stack.imgur.com/ML9OY.png)
* 이전에 VAX minicomputer에서 section과 page, offset을 나눈 logical address 구조를 사용
### 2. Hashed Page Tables
* 32 bit보다 큰 주소를 다루기 위해 사용
* 각 entry는 hash collision을 대비하기 위해 linked list로 구성
* 각 element는 virtual page number, 대응되는 page frame의 값, 다음 element를 기라키는 pointer
* 동작 방식
    1. virtual page number가 hash table에 hash되어 들어감
    1. linked list를 탐색하며 match가 있으면 page frame 사용해 physical address로 변환, miss일 경우 linked list의 남은 entry 탐색
    ![hashed page table](https://i.stack.imgur.com/Fbv32.png)
* 변형으로 clustered page table이 있음
    * 하나의 entry가 여러 page 가리킴
    * noncontiguous하거나 scattered된 memory를 참조하는 sparse address 공간에 유용

### 3. Inverted Page Tables
* 일반적인 page table
    * 하나의 entry에 page를 대응시키고, process는 page table에서 virtual address를 통해 실제 page를 찾아감
    * 너무나 많은 entry로 인해 공간을 많이 차지
* Inverted Page Table
    * 하나의 entry에 하나의 real page 주소 참조
    * 각 entry는 process 정보와 실제 주소에 저장된 virtual address로 구성
    * 하나의 page table만 system에 존재하고, 각 physical memory의 page마다 하나의 entry만을 갖게 됨
    * Entry는 <process-id, pagge-number, offset> 으로 구성
    * page table에서 match를 찾기위해 탐색
    * i 번째에서 match를 찾으면 physical address는 <i, offset>인 것임
    ![Inverted page table](https://img1.daumcdn.net/thumb/R720x0.q80/?scode=mtistory2&fname=http%3A%2F%2Fcfile28.uf.tistory.com%2Fimage%2F9933253359C4A8301D6602)
* 특장
    * memory를 획기적으로 줄여줌
    * search time 증가

### 4. Oracle SPARC Solaris
* 두 개의 hash table 운영: 하나는 kernel, 하나는 user용
* 각각의 virtual memory는 physical memory에 map 됨
* 각각의 entry는 contiguous 함
* hash table을 통해 각 address가 탐색되면 느려질 수 있기 때문에 Translation table entry(TTE)를 가진 TLB를 적용

## Example: Intel 32 and 64-bit Architectures
### 1. IA-32 Architecture
### 2. x86-64

## Example: ARM Architecture

## Summary
