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
1. Direct Access
1. Other Access Methods

## Directory and Disk Structure
1. Storage Structure
1. Directory Overview
1. Single-Level Directory
1. Two-Level Directory
1. Tree-Structured Directories
1. Acyclic-Graph Directories
1. General Graph Directory

## File-System Mounting

## File Sharing
1. Multiple Users
1. Remote File Systems
    1. The Client-Server Model
    1. Distributed Information Systems
    1. Failure Modes
1. Consistency Semantics
    1. UNIX Semantics
    1. Session Semantics
    1. Immutable-Shared-Files Semantics

## Protection
1. Types of Access
1. Access Control

## Summary