# Terms

## C

### Callback Framework

- 자기 자신의 참조를 다른 객체에 넘겨서 특정 이벤트의 callback에 이용하게 함

### Covariant Return Type

- 공변 반환 타입
- 반환타입을 Subclass로 Override 가능
- `java 1.5`부터 적용

## D

### Data warehouse

- 데이터를 추출, 변환, 요약하여 능동적으로 사용자에게 제공할 수 있는 데이터의 집합체

### Decorator pattern

- 주어진 상황 및 용도에 따라 어떤 객체에 책임을 덧붙이는 패턴

## E

### Equality

#### Logical Equality

- 논리적 동치성
- instance 변수 값이 일치

#### Object Equality

- 객체 식별성(Object Identity)이 일치
- 정확히 같은 instance

## F

### Failure Atomicity

- 객체의 메서드가 예외를 발생시킨 이후에도 객체를 사용할 수 있어야 함
- ref: [Stack Overflow](https://stackoverflow.com/questions/29842845/what-is-failure-atomicity-used-by-j-bloch-and-how-its-beneficial-in-terms-of-i)

### First-Class Object

- 일급 객체
- 다른 객체들에 일반적으로 적용되는 연산을 모두 지원하는 객체
- 보통 함수에 매개변수로 넘기기, 수정하기, 변수에 대입하기와 같은 연산을 지원할 때 일급 객체라 함
- ref: [wiki](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4)

## H

### Hadoop

- 분산 처리를 위한 프레임워크
- [HDFS](#HDFS)와 [MapReduce](#MapReduce) 구현

### HBase

- 비관계형 분산 데이터베이스
- [HDFS](#HDFS)위에서 동작

### HDFS

- [Hadoop](#Hadoop) 분산 파일 시스템
- [Hadoop](#Hadoop) 프레임워크를 위해 여러 기계에 대용량 파일을 나눠서 저장

### Hive

- [Hadoop](#Hadoop)에서 동작하는 [Data warehouse](#Data-warehouse)

### Hue

- [Hadoop](#Hadoop) User Experience
- 웹 기반 [Hadoop](#Hadoop) 인터페이스
- [Hive](#Hive) 작업 및 [Spark](#Spark) 작업 등을 실행 가능

## J

### JPA

- Java Persistent API
- RDB의 관리를 표현한 Java API
- Hibernate 등이 지원

### jQuery

- [HTML](#HTML)의 조작을 단순화한 [javascript](#Javascript) [library](#Library)
- [DOM](#DOM), event, 특수 효과, [ajax](#Ajax) 함수 포함

```HTML
<script type="text/javascript" src="path/to/jQuery.js"></script> // 웹페이지에 포함
```

## K

### Kafka

- pub-sub 모델의 [Message Queue](#Message-queue)
- publisher와 subscriber 모두 topic만 바라보는 구조

## L

### Liskov substitution principle

- 리스코프 치환 원칙
- 어떤 타입에 있어 중요한 속성이면, 하위 타입에서도 중요함
- 상위 타입의 객체를 하위 타입의 객체로 치환할 수 있어야 함
- ref: [wiki](https://ko.wikipedia.org/wiki/%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84_%EC%B9%98%ED%99%98_%EC%9B%90%EC%B9%99)

## M

### MapReduce

- Map: 흩어진 데이터를 key-value 값으로 묶어줌
- Reduce: key를 중심으로 필터링/정렬

### Method

- Class에서 생성된 Instance와 관련된 동작 정의
- 데이터와 멤버 변수에 대한 접근 권한 가짐

### Mixin

- 다른 클래스의 부모클래스가 되지 않으면서 다른 클래스에서 사용할 수 있는 메서드를 포함하는 클래스
- 장점
  - 필요한 기능만 상속 가능
  - 다중 상속 가능
  - 코드 재사용성: 여러 class의 기능을 하나로 묶음
- ref: [wiki](https://ko.wikipedia.org/wiki/%EB%AF%B9%EC%8A%A4%EC%9D%B8)

### Mutable Companion Class

- Immutable class에서 mutable class를 통해 연산해 제공해주는 역할

```java
public final class Money {
  private BigDecimal amount;

  public Money(BigDecimal bigDecimal) {
   this.amount = bigDecimal;
  }

   public BigDecimal getAmount() {
    return this.amount;
   }

 // Notice the exclusively accessibility
 public Money add(BigDecimal money) {
  return new Money(this.amount.add(money));
 }

 private MoneyMutable getMutableVersion() {
  return new MoneyMutable(this.amount);
 }

 public Money complexOperation(BigDecimal money) {
  MoneyMutable mutableMoney = this.getMutableVersion();
  mutableMoney.add(money);
  mutableMoney.divide(money);
  return new Money(mutableMoney.getAmount());
 }
}
```

## N

### Named Optional Parameters

- 이름을 가진 선택적 인자
- python, scala 등에 존재

```python
def my_func(a, b, name="default name"):
    return 

my_func("a", "b")  # name 인자를 넣지 않아도 됨
my_func("a", "b", name="new name")  # named parameter
```

## O

### ORM

- Object Relation Mapping
- 가상 객체 데이터베이스 구축
- 데이터를 객체로 변환

## P

### Presto

- 분산된 [SQL](#SQL) query 엔진
- [Hive](#Hive)와 [MapReduce](#MapReduce)에 비해 훨씬 빠름

## R

### RDD

- `Resilient Distributed DataSet`
- 여러 노드에 걸쳐 저장되는, 변경 불가능한 데이터 집합
- 하나의 RDD는 여러 개의 파티션으로 분리
- 연산
  - Transformation: 기존 RDD로 filter/map 등을 이용해 새 RDD 생성
  - Action: RDD 기반으로 계산해서 결과 생성 (count 등)

### Referential Transparency

- 참조 투명성
- 함수가 함수 외부에 영향을 받지 않는 것
- 함수의 결과는 파라미터에만 의존
- 참조 투명성을 지닌 코드는 다음과 같은 특징을 가짐
  - self-contained: 자기 충족적, 외부에 의존하는 코드가 없음
  - deterministic: 결정론적, 동일한 parameter에 동일한 결과
  - no exception: out of memory, stack overflow 외의 exception 발생 X
  - 다른 코드가 예기치 않는 실패하지 않게 함 (매개변수 변경 및 외부 데이터 변경 X)
  - db, fs, network 등의 외부기기로 인한 동작이 멈추지 않음
- ref: [Jinwoo's Blog](https://jinwooe.wordpress.com/2017/12/21/%EB%B6%80%EC%88%98-%ED%9A%A8%EA%B3%BC-side-effect-%EC%B0%B8%EC%A1%B0-%ED%88%AC%EB%AA%85%EC%84%B1-referential-transparency/)

## S

### Spark

- 빅데이터를 분석하는 애플리케이션의 성능을 향상시키는 오픈소스 병렬 처리 [Framework](#Framework)
- 너무 크거나 복잡한 데이터 처리

### SQL Aggregation

- sql의 집계함수
- GROUP BY 절과 함께 사용
- APPROX_COUNT_DISTINCT, AVG, COUNT, GROUPING, MAX, MIN, SUM, 등

### Static Factory Method

- 객체를 생성하는 메서드를 `static`으로 선언한 것
- 다음과 같은 이름으로 자주 명명됨

    ```Java
    from();  // 매개변수 받아서 해당 타입의 instance 반환
    of();  // 여러 매개변수를 받아 적합한 타입의 instance 반환
    valueOf();  // from 및 of의 더 자세한 버전
    instance(); getInstance();  // 매개변수로 명시한 instance 반환 (같은 instance을 보장X)
    create(); newInstance();  // 매개변수로 명시한 instance의 새로운 instance 반환
    getType();  // getInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드 정의할 때 쓰임
    newType();  // newInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드 정의할 때 쓰임
    type();  // getType, newType의 간결한 버전
    ```

## T

### Template engine

- template 양식에 데이터 입력 자료를 합성하여 결과 문서 출력하는 소프트웨어
- jsp, thymeleaf 등

### Template method pattern

- 알고리즘의 구조를 메소드에 정의하고 (메서드 호출 순서 등), 하위 클래스에서 알고리즘 구조의 변경없이 (호출되는 각 메서드의) 알고리즘을 재정의 하는 패턴
  ```java
  public abstract class AbstractClass {
    
    protected abstract void hook1();
    
    protected abstract void hook2();
    
    public void templateMethod() {
        hook1();
        hook2();
    }
    
  }

  // 상속
  public class ConcreteClass extends AbstractClass {

    @Override
    protected void hook1() {
        System.out.println("ABSTRACT hook1 implementation");
    }

    @Override
    protected void hook2() {
        System.out.println("ABSTRACT hook2 implementation");
    }

  }

  // 사용
  concreteClass.templateMethod();
  ```

- ref: [YABOONG](https://yaboong.github.io/design-pattern/2018/09/27/template-method-pattern/)

### Thread safe

- 여러 [thread](../OS/CS_TextBook/02%20Process%20Management/chapter04.MultithreadedProgramming.md#Overview)로부터 동시에 접근될 경우, 프로그램 실행에 문제가 없음을 의미
- 종류
  - Thread safe: [race condition](../OS/CS_TextBook/02%20Process%20Management/chapter06.Synchronization.md#Background)으로 부터 자유로움
  - Conditionally safe: 각 thread가 서로 다른 객체에는 자유롭게 접근 가능하고, shared data에 대해서 [race condition](../OS/CS_TextBook/02%20Process%20Management/chapter06.Synchronization.md#Background)으로부터 보호됨
  - Not thread safe: 동시에 여러 thread가 접근할 수 없음
- thread safe를 적용 방법
  - Re-enterancy
  - Thread-local storage
  - Immutable objects
  - Mutual exclusion
  - Atomic operations

- ref: [곰팡이 먼지연구소](https://gompangs.tistory.com/entry/OS-Thread-Safe%EB%9E%80), [wiki](https://en.wikipedia.org/wiki/Thread_safety)

## V

### Vagrant

- portable한 virtual 소프트웨어 개발 환경의 생성 및 유지보수를 위한 open source software
- 명령어
  - `vagrant up` : 설치된 virtual machine 시작

## Z

### Zeppelin

- web 기반 notebook style의 데이터 분석 툴
- 코드 작성 - 실행 - 결과 확인 - 코드 수정 가능

### [Znode](../Tool/kafka.md#Znode)

### ZooKeeper

- 분산 시스템을 위한 네이밍 레지스트리 제공
- 정보 공유, 상태 체크, 동기화 처리 프레임워크
