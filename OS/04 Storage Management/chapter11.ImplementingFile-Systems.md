# Implementing File-Systems

## File-System Structure
* Disk가 secondary storage로 제공될 수 있게 해주는 두 가지 특징
    1. disk는 같은 곳에 읽고, 변경하여, 다시 쓰는 것 가능
    1. 어떤 block이든지 직접 접근 가능
* I/O efficiency를 향상시키기 위해 block 단위로 memory와 disk 사이 I/O 발생 
* Sector
    * 각 block은 1개 이상의 sector로 이루어짐
    * 1 sector는 32 bytes에서 4,096 bytes 사이
    * 보통은 512 bytes
* File systems
    * data를 저장/위치/회수를 쉽게 함으로써 disk 접근을 편리하게 함
    * 설계 문제
        1. file system을 user에게 어떻게 보여줄 것인가: file, 속성, 동작, directory structure 등에 대한 정의
        1. physical device에 logical file system을 어떤 algorithm과 자료구조로 올릴 것인가
* Layered File System
    * ![layered file system](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/images/Chapter12/12_01_LayeredFileSystem.jpg)

    1. devices
    1. I/O control
        * device drivers와 intterupt handler로 구성되어 main memory와 disk system 사이의 정보 전달 역할
        * device drivers: translator, high-level commands를 low-level 및 hardware-specific instructions로 만들어 줌
    1. basic file system
        * generic commands를 적합한 device driver에게 전달
        * file system, directory, data block과 관련된 memory buffer와 cache 관리
    1. file-organization module
        * file, logical blocks, physical blocks에 대해 파악
        * logical block address를 physical block address로 번역
        * free space manager를 가지고 unallocated block을 추적
    1. logical file system
        * metadata 정보를 관리
        * Metadata: 실제 데이터를 제외한 file-system 구조의 전부
        * 소유, 권한, 위치정보 등에 대한 file 정보를 담은 `File Control Block`(FCB, UNIX에서는 inode) 관리
    1. application programs

    * 장점: code의 중복 최소화
    * 단점: overhead 발생

* 다양한 file systems
    * UNIX: `Berkely Fast File System(FFS)`에 기반을 둔 `UNIX file systems(UFS)`
    * Windows: `FAT`, `FAT32`, `NTFS(Windows NT File Systems)`
    * Linux: `extended file system` (`ext3`, `ext4`) 

## File-System Implementation
### 1. Overview
* File system 구성
    * `boot control block` (per volume)
        * volume에 있는 OS를 boot하기 위한 정보
        * 보통 volume의 첫 번째 block에 위치
        * UFS에서는 `boot block`, NTFS에서는 `partition boot sector`로 불림
    * `volume control block` (per volume)
        * partition에 들어있는 block 갯수, block 크기, 빈 block 갯수, free-block pointer, 빈 FCB 갯수, FCB pointer 등 volume에 대한 정보
        * UFS에서는 `superblock`, NTFS에서는 `master file table`로 불림
    * directory structure (per file system)
        * file 구성에 쓰임
        * UFS에서는 file 이름과 연동된 inode number 포함
        * NTFS에서는 master file table에 저장
    * `FCB` (per file)
        * file에 대한 자세한 정보 보유
        * directory entry에 연계된 유일한 id number 보유
        * NTFS에서는 Relational Database 구조인 master file table에 row per file 형태로 보관

* In-Memory 정보
    * file-system 관리 및 성능 향상에 쓰임
    * in-memory `mount table`: 각 mounted 된 volume에 대한 정보
    * in-memory directory-structure caches: 최근에 접근한 directory 정보 보유 
    * `system-wide open-file table`: opened 된 file의 FCB 복사본 보유
    * `per-process open-file table`: system-wide open-file table의 적절한 entry에 대한 pointer 및 다른 정보들 보유
    * buffers: disk에서 읽히거나 쓰임당할 때, file-system block을 hold

* File 생성 과정
    1. application program이 logical file system 호출
    1. 새 FCB 할당
    1. 적합한 directory를 메모리에서 읽어 새 file 이름과 FCB로 update 후 disk에 씀

* Directory 처리
    * File로 처리: UNIX, type만 directory로
    * File과 따로 처리: Windows, system call 분리

* File 생성 후의 I/O
    1. open()을 통해 Open 되어야 함
        1. logical file system에 file name을 넘김
        1. open() system call은 system-wide open-file table에서 이미 다른 process에 의해 file이 열려있는지 확인
            * file이 열려있으면 해당 table을 가리키는 per-process open-file table entry 생성
            * file이 열려있지 않으면 보통 memory에 cached 된 directory structure에서 해당하는 file name 찾음, file을 찾으면 system-wide open-file table에 FCB 복사
    1. per-process open-file table에 entry가 만들어지고, 해당 entry는 system-wide open-file table의 해당하는 entry에 pointer 및 file 안의 현재 location에 대한 pointer, access mode 등을 가짐
    1. open()은 per-process open-file table의 entry를 가리키는 pointer 반환
    1. 이후 해당 pointer를 이용해 모든 operation 실행
        * UNIX에서 해당 pointer를 `file descriptor`, Windows에서는 `file handle`이라고 부름
    * ![in-memory file-system structures](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/images/Chapter12/12_03_FileSystemStructures.jpg)

    1. file을 닫을 때면, close()를 호출
    1. per-process table entry가 사라지고, system-wide entry의 open count가 줄어듬
    1. 모든 user가 해당 file을 close할 때, updated된 metadata가 disk 기반 directory structure에 복사되어지고, system-wide open0file table entry가 사라짐

