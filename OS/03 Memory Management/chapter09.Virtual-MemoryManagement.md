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
1. Performance of Demand Paging

## Copy-on-Write

## Page Replacement
1. Basic Page Replacement

1. FIFO Page Replacement

1. Optimal Page Replacement

1. LRU Page Replacement

1. LRU-Approximation Page Replacement

1. Counting-Based Page Replacement

1. Page-Buffering Algorithms

1. Applications and Page Replacement

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