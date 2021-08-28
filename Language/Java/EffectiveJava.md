# 객체 생성과 파괴

## 1. 생성자 대신 정적 팩터리 메서드를 고려하라

- 클래스 생성 수단: public constructor, static factory method

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

### 장점

#### 1. 이름을 가질 수 있다

- 이름을 통해 객체 특성 예측 가능
- 다른 시그니처로 다른 객체 반환시 헷갈릴 수 있음

#### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다

- immutable class는 인스턴스를 미리 만들어 놓거나 캐싱하여 재활용하여 불필요한 객체 생성 방지
- Flyweight pattern과 비슷
- 성능에 좋음
- `instance-controlled`(인스턴스 통제): 언제 어느 인스턴스 생존할지 통제 가능
  - 싱글턴(singleton), 인스턴스화 불가(noninstantiable)로 만들 수 있음
  - 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장

#### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다

- 반환할 객체의 클래스를 자유롭게 선택 가능 => 구현 클래스를 공개하지 않고 그 객체 반환 가능
- 정적 팩터리 매서드를 사용하는 클라이언트는 얻은 객체를 구현 클래스가 아닌 인터페이스만으로 다루게 됨 (좋은 습관)

#### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

- 반환 타입의 하위 타입이면 어떤 클래스의 객체를 반환하든 상관 없음
- 사용하는 클라이언트의 입장에서는 어떤 클래스인지 알 수 없고 알 필요 없음
- 내부에서 반환 객체 변경해도 문제 생기지 않음

#### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. => 이해 잘 안되어서 나중에 다시 읽어보기

- 서비스 제공자 프레임워크(Service provider Framework) 만드는 근간 => JDBC 등
- 서비스 제공자 프레임워크
  - 서비스 인터페이스: 구현체 동작 정의
  - 제공자 등록 API: 제공자가 구현체 등록할 때 사용
  - 서비스 접근 API: 클라이언트가 서비스 인스턴스 얻을 때 사용, 유연한 정적 팩터리의 실체

### 단점

#### 1. 상속을 하려면 public이나 protected 생성자가 필요하니, 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다

#### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다

```Java
// 자주 사용되는 명명 방식
from(instance);  // 매개변수 받아서 해당 타입의 instance 반환
of(parameter1, parameter2);  // 여러 매개변수를 받아 적합한 타입의 instance 반환
valueOf();  // from 및 of의 더 자세한 버전
instance(); getInstance();  // 매개변수로 명시한 instance 반환 (같은 instance을 보장X)
create(); newInstance();  // 매개변수로 명시한 instance의 새로운 instance 반환
getType();  // getInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드 정의할 때 쓰임
newType();  // newInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드 정의할 때 쓰임
type();  // getType, newType의 간결한 버전
```

## 2. 생성자에 매개변수가 많다면 빌더를 고려하라

### 점증적 생성자 패턴(telescoping constructor pattern)

- 매개변수 수를 늘려가며 생성자를 모두 선언하는 방식

#### 점층적 생성자 패턴도 쓸 수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다

- 실수 유발

### 자바빈즈 패턴(JavaBeans Pattern)

- 매개변수 없는 생성자 + `setter` 이용

#### 객체 하나를 만드려면 메서드 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성(consistency)이 무너진 상태에 놓이게 된다

- 생성자에서 일관성 확인이 불가해 매 `setter`에서 확인

#### 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며

- public으로 `setter` 가 오픈되어있어 불변 불가
- 스레드 안전성을 얻으려면 프로그래머의 추가 작업 필요
- 사용 전 `freeze` 메서드 호출을 통해 단점 완화 가능하지만 `freeze` 호출 강제화 불가

### Builder Pattern

- 안전성(점층적 생성자 패턴) + 가독성(자바빈즈 패턴)
- 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자/정적팩터리를 호출해 빌더 객체를 얻고, 빌더 객체의 `setter` 사용
- 메서드 연쇄(method chaining) / 플루언트 API(fluent API)

```Java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)    // 필수 매개변수 세팅
    .calories(100).sodium(35).build();                          // 선택 매개변수 세팅
```

#### 빌더 패턴은 명명된 선택적 매개변수(named optional parameters)를 흉내낸 것

- 불변식(invariant): 프로그램이 실행되는 동안 반드시 만족해야하는 조건

#### 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다

- 추상 클래스에는 추상 빌더, 구체 클래스(concrete class)에는 구체 빌더

### 기타 정리할 용어

- simulated self-type
- covariant return typing (구체 반환 타이핑)

## 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

