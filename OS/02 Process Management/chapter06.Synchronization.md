# Synchronization

## Background
* 어떤 상황에서든 interrupt로 인해 다른 process를 실행할 수 있음
* 동시에 두 개 이상의 process가 데이터 접근 및 조작 시 incorrect state 발생 가능
* race condition: 동시에 두 개 이상의 process가 데이터 조작 시 실행 순서에 따라 값이 바뀔 수 있는 상황
    * process synchronization 및 coordination 으로 해결 필요

## The Critical-Section Problem
* critical section: process가 공통 변수를 바꾸거나 table을 update하거나 file을 writing 하는 부분 등의 상황
    * entry section으로 시작해 exit section으로 끝나고, critical section 외 나머지 code는 reaminder section
* 동시에 두 개 이상의 process가 critical section에 접근하지 못하게 할 때, process끼리 cooperate하게 하는 protocol
* 다음 3 가지 조건을 만족해야 함
    * Mutual exclusion: 두 개 이상의 process가 동시에 접근 불가
    * Progress: critical section에 존재하는 process가 없다면, 진입을 원하는 process는 진입 시켜야 함
    * Bounded waiting: process가 진입 요청을 보냈고 이에 진입하기 전까지 다른 process들이 critical section에 들어갈 수 있는 횟수 제한 존재, 즉 언젠간 무조건 들어갈 수 있다는 것
* critical section을 다루기 위한 OS의 두 접근
    * preemptive kernels
        * process가 kernel mode에 있을 때도 자원을 선점 당할 수 있도록 함
        * 동시에 2개의 kernel mode로 운영이 가능하기에 SMP architecutre에 적용 어려움
        * 더 responsive 함
    * nonpreemptive kernels
        * process가 kernel mode에 있을 때는 선점 당하지 못하게 함
        * kernel mode를 나가거나, block, CPU 자원 자진 반납을 할 때까지 kernel mode 실행
        * race condition으로 부터 자유로움

## Peterson's Solution
* 두 process 사이에서의 critical section 접근 문제 해결
```c++
do {
    flag[i] = true; // 나의 request 여부
    turn = j;   // 자신이 더 나중에 진입했는지 확인
    while (flage[j] && turn == j);  // 상대가 request + 자신이 더 나중에 진입 => 대기
        /* critical section exists here */
    flag[i] == false;
        /* remainder section exists here */
} while (true);
```
* Critical section problem의 3 가지 조건 모두 만족

## Synchronization Hardware
* Peterson's Solution은 현대 computer architecture에 보장이 안 됨
* Critical section을 lock하는 locking 을 통해서 해결
* test_and_set 방식
    ```c++
    boolean test_and_set(boolean *target) {
        boolean rv = *target;
        *target = true;

        return rv;
    }

    int main(int argc, char **argv) {
        /* remainder section */
        do {
            while (test_and_set(&lock));    // waiting for gain lock
                /* critical section */
            lock = false;
                /* remainder section */
        } while (true);
    }
    ```
    * test_and_set 중간에 interrupt가 일어날 경우 문제가 생길 수 있음
    * 이를 hardware instruction으로 지정하여 atomical한 실행을 보장함
    * 다만, 이 방식은 mutex를 보장하지만 bounded waiting은 보장 못 함

* compare_and_swap 방식
    ```c++
    int compare_and_swap(int *value, int expceted, int new_value) {
        int temp = *value;

        if (*value == expected)
            *value = new_value
        return temp;
    }
    ```
    * test_and_set과 나머지 동일

* test_and_set을 이용한 bounded-waiting
    ```c++
    do {
        waiting[i] = true;  // lock이 활성화 되어있을 때, i번째 process가 진입 가능한지 여부의 반대
        key = true; // lock이 활성화 되어있는지 여부
        while (waiting[i] && key)
            key = test_and_set(&lock);
        waiting[i] = false;
            /* Critical section */
        j = (i+1) % n;  // size n짜리 queue에서 다음 index 접근
        while ((j != i) && ! waiting[j])    // queue를 check하며 critical section에 접근하려는 process의 index로 전환
            j = (j + 1) % n;

        if (j == i) // 접근하려는 process 없을 시 lock 비활성화
            lock = false;
        else    // 접근하려는 process의 진입을 허용해 둠
            waiting[j] = false;
            /* Remainder section */
    } while (true);
    ```


## Mutex Locks
* OS 설계자들이 만든 critical section problem 해결 software tool
* acquire(), release()로 구성
* acquire 시 busy waiting (process가 살아있는 상태로 대기)이기에 spinlock이라 불림
* spinlock은 resource를 소모하기도 하지만 context switch가 없어 짧은 시간 내에 critical section에 진입이 예상되는 경우 유리

## Semaphores
* 두 개의 atomic operation에 의해서만 접근될 수 있는 integer variable
* 두 개의 operation
    * wait()
        ```c++
        wait(S) {
            while (S <= 0); // busy waiting
            S--;
        }
        ```
    * signal()
        ```c++
        signal(S) {
            S++;
        }
        ```
1. Semaphore Usage
    * 종류
        * counting semaphore: 제한되지 않은 범위
        * binary semaphore: 0 혹은 1, mutex와 같음
1. Semaphore Implementation
    * busy waiting을 막기 위한 접근
        * wait(): S-- 해놓고 접근하지 못하는 상황이면 queue에 추가 후 sleep/block
        * signal(): S++ 후, deque 한 process wakeup
1. Deadlocks and Starvation
    * deadlock: 두 개 이상의 process가 무기한으로 waiting 하는 것
        * 서로 다른 두 개의 process가 다른 semaphore 획득 시, 일어날 수 있음
        * 이로 인해 영원히 waiting 상태가 되는 starvation 발생 가능
1. Priority Inversion
    * priority가 높은 process가 낮은 process에 자원 처리 순서가 밀리는 상황
    * priority-inheritance protocol로 해결
        * 기다리는 가장 높은 priority의 process가 현재 실행 중인 process에게 자신의 priority 상속 및 process의 resource 점유 해제 시 원래의 priority로 복귀
        * 상속받은 priority로 인해 가장 높은 priority의 process는 resource를 더 낮은 process에 뺏기지 않고 바로 획득 가능

## Classic Problems of Synchronization
1. The Bounded-Buffer Problem
    * buffer가 차면 생산자가 저장할 공간이 없어짐
    * semaphore를 통해 buffer가 차있을 경우 생산자의 접근을 통제해 해결 가능
1. Ther Readers-Writers Problem
    * reader는 읽고 싶어하고 동시에 여럿 접근 가능하나, writer는 하나만 접근 가능
    * reader-writer lock: read access와 write access 구분
        * read mode: writer만 locked
        * write mode: 잡은 writer 빼고 모두 locked
1. The Dining-Philosophers Problem
    * 해결법
        * 최대 n-1명만 동시에 앉을 수 있게 설정
        * 양쪽 젓가락이 모두 사용가능할 때 집도록 함
        * asymmetric 방법: 홀수 철학자는 왼쪽-오른쪽, 짝수 철학자는 오른쪽-왼쪽 순으로 집기

## Monitors
1. Monitors Usage
1. Dining-Philosophers Solution Using Monitors
1. Implementing a Monitor Using Semaphores
1. Resuming Processes within a Monitor

## Synchronization Examples

## Alternative Approaches
1. Transactional Memory
1. OpenMP
1. Functional Programming Languages

## Summary