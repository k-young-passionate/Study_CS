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

### Variable

#### static variable

- 클래스 변수

#### instance variable

- 인스턴스 변수

#### local variable

- 지역 변수

```java
class House {
    static String house; // static variable
    String name; // instance variable

    void method() {
        int value = 2; // local variable
    }
}
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