- singleton: 인스턴스를 오직 하나만 생성할 수 있는 Class

#### 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다

- mock으로 대체할 수 없기 때문

### singleton 구현 방식

- 공통 방식: 생성자는 private으로 감춰두고, 접근 수단으로 public statc 멤버를 마련해 둠
- 클라이언트는 이를 통제할 수 없으나, 예외적으로 refelction API인 `AccessibleObject.setAccessible`로 `private` 생성자 호출

1. `public static final`로 선언
  - 클라이언트가 singleton임이 API에 명백히 드러남
  - 간결함
  ```java
  public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
      public void leaveTheBuilding() { ... }
  }
  ```

1. 정적 팩터리 방식
    - 항상 같은 객체의 참조 반환
    - 마음이 바뀌면 API 유지하고 싱글턴이 아닌 구조로 바꿀 수 있음
    - generic singleton factory로 만들 수 있음

    ```java
    public class Elvis {
        private static final Elvis INSTANCE = new Elvis();
        private Elvis() { ... }
        public static Elvis getInstance() { return INSTANCE; }

        public void leaveTheBuilding() { ... }
    }
    ```

1. 원소가 하나인 열거 타입 선언
    - 더 간결
    - 추가 노력 없이 직렬화 가능
    - 제 2의 인스턴스가 생기는 일을 막아 줌

#### 대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다

    - 다만 `Enum` class 외의 클래스를 상속해야 한다면, 사용할 수 없음

    ```java
    public enum Elvis {
        INSTANCE;

        public void leaveTheBuilding() { ... }
    }
    ```

### Serialize 방식

- 모든 instance를 transient(일시적)이라고 선언 후 `readResolve` 메서드 제공해야 함
  - 이렇게 하지 않으면 역직렬화시 새로운 인스턴스가 만들어짐

    ```java
    private Object readResolve() {
        // 가짜 Elvis는 garbage collector에 맡김
        return INSTANCE;
    }
    ```

## 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

### 정적 메서드/필드만 사용하는 클래스

- 객체지향적이지는 않지만 쓰임새가 있음
  - `java.lang.Math`, `java.util.Arrays`
  - 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드/팩터리 모아 둠
  - final 클래스와 관련된 메서드 모아놓음

#### 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다

#### private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다

## 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

#### 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다

#### 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식

- 이 패턴의 쓸만한 변형으로 생성자에 자원 팩터리를 넘겨주는 패턴이 있다: Factory Method Pattern

## 6. 불필요한 객체 생성을 피하라

- 똑같은 기능의 객체를 매번 생성하기보다는 객체하나를 재사용하는 편이 나을 때가 많다.

```java
String s = new String("bikini");  // 코드 실행 시, 매 번 객체 생성
String s = "bikini"  // 하나의 인스턴스 재사용 보장
```

- 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있음

```java
Boolean.valueOf(String);
```

### 문자열 유효 로직 예제

```java
// AS-IS
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})");
}

// TO-BE
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(...);

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

#### `String.matches`는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기 적합하지 않다

- 또한, lazy initialization(지연 초기화)를 통해 처음 호출될 때 불필요한 초기화를 없앨 수 있지만 권하지 않음 => 성능이 크게 개선되지 않을 때가 많음

### 훨씬 덜 명확하거나, 직관에 반대되는 상황도 있다

- Adaptor(View)
  - 실제 작업을 뒷단 객체에 위임하고, 자신은 제 2의 인터페이스 역할을 해주는 객체
  - 뒷단 객체 외에는 관리할 상태가 없으므로, 뒷단 객체 하나당 어댑터 하나만 만들어지면 충분

### 불필요한 객체를 만들어내는 예

- auto boxing
  - 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어쓸 때 자동으로 상호 변환해주는 기술

#### auto boxing은 기본타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다

```java
Long sum = 0L;
for (long i=0; i<=Integer.MAX_VALUE; i++)
    sum += i;  // Long 객체가 Integer.MAX_VALUE 번 생성해 매우 느려짐
```

#### 박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자

### 객체 풀을 만들어 갯수를 줄이는게 좋은 예

- DB 연결
- 보통의 경우 JVM이 잘 되어있어서 빠름 => 객체 풀 유지할 필요 없음

#### 필요 없는 객체를 반복 생성했을 때 피해 <<<넘사벽<<< defensive copy(방어적 복사)가 필요한 상황에서의 객체 재사용 피해

## 7. 다 쓴 객체 참조를 해제하라

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

## 8. finalizer와 cleaner 사용은 피하라

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

## 9. try-finally보다는 try-with-resources를 사용하라

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
