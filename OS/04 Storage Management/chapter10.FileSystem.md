# File System

## File Concept
* file: 논리적 저장 단위
1. File Attributes
    * Name: human-readable 형태의 이름
    * Identifier: unique한 tag, non-human-readable name
    * Type: 다른 type의 file을 지원하기 위해 system에 필요한 정보
    * Location: device에서의 file 위치를 가리키는 pointer 정보
    * Size: 현재의 size 혹은 추정되는 최대 size
    * Protection: Access control 정보, read/write/execute
    * Time, date, and user identification: 생성, 최근 수정, 최근 사용일자 저장, protection/security/usage monitoring에 유용
1. File Operations
    * 기본 operation
        * Creating a file
            1. 빈 공간 찾기
            1. new file의 entry가 directory에 생성
        * Writing a file
            1. file 이름과 적힐 정보를 지정해 system call
            1. 해당 file의 location을 찾음
            1. write pointer를 해당 location에 유지해 다음 write가 일어날 장소 확인
            1. write 발생 시 write pointer update
        * Reading a file
            1. file 이름과 저장할 장소를 지정해 system call
            1. 해당 file의 location을 찾음
            1. read pointer를 해당 location에 유지해 다음 read가 일어날 장소 확인
            1. read 발생 시 read pointer update
            * process에서 read/write를 동작시키므로, read/write pointer로 current-file-position pointer를 사용
        * Repositioning within a file == Seek
            1. 해당 location 찾기
            1. 주어진 값으로 current-file-position pointer가 이동
            * 실제 I/O는 필요하지 않음
        * Deleting a file
            1. file 이름을 지정해 찾음
            1. 위치를 찾으면 해당 space release 후 directory entry 지움
        * Truncating a file
            * file의 내용은 지우지만, attributes는 유지하고 싶을 때
            * file length를 제외한 모든 attributes 유지
    * 이 기본 6개 operation들을 이용해 다른 operation을 만들 수 있음
        * Copy: create + read + write
    * open(), close()
        * File operation에 named file과 연계된 directory를 찾는 과정을 포함
        * 모든 file은 이용 전 open()을 호출해야하고, opened 된 파일들에 대해 open-file table로 정보 유지
        * 일부 system은 암묵적으로 file을 열고 닫지만, 대부분의 system은 개발자가 open(), close()를 호출해 처리하게 함
            * access mode: create, read-only, read-write, append-only
            * open-file table에 개체에 대한 pointer를 반환, 이 포인터는 추가 searching을 더 하지 않아도 되게 해줌
            * open(), close() 구현은 여러 process가 file open을 동시에 수행할 때, 더 복잡해짐
    * Two levels of Internal Tables
        * per-process table: process가 연 모든 file 추적, current file pointer 등 process가 file을 사용하는 것과 관련있는 정보 보유
        * system-wide table: disk의 file, 접근 일자, file size 등 process와 무관한 정보 보유
        * process가 open() 호출 시, system-wide table에 해당하는 file entry의 open count 증가, close()시 감소
    * open file과 관련된 정보
        * File Pointer: 각 process의 current-file-position pointer 역할
        * File-open count: file의 재사용을 위해 모든 open count가 0이 되기 전까지 entry 보관
        * Disk location of the file: file 수정 시, file 위치와 관련된 정보를 memory에 보관하여 disk에서 읽지 않도록 함
        * Access rights: per-process table에 보관해 I/O 통제
    * File-locking mechanism
        * Mandatory: process가 exclusive lock을 얻으면 OS가 다른 process의 locked된 file 접근을 막음
        * Advisory: OS가 하는 일은 없고, 자체적으로 locking을 적용해야 함

1. File Types
    * type 구분 방법
        * 이름에 포함: name.extension
        * 이름에 포함 시 OS에게 힌트가 됨
    * executable, object, source code, batch, markup, word processor, library, print or view, archive, multimedia 등으로 구분
    * UNIX system에서는 magic number를 file의 시작에 넣어 type을 대강 구분할 수 있게 함
1. File Structure
    * File type은 File의 internal structure를 가리킴
    * OS가 여러 file structure 지원 시의 단점: 번거로움, 각각에 대한 처리 code 필요, 새로운 file type이 나타났을 때 지원해줘야함
    * 일부 OS는 최소한의 file structure 지원 (UNIX, Windows)
        * 각 application program이 적합한 structure를 해석하도록 함
        * 다만, 최소한 executable file에 대해서는 OS가 지원해줘야함