### 2. Partitions and Mounting
* Partition 종류
    * `Raw disk`: file system이 사용되어지지 않음
        * UNIX에서 swap space로 이용 가능
        * database도 이용가능
        * RAID system의 bitmap 등의 정보 저장 가능
    * `Cooked disk`: file system 보유

* Boot information
    * sequential series of blocks로 각각의 partition에 저장
    * memory에 load 시, first byte 같이 predefined된 영역에서 실행
    * boot loader
        * file system을 찾고 kernel을 load해 실행시킴
        * 특정 OS를 어떻게 boot할지에 대한 instruction 보유 (ex. dual boot)

* root partition
    * OS kernel 및 system files 보유
    * boot time에 mounted
    * mount 완료 후, OS가 file system의 valid 여부 확인
    * invalid하면 partition은 user의 intervention 없이 consistency check 및 corrected 되어짐
    * OS가 in-memory mount table에 file system이 mounted 됐다고 기록

* Mount
    * Windows
        * Letter and colon으로 표기 (F: 등)
    * UNIX
        * 아무 directory에 mount 가능
        * i-node의 in-memory 복사본의 flag를 setting함으로써 mounting 가능

### 3. Virtual File Systems
* multiple types of file systems를 지원할 수 있게 해줌
* UNIX에서는 object-oriented technique으로 적용을 단순화하고 조직화하고 모듈화
* Layer 구성
    * file-system interface
        * file descriptor 및 open(), read(), write(), close() 등의 system call을 기반
    * `virtual file system` (`VFS`)
        * 기능
            1. file-system-generic 기능 구분: 여러 type의 local file systems를 투명하게 접근하는 것을 허용
            1. network를 통한 file을 유일하게 표현하는 방식 제공: vnode로 network file system의 file 표현
    * 각 file-system 혹은 remote file system protocol
* VFS는 각 file-system에 맞는 operation을 동작시킴
* Linux VFS에 의해 정의된 4가지 main object
    * inode object: 각각의 file
    * file object: open file
    * superblock object: 전체 file system
    * dentry object: 각 directory entry

## Directory Implementation
### 1. Linear List
* 가장 간단한 구현 방법
* 파일 생성 시, 같은 이름의 파일이 존재하지 않는 것을 확인 후 directory 끝에 entry 추가
* 파일 삭제 시, 파일 찾아서 space release
    * 빈 공간 처리 방법
        * mark unused
        * free directory entries 관리
        * last entry를 빈 space로 가져와 length 줄임

* 단점
    * 작동에 시간 오래 걸림
    * file 찾는데 linear search를 함: sort 하면 시간 줄지만, 매 번 sort하기 복잡

### 2. Hash Table
* 빠른 search 시간과 직관적인 insertion/deletion 가능

* 단점
    * collision 처리 필요
    * table size에 따라 hash function이 달라짐

* chained-overflow hash table 사용
    * 각 hash entry는 linked list로 연결
    * entry를 추가함으로써 collision 제거
    * search 속도는 느려지지만 linear search로 전체를 탐색하는 것보다는 훨씬 빠름

## Allocation Methods
### 1. Contiguous Allocation
* 각 file이 연속된 block을 차지해야 함
* disk address와 길이로 정의
* access가 쉬움
* 단점
    * 새 파일을 위한 space 찾기 어려움
    * dynamic storage-allocation problem 발생: first-fit, best-fit, worst-fit 등
        * external fragmentation 야기
        * file system을 모두 옮긴 후, free space를 compact 후 file 다시 불러오는 것으로 해결 가능하나 off-line이 되어야하고 그로 인해 down-time 발생

### 2. Linked Allocation
* 각 file이 disk block의 linked list 형태로 존재
* 각 directory는 file의 first와 last block을 pointer로 가리킴
* file 생성 시, directory의 새 entry를 추가하고, first block에 연결 후, free block들을 찾아 작성 후 연결
* 단점
    * sequential access file일 경우에만 효과적임
    * pointer에 너무 많은 공간 쓰임
        * cluster라고 불리는 단위로 block을 묶어서 해결 가능하나, internal fragmentation 발생
    * 안정성에서 문제: 포인터가 사라지면 파일 복구 힘듬
* `File-Allocation Table` (`FAT`)
    * MS-DOS에서 사용
    * Linked Allocation의 적용 사례

### 3. Indexed Allocation
* linked allocation에서 pointer들을 index block에 모아놓고 사용
* ![indexed allocation of disk space](https://media.geeksforgeeks.org/wp-content/uploads/indexedAllocation.jpg)
* Index block의 다양한 크기에 대한 해결책
    * Linked scheme: 여러 index blocks를 link
    * Multilevel index: first-level block은 second-level block을 가리키고, second-level block은 각 file block을 가리킴
    * Combined scheme
        * UNIX-based systems에서 사용
        * 첫 12 pointer는 direct block으로 직접 file 가리킴
        * 나머지는 indirect block으로 index block 가리킴
        * double indirect block, triple indirect block은 multilevel index 도입

### 4. Performance


## Free-Space Management
### 1. Bit Vector


### 2. Linked List


### 3. Grouping


### 4. Counting


### 5. Space Maps


## Efficiency and Performance
### 1. Efficiency


### 2. Performance


## Recovery
### 1. Consistency Checking


### 2. Log-Structured File Systems


### 3. Other Solutions


### 4. Backup and Restore


## NFS
### 1. Overview


### 2. The Mount Protocol


### 3. The NFS Protocol


### 4. Path-Name Translation


### 5. Remote Operations


## Example: The WAFL File System

## Summary