# Effective Java

## 객체 생성과 파괴

### 7. 다 쓴 객체 참조를 해제하라

- 다 쓴 참조(obolete reference) 해제해야 함
- `null` 할당 => grabage collector에 비활성 영역임을 알림

    ```java
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        element[size] = null;
        return result;
    }
    ```

#### 객체 참조를 `null` 처리하는 일은 예외적인 경우여야 한다

#### 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다

#### 캐시 역시 메모리 누수를 일으키는 주범이다

- key 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 경우
  - `WeakHashMap`을 사용
- 보통의 경우 (유효 기간 정확히 정의 어려운 경우)
  - 백그라운드 스레드 이용 (`ScheduledThreadPoolExecuter` 등)
  - `LinkedHashMap`을 이용할 경우 `removeEldestEntry` 메서드 이용해 새 엔트리 추가시 부수 작업으로 수행
  - 더 복잡한 캐시의 경우 `java.lang.ref` 패키지 이용

#### listener/callback

- client가 Callback 등록 후 명확히 해지하지 않으면 계속 쌓여감
- 약한 참조(weak reference)로 저장 `ex) WeakHashMap`

### 8. finalizer와 cleaner 사용은 피하라

- Java가 제공하는 두 가지 객체 소멸자: `finalizer`, `cleaner`
- 객체가 소멸될 때 호출 ()
- C++의 destructor와는 다르고, 같은 역할을 하는 것은 `try-with-resources` 혹은 `try-finally`
- 활용
    1. close 메서드를 호출하지 않는 것에 대비한 안전망
    1. Native Peer와 연결된 객체에서 필요

#### finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요

- java 9에서부터 deprecated API

#### cleaner는 finalizer보다는 덜 위험하지만, 여전이 예측할 수 없고, 느리고, 일반적으로 불필요

- 자신을 수행할 스레드를 제어할 수 있다는 점은 나은 점

#### finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다

- 해당 작업의 신속성은 GC 알고리즘에 달려있음
- class에 `finalizer` 달아두면 인스턴스 자원 회수가 제멋대로 지연될 수 있음

#### 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안 된다

- `ex) shared resources의 lock 해제`
- java 언어 명세는 수행 시점과 수행 여부조차 보장하지 못함
- 관련 메소드
  - `System.gc`, `System.runFinalization`: 실행 가능성을 높여주나 보장은 못함
  - `System.runFinalizersOnExit`, `Runtime.runFinalizersOnExit`: 보장해주나 심각한 결함 존재
- finalizer는 동작 중 중간에 예외 발생 시, 무시하고 작업 중지 - 로직이 끝까지 시행되지 않을 수 있음

#### 심각한 성능 문제 동반

- 안전망 없이 사용 시, 생성에서 수거까지 50배 정도 느려짐
- 안전망과 사용 시, 5배 정도로만 느려짐

#### finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제 야기

- 생성자/직렬화 과정에서 예외 발생 시, 불완전한 객체에서 하위 클래스의 finalizer가 수행될 수 있게 함
- 자신의 참조를 할당해 GC가 수집하지 못하게 막을 수 있음

#### 객체 생성을 막으려면 생성자에서 exception 발생하면 되지만, finalizer가 있다면 final로 선언된 아무일도 하지 않는 finalize 메서드를 만들자

#### AutoCloseable

- finalizer와 cleaner의 대체제
- `AutoCloseable` 구현 후, instance 다 쓸 경우 `close()` 메서드 호출해주면 됨

```java
public class Room implements AutoCloseable {
    ...
    @Override
    public void close() {
        cleanable.clean(); // cleaner는 close() 호출 혹은 GC에 의해 단 한 번만 불림
    }
}
```

### 9. try-finally보다는 try-with-resources를 사용하라

- `InputStream`, `OutputStream`, `java.sql.Connection` 등의 자원은 `close()` 호출이 필요
- 안전망으로 finalizer를 활용하고는 있지만 믿음직스럽지 못함
- 전통적인 자원 닫힘 보장 수단: try-finally

    ```java
    try {
        return br.readLine();
    } finally {
        br.close();
    }
    ```

- 하지만 자원이 여러개 사용된다면, 전통적 방법으로는 중첩 try-finally 사용

    ```java
    try {
        try {
            in.read(buf);
            out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
    ```

- 하지만 finally에서 exception이 발생한다면...? => 혹시나 발생했을 수도 있는 try 문 안의 로그도 못보고, `close()`도 실패
- java 7부터 제공하는 `try-with-resources` 덕에 해결 (AutoCloseable interface를 구현한 자원만 사용 가능)

    ```java
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
        ...
        while ((n = in.read(buf)) >= 0) {
            out.write(buf, 0, n);
        }
    }
    ```

- try문 이후 자동으로 `AutoCloseable.close()` 호출
- catch 문도 사용 가능

# Java 관련 용어 정리

## Reference API

- Garbage Collector와 상호작용 할 수 있도록 특별한 reference를 관리해 줌

### Strong Reference

- GC 대상이 되지 않음
- prime 변수들에 해당

```java
Integer prime = 1;
```

### Soft Reference

- `null` 상태가 되면 GC 대상
- 단, 메모리가 부족하지 않으면 GC 하지 않음

```java
SoftReference<Integer> soft = new SoftReference<Integer>(prime);
```

### Weak Reference

- `null` 상태가 되면 GC 대상
- 다음 GC 때 제거

```java
WeakReference<Integer> soft = new WeakReference<Integer>(prime);
```

### ref

- <http://www.pawlan.com/monica/articles/refobjs/>
- <http://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/>

## Finalizer

```java
public class JavafinalizeExample1 {  
     public static void main(String[] args)   
    {   
        JavafinalizeExample1 obj = new JavafinalizeExample1();   
        System.out.println(obj.hashCode());   
        obj = null;   
        // calling garbage collector    
        System.gc();   
        System.out.println("end of garbage collection");   
  
    }   
    @Override  
    protected void finalize()   
    {   
        System.out.println("finalize method called");   
    }   
}  
```

### ref

- <https://www.javatpoint.com/java-object-finalize-method>
- <https://www.geeksforgeeks.org/finalize-method-in-java-and-how-to-override-it/>

## Cleaner

```java
 public class CleaningExample implements AutoCloseable {
    // A cleaner, preferably one shared within a library
    private static final Cleaner cleaner = <cleaner>;

    static class State implements Runnable {

        State(...) {
            // initialize State needed for cleaning action
        }

        public void run() {
            // clean 작업 수행
        }
    }

    private final State;
    private final Cleaner.Cleanable cleanable

    public CleaningExample() {
        this.state = new State(...);
        this.cleanable = cleaner.register(this, state);  // clean 작업 등록
    }

    public void close() {
        cleanable.clean();
    }
}
```

### ref

- <https://docs.oracle.com/javase/9/docs/api/java/lang/ref/Cleaner.html>
- <https://blog.javarouka.me/2018/11/26/Finalizer%EC%99%80-Cleaner/>

## Native Peer

- 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
- 자바객체가 아니므로 GC가 그 존재를 알지 못함
- `ex) JFrame`

### ref

- <https://stackoverflow.com/questions/48260485/what-is-a-native-peer>