1. Internal File Structure
    * Disk는 block 단위로 잘 운영됨
    * logical record의 길이는 다양하지만, 원활한 동작을 위해 physical block에 몇 개의 logical records를 packing 하여 맞춤
    * logical record size, physical block size, packing technique을 통해 physical block 하나에 해당하는 logical records의 갯수를 결정할 수 있음
    * packing은 user application 혹은 OS에서 이루어질 수 있음, logical records에서 physical blocks으로 변환하는건 쉬운 문제임
    * disk는 항상 block 단위로 allocated되어지기에, 마지막 block은 낭비되는 경우 잦음 => internal fragmentation
    
## Access Methods
1. Sequential Access
    * file의 information을 순서대로 접근
    * 단순하고 꽤나 잘 쓰임 - editors, compilers의 접근 방법
    * read_next(), write_next()
    * 처음으로 이동할 수 있고, 일부 system에서는 앞뒤로의 이동 지원
    ![sequential access file](https://www.cs.csustan.edu/~john/Classes/CS3750/Notes/Chap13/13_04seqAccess.jpg))

1. Direct Access (Relative Access)
    * file을 block/records의 번호가 매겨진 순서로 여김
    * 접근할 block의 순서를 정할 수 있음
    * read(n), write(n)
    * 여기서 사용하는 block number는 relative block number
        * file의 시작점에서 번호 시작

1. Other Access Methods
    * direct access 기반으로 접근
    * index를 만들어 그 기반으로 접근
    * file이 크면 index file도 커지기에, index에 대한 index도 마련

## Directory and Disk Structure
* Partitioning: file system 각각의 size를 제한하여 여러 file system을 하나의 device에 사용 가능하게 하는 것
* Volume: file system이 포함된 개체
* Device directory / volume table of contents: file system 안에 있는 files 에 대한 정보

1. Storage Structure
    * Solaris의 File system 종류
        * tmpfs - temporary file system: main memory에 생김
        * objfs - virtual file system: debugger가 kernel symbol에 접근할 수 있게 해줌
        * ctfs - virtual file system: process의 시작과 실행을 관장하는 접근 주소 보관
        * lofs - loop back file system: 다른 곳에서도 file system을 접근할 수 있게 해줌
        * procfs - virtual file system: 모든 process의 정보 보관
        * ufs, zfs - general purpose file systems

1. Directory Overview
    * directory: file 이름을 directory entry로 변환해주는 symbol table로 볼 수 있음
    * operation
        * search a file
        * create a file
        * delete a file
        * list a directory
        * rename a file
        * traverse the file system

1. Single-Level Directory
    * 모든 file이 하나의 directory에 담겨 있음
    * 한계
        * 파일이 많을 경우 모두 다른 이름을 유지해야 함
        * user 가 여럿일 경우 모든 파일 이름을 기억하기 쉽지 않음(?)

1. Two-Level Directory
    * User File Directory (UFD): 각 user가 가진 directory, 각 user만을 위한 파일 존재
    * Master File Directory (MFD): user 이름 혹은 account number로 기록, 각 UFD의 진입 point
    * 한계
        * user 끼리 file 공유가 어려움

1. Tree-Structured Directories
    * root directory를 가지고, 이 아래 각 file이 unique한 path 이름을 가짐
    * 각 directory는 file 혹은 subdirectory를 가짐
    * 각 process는 current directory를 가짐
    * path 종류
        * absolute path name: root부터 명시
        * relative path name: current directory부터 명시

1. Acyclic-Graph Directories
    * tree structure에서는 directory 및 file이 공유되지 않았지만, 여기서는 cycle 없이 가능
    * 공유된 file은 copied 된 것이 아닌 shared 된 것
    * file sharing 구현
        * link: file을 가리키는 pointer
        * duplication
    * 한계
        * 하나의 file이 여러 absolute path name 가짐
        * 삭제 시, dangling reference 발생 가능 => symbolic link를 이용해서 해결 혹은 모든 reference가 사라질 때까지 보존 (hardlink 기준)

1. General Graph Directory
    * Acyclic-Graph Directory는 acyclic 보장이 힘듬
    * 한계
        * traverse algorithm이 어려움 => traverse 횟수 제한으로 극복 가능
        * 파일 지울 때 self referencing으로 delete 안 될 수 있음 => garbage collection을 통해 지우고 reallocate 해줌
    * Garbage Collection: file system 전체 탐색하며 marking 후, unmarked 된 곳을 free함

## File-System Mounting
* file처럼 file system도 process에게 사용되기 전에 mounted 되어야 함
* device의 name과 mount point(mount 될 위치)를 받으면 mount 해줌

## File Sharing
1. Multiple Users
    * file/directory의 owner(user)와 group 속성 부여
        * user: file의 속성 및 권한을 바꿀 수 있는 사람
        * group: file의 접근 권한을 공유하는 user들의 subset
1. Remote File Systems
    1. The Client-Server Model
        * network name 혹은 IP 등으로 user 식별 가능하나 spoofed 될 수 있음
        * encrypted key를 이용한 보안 인증이 있지만, 호환성이나 key 교환 보안 문제 등의 어려움 존재
        * DFS (Distributed File System) protocol: remote file system이 mounted 되었을 때, network를 통해 file operation을 전달
    1. Distributed Information Systems
        * Distributed Naming Services라고 불리기도 함
        * remote computing에 필요한 정보에 통일된 접근을 제공
        * Domain Naming Systems(DNS)를 통해 scalable하게 운영될 수 있고, 이전에는 ftp 혹은 email로 운영되었음
        * 여러 방식
            * NIS(Network Information System): 모든 user name, host name, printer information 등의 정보를 중앙화한 주소록(yellow page), 여러 산업체에서 도입
            * CIFS(Common Internet File System): user name과 password를 통해 login을 하게 하여 신원 확인
            * LDAP(Lightweight Directory-Access Protocol): 산업체에서 도입한 안전한 분산 naming 방식, 직원들의 정보 저장

    1. Failure Modes
        * Local File System 문제 원인: disk 문제, metadata 변형, disk controller 문제, cable 문제, host-adapter 문제 등
        * Remote File System 문제 원인: Local File System + network 등의 결합으로 훨씬 많은 원인
        * recovery 적용을 위해 client와 server 양쪽에 state information을 유지해야 함
            * failure 벌어졌을 때, opened 된 file 등에 대한 정보를 유지하면 복구 가능
        * NFS protocol: stateless 방식 사용하여 read/write 등의 명령에 대해 remote file system이 mounted 된 상황이 아니라면 일어나지 않은 일로 처리

1. Consistency Semantics
    * Consistency Semantics는 file sharing을 지원하는 file system을 평가하는 기준
    1. UNIX Semantics
        * file의 수정이 file을 연 다른 user에게 바로 보여짐
        * 원본에 관계없이 모든 access를 interleave(동시 접근)하는 단일 image 보유
    1. Session Semantics
        * file의 수정이 file을 연 다른 user에게 바로 보여지지 않음
        * file이 닫히면 이후 열린 파일에 수정 내역이 반영되어서 보임, 열려있는 파일에는 반영되어지지 않음
    1. Immutable-Shared-Files Semantics
        * 수정될 수 없음
## Protection
1. Types of Access
    * 종류
        * Read
        * Write
        * Execute
        * Append
        * Delete
        * List
    * 각각의 access에 대해 통제

1. Access Control
    * ACL (Access control list)를 이용해 user의 name과 각 user에게 부여된 access 종류 지정
    * 적용에 어려운 점
        * user list를 모르면 일일이 적용하는 것은 귀찮은 일
        * directory entry의 크기가 가변적이어서 관리 복잡
    * 적용을 위한 분류
        * Owner: file 생성자
        * Group: 같은 접근 권한을 가진 user들의 set
        * Universe: 나머지 user
    * 적용
        * UNIX에서는 각 분류당 3bit씩 Read/Write/Execute 권한 부여 (owner/file's group/other users)

1. Other Protection Approaches
    * Password 인증으로 파일 접근하는 방식
        * 단점: 기억하기 어려움, 다 통일하면 보안에 취약
    * directory의 경우, file과 다른 보안 필요
        * file list 접근 제어
        * path name에 대한 접근 제어

## Summary