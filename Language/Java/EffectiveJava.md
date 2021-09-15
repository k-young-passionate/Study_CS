# 객체 생성과 파괴

## 1. 생성자 대신 정적 팩터리 메서드를 고려하라

- 클래스 생성 수단: public constructor, static factory method

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

### 장점

**1. 이름을 가질 수 있다.**

- 이름을 통해 객체 특성 예측 가능
- 다른 시그니처로 다른 객체 반환시 헷갈릴 수 있음

**2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.**

- immutable class는 인스턴스를 미리 만들어 놓거나 캐싱하여 재활용하여 불필요한 객체 생성 방지
- Flyweight pattern과 비슷
- 성능에 좋음
- `instance-controlled`(인스턴스 통제): 언제 어느 인스턴스 생존할지 통제 가능
  - 싱글턴(singleton), 인스턴스화 불가(noninstantiable)로 만들 수 있음
  - 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장

**3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.**

- 반환할 객체의 클래스를 자유롭게 선택 가능 => 구현 클래스를 공개하지 않고 그 객체 반환 가능
- 정적 팩터리 매서드를 사용하는 클라이언트는 얻은 객체를 구현 클래스가 아닌 인터페이스만으로 다루게 됨 (좋은 습관)

**4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.**

- 반환 타입의 하위 타입이면 어떤 클래스의 객체를 반환하든 상관 없음
- 사용하는 클라이언트의 입장에서는 어떤 클래스인지 알 수 없고 알 필요 없음
- 내부에서 반환 객체 변경해도 문제 생기지 않음

**5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. => 이해 잘 안되어서 나중에 다시 읽어보기.**

- 서비스 제공자 프레임워크(Service provider Framework) 만드는 근간 => JDBC 등
- 서비스 제공자 프레임워크
  - 서비스 인터페이스: 구현체 동작 정의
  - 제공자 등록 API: 제공자가 구현체 등록할 때 사용
  - 서비스 접근 API: 클라이언트가 서비스 인스턴스 얻을 때 사용, 유연한 정적 팩터리의 실체

### 단점

**1. 상속을 하려면 public이나 protected 생성자가 필요하니, 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.**

**2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.**

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

**점층적 생성자 패턴도 쓸 수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.**

- 실수 유발

### 자바빈즈 패턴(JavaBeans Pattern)

- 매개변수 없는 생성자 + `setter` 이용

**객체 하나를 만드려면 메서드 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성(consistency)이 무너진 상태에 놓이게 된다.**

- 생성자에서 일관성 확인이 불가해 매 `setter`에서 확인

**자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며.**

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

**빌더 패턴은 명명된 선택적 매개변수(named optional parameters)를 흉내낸 것.**

- 불변식(invariant): 프로그램이 실행되는 동안 반드시 만족해야하는 조건

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.**

- 추상 클래스에는 추상 빌더, 구체 클래스(concrete class)에는 구체 빌더

### 기타 정리할 용어

- simulated self-type
- covariant return typing (구체 반환 타이핑)

## 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

- singleton: 인스턴스를 오직 하나만 생성할 수 있는 Class

**클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.**

- mock으로 대체할 수 없기 때문

### singleton 구현 방식

- 공통 방식: 생성자는 private으로 감춰두고, 접근 수단으로 public static 멤버를 마련해 둠
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

**대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.**

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

**추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.**

**private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.**

## 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

**사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**

**인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**

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

**`String.matches`는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기 적합하지 않다.**

- 또한, lazy initialization(지연 초기화)를 통해 처음 호출될 때 불필요한 초기화를 없앨 수 있지만 권하지 않음 => 성능이 크게 개선되지 않을 때가 많음

### 훨씬 덜 명확하거나, 직관에 반대되는 상황도 있다

- Adaptor(View)
  - 실제 작업을 뒷단 객체에 위임하고, 자신은 제 2의 인터페이스 역할을 해주는 객체
  - 뒷단 객체 외에는 관리할 상태가 없으므로, 뒷단 객체 하나당 어댑터 하나만 만들어지면 충분

### 불필요한 객체를 만들어내는 예

- auto boxing
  - 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어쓸 때 자동으로 상호 변환해주는 기술

**auto boxing은 기본타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다.**

```java
Long sum = 0L;
for (long i=0; i<=Integer.MAX_VALUE; i++)
    sum += i;  // Long 객체가 Integer.MAX_VALUE 번 생성해 매우 느려짐
```

**박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.**

### 객체 풀을 만들어 갯수를 줄이는게 좋은 예

- DB 연결
- 보통의 경우 JVM이 잘 되어있어서 빠름 => 객체 풀 유지할 필요 없음

**필요 없는 객체를 반복 생성했을 때 피해 <<<넘사벽<<< defensive copy(방어적 복사)가 필요한 상황에서의 객체 재사용 피해.**

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

**객체 참조를 `null` 처리하는 일은 예외적인 경우여야 한다.**

**자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.**

**캐시 역시 메모리 누수를 일으키는 주범이다.**

- key 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 경우
  - `WeakHashMap`을 사용
- 보통의 경우 (유효 기간 정확히 정의 어려운 경우)
  - 백그라운드 스레드 이용 (`ScheduledThreadPoolExecuter` 등)
  - `LinkedHashMap`을 이용할 경우 `removeEldestEntry` 메서드 이용해 새 엔트리 추가시 부수 작업으로 수행
  - 더 복잡한 캐시의 경우 `java.lang.ref` 패키지 이용

**listener/callback**

- client가 Callback 등록 후 명확히 해지하지 않으면 계속 쌓여감
- 약한 참조(weak reference)로 저장 `ex) WeakHashMap`

## 8. finalizer와 cleaner 사용은 피하라

- Java가 제공하는 두 가지 객체 소멸자: `finalizer`, `cleaner`
- 객체가 소멸될 때 호출 ()
- C++의 destructor와는 다르고, 같은 역할을 하는 것은 `try-with-resources` 혹은 `try-finally`
- 활용
    1. close 메서드를 호출하지 않는 것에 대비한 안전망
    1. Native Peer와 연결된 객체에서 필요

**finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요**

- java 9에서부터 deprecated API

**cleaner는 finalizer보다는 덜 위험하지만, 여전이 예측할 수 없고, 느리고, 일반적으로 불필요**

- 자신을 수행할 스레드를 제어할 수 있다는 점은 나은 점

**finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.**

- 해당 작업의 신속성은 GC 알고리즘에 달려있음
- class에 `finalizer` 달아두면 인스턴스 자원 회수가 제멋대로 지연될 수 있음

**상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안 된다.**

- `ex) shared resources의 lock 해제`
- java 언어 명세는 수행 시점과 수행 여부조차 보장하지 못함
- 관련 메소드
  - `System.gc`, `System.runFinalization`: 실행 가능성을 높여주나 보장은 못함
  - `System.runFinalizersOnExit`, `Runtime.runFinalizersOnExit`: 보장해주나 심각한 결함 존재
- finalizer는 동작 중 중간에 예외 발생 시, 무시하고 작업 중지 - 로직이 끝까지 시행되지 않을 수 있음

**심각한 성능 문제 동반**

- 안전망 없이 사용 시, 생성에서 수거까지 50배 정도 느려짐
- 안전망과 사용 시, 5배 정도로만 느려짐

**finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제 야기**

- 생성자/직렬화 과정에서 예외 발생 시, 불완전한 객체에서 하위 클래스의 finalizer가 수행될 수 있게 함
- 자신의 참조를 할당해 GC가 수집하지 못하게 막을 수 있음

**객체 생성을 막으려면 생성자에서 exception 발생하면 되지만, finalizer가 있다면 final로 선언된 아무일도 하지 않는 finalize 메서드를 만들자.**

**AutoCloseable**

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
- 같이 읽어볼 것:<https://ryan-han.com/post/java/try_with_resources/>

# 모든 객체의 공통 메서드

## 10. `equals`는 일반 규약을 지켜 재정의하라

### 재정의하지 않아야 하는 경우

**각 인스턴스가 본질적으로 고유하다.**

- 값을 표현하는 것이 아닌, 동작하는 개체
- ex) `Thread`

**인스턴스의 '[논리적 동치성(logical equality)](../../Terms/Terms.md#logical-equality)'을 검사할 일이 없다.**

**상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**

**클래스가 `private`이거나 `package-private`이고, `equals` 메서드를 호출할 일이 없다.**

### `equals`를 재정의 해야할 때

- [Object identity](../../Terms/Terms.md#object-identity)이 아닌 [logical equality](../../Terms/Terms.md#logical-equality)를 검사해야 하는데, 재정의되지 않았을 경우
- [logical equality](../../Terms/Terms.md#logical-equality)를 검사하도록 재정의 했을 경우, `Map`의 key와 `Set`의 원소로 이용 가능
- `enum` 같이 값이 같은 instance가 만들어지지 않는다는 보장 시, 재정의 하지 않아도 됨

### `equals` 일반 규약

- reflexivity(반사성): `null`이 아닌 모든 참조 값 `x`에 대해, `x.equals(x)`는 `true`
- symmetry(대칭성): `null`이 아닌 모든 참조 값 `x`, `y`에 대해, `x.equals(y)`가 `true`이면 `y.equals(x)`도 `true`
- transivity(추이성): `null`이 아닌 모든 참조 값 `x`, `y`, `z`에 대해, `x.equals(y) == true` && `y.equals(z) == true` 이면 `x.equals(z) == true`
- consistency(일관성): `null`이 아닌 모든 참조 값 `x`, `y`에 대해, `x.equals(y)`는 항상 같은 값 return
- null 아님: `null`이 아닌 모든 참조 값 x에 대해, `x.equals(null)`은 항상 `false`

**`equals` 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.**

**`equals`의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.**

### `equals` 구현 방법

1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
  - 비교 작업이 복잡할 때 도움
1. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
  - 올바른 타입이 아니면 `false` 반환
1. 입력을 올바른 타입으로 형변환한다.
1. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

### 주의사항

- `equals`를 재정의할 땐 `hashCode`도 반드시 재정의
- 너무 복잡하게 해결하려 들지 말자.
- `Object` 외의 타입을 매개변수로 받는 `equlas` 메서드는 선언하지 말자.
  ```java
  public boolean equals(Object o) {  // Object type 이어야만 유효한 상속
    ...
  }
  ```

## 11. `equals`를 재정의하려거든 `hashCode`도 재정의하라

**`equals`를 재정의한 클래스 모두에서 `hashCode`도 재정의해야 한다.**

- 그렇지 않으면 `hashCode` 일반 규약을 어기게 되어 `HashMap`, `HashSet` 등에 사용시 문제 발생
- `equals`가 두 객체를 같다고 판단했으면, `hashCode`도 같아야 함

**성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.**

- hash 품질이 나빠짐

**hashCode가 반환하는 값의 생성규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.**

## 12. `toString`을 항상 재정의하라

**`toString`을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.**
- 직접 호출하지 않더라도 다른 어딘가에 쓰일 수 있음

**실전에서 `toString`은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.**

### 포맷 문서화하기
- 장단점
  - 장점: 표준적, 명확, human-readable
  - 단점: 한 번 명시하면, 계속 얽매이게 됨 (바꿀 수 없음)
- **포맷을 명시하든 아니든 여러분의 의도는 명확히 밝혀야 한다.**
- **`toString`이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.**
  - 그렇게 하지 않으면 프로그래머는 `toString`을 parsing 해야함

## 13. `clone` 재정의는 주의해서 진행하라

### `Clonable`

- 복제해도 되는 class임을 명시하는 mixin interface
- method가 없음
- clone method가 선언된 곳은 Object이고 `protected`로 선언해 외부에서 호출 불가
- `clone`의 동작 방식 결정: `Clonable` 구현한 instance에서 `clone`을 호출하면 해당 객체의 필드를 하나하나 복사한 객체 반환
- `Clonable`을 구현하지 않은 class에서는 `CloneNotSupportedException`을 던짐
- **실무에서 Cloneable을 구현한 class는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.**
- **사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야한다.**
- instance field가 final이면 작동하지 않음
  - **Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌**
- 가변 상태를 공유하지 않아야 함
- `public`인 **`clone` method에서는 throws 절을 없애야 함**
- 상속용 class는 `Cloneable`을 구현해서는 안됨


### 복사

```java
x.clone() != x; // 참
x.clone().getClass() == x.getClass(); // super.clone() 을 호출한다면 참
x.clone().equals(x); // 일반적으로 참
```

#### `Cloneable`을 사용하는 것보다는 복사 생성자와 복사팩터리를 이용하는 것을 추천

- 복사 생성자: 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자

## 14. `Comparable`을 구현할지 고려하라

### `compareTo`

- `Comparable`의 method
- `equals`와 다른 점
  - 동치 + 순서까지 비교 가능 => `sort` 가능
  - generic 하다
- 규약
  ```java
  sgn(x.compareTo(y)) == -sgn(y.compareTo(x));
  x.compareTo(y) > 0; y.compareTo(z) > 0; x.compareTo(z) > 0; // 앞 두개가 참이면 참
  x.compareTo(y) == 0; sgn(x.compareTo(z)) == sgn(y.compareTo(z)); // 앞이 참이면 참
  (x.comapreTo(y) == 0) == (x.equals(y)); // 필수는 아니지만 지키는 것이 좋음
  ```
- 비교자 생성 메서드를 이용하는 방식도 존재: 코드는 깔끔, 약간의 성능 저하
  ```java
  private static final Comparator<PhoneNumber> COMPARATOR = 
                  comparingInt((PhoneNumber pn) -> pn.areaCode)
                  .thenComparingInt(pn -> pn.prefix)
                  .thenComparingInt(pn -> pn.lineNum);

  public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
  }
  ```
- primary type field 비교 시, **compareTo 메서드에서 관계연산자 <와 >를 사용하는 방식은 오류를 유발하니, 이제는 추천하지 않는다.** 대신 `compareTo`를 이용하라.

# 클래스와 인터페이스

## 15. 클래스와 멤버의 접근 권한을 최소화하라

- 잘 구현된 컴포넌트
  - 클래스 내부 데이터 & 구현 정보를 잘 숨김
  - 구현과 API 깔끔히 분리 + API를 통해서만 소통

### 정보 은닉의 장점

- 시스템 개발속도 up - 여러 component 병렬로 개발 가능
- 시스템 관리 비용 down - 디버깅 속도 down, component 교체 부담 down
- 성능 최적화에 도움 - 원하는 component만 최적화 쉬움
- 소프트웨어 재사용성 up - dependency 낮은 component는 다른 환경에서도 사용 가능
- 큰 시스템 개발 난이도 down - 개별 component 동작 체크 가능

### Java가 제공하는 정보 은닉 장치

- 선언된 위치
- 접근 제한자
  - `private`: 선언한 top level class에서만 접근 가능
  - `package-private` (default): 멤버가 소속된 패키지 안의 모든 클래스에서 접근 가능
  - `protected`: 이 멤버를 선언한 클래스의 하위 클래스에서도 접근 가능
  - `public`: 모든 곳에서 접근 가능

### 정보 은닉의 기본 원칙

**모든 class와 member의 접근성을 최대한 좁혀야 한다.**

- top level class/interface에 부여할 수 있는 접근 수준: `package-private`(default) & `public`
  - `public`(+ `protected`): 공개 API, 하위 호환을 통해 영원히 관리해주어야 함
  - `package-private`(+ `private`): 내부 구현, 언제든 수정 가능, 패키지 외부에서 쓸 이유 없다면 이것으로 선언

### 접근 제어 설계 원칙

- 권한 부여 순서
  1. API 제외 모든 멤버는 `private`으로 선언
  1. 같은 package의 다른 class가 접근해야 하는 것들은 `private` 에서 `package-private`으로 풀어줌
  1. 권한을 풀어주는 일이 잦아진다면, component를 더 분리해야하는지 고민
- Serializable class에서는 `private`, `package-private`이 공개 API가 될 수 있음
- test를 위해 `private`을 `package-private`으로 풀어주는 정도까진 괜찮음
- interface 구현 시, 해당 interface에 선언된 모든 method는 `public`으로 선언해야 함

**`public` class 의 [instance field](Java.md#instance-variable)는 되도록 `public`이 아니어야 한다.**

- final까지도 아니면 해당 필드에 담을 수 있는 값을 제한할 힘을 잃게 됨
- `public` 가변 필드를 갖는 클래스는 일반적으로 [스레드 안전(thread safe)](../../Terms/Terms.md#thread-safe) 하지 않음
- `public` 불변 필드도 제거할 수 없어 리팩터링 불가
- 꼭 필요한 구성요소로써의 상수라면 `public static final`로 선언

**클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안 된다.**

- client에서 배열의 내용 수정 가능
- 방어
  - `private`로 선언 후, `public final` 리스트를 추가
  - `private`로 선언 후, 복사본 반환

## 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.**

### public class

- instance field만 사용한 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못함
- 필드를 모두 `private`으로 바꾸고 접근자 추가

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setX(double y) { this.y = y; }
}
```

**패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 내부 표현방식을 언제든 바꿀 수 있는 유연성 획득**

### `package-private` 클래스, private 중첩 클래스

**`package-private` 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.**

- 클래스가 표현하려는 추상 개념만 올바르게 표현
- 선언/사용 면에서 훨씬 깔끔

### 예외 사례

- `java.awt.package`의 `Point`, `Dimension` class
- public 클래스이지만 내부 필드 직접 노출
- 성능 문제 해결 불가 (item 67)

### 필드 불변

- 직접 노출할 때의 단점이 조금은 줄음
- 표현 변경 시 API를 변경해야 함
- 필드 읽을 때, 부수작업 수행 불가

## 17. 변경 가능성을 최소화하라

- 불변 클래스: 인스턴스 내부 값을 수정할 수 없는 클래스
  - 예시: `String`, `BigInteger`, `BigDecimal`

### 클래스를 불변으로 만들기 위한 규칙

**객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.**

**클래스를 확장할 수 없도록 한다.**

- 하위 클래스가 객체의 상태를 변하게 할 수 있음

**모든 필드를 final로 선언한다.**

- 시스템이 강제
- 새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건네도 문제없이 동작하게끔 보장

**모든 필드를 private으로 선언한다.**

- 클라이언트에서 직접 접근해서 수정하는 일을 막아줌
- `public final`로 선언해도 되지만, 다음 릴리즈에서 내부 표현을 바꾸지 못함

**자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.**

- 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 해당 객체의 참조를 얻을 수 없도록 해야함
- 생성자, 접근자, `readObject` 메서드(아이템 88)에서 방어적 복사 수행해야 함

### 예시

- 불변 복소수 클래스
  - 연산할 경우 새로운 인스턴스 만들어 반환
  - 메서드 이름으로 동사 `add` 등이 아닌 `plus` 등을 사용하여 객체 값이 변하지 않는다는 것을 강조

### 장점

**불변 객체는 단순하다.**

- 상태가 파괴될 때까지 간직
- 가변일 경우에는 믿고 사용하기 어려울 수 있음

**불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.**

- 동시에 사용해도 훼손 X
- 안심하고 공유 가능
- 최대한 재활용 권장

**불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.**

- 예시: `BigInteger`
  - 부호는 `int` 변수, 크기는 `int` 배열
  - 부호 반대인 객체 생성시, 크기 배열은 공유 가능

**객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.**

- Map의 key와 Set의 원소로 쓰기에 적합
- 안에 담긴 값이 변하지 않아 허물어질 일이 없음

**불변 객체는 그 자체로 [Failure atomicity(실패 원자성)](../../Terms/Terms.md#failure-atomicity)(아이템 76)을 제공한다.**

- 잠깐이라도 불일치 상태에 빠질 가능성 없음

### 단점

**값이 다르면 반드시 독립된 객체로 만들어야한다.**
- 대처방법 1. multi-step operation을 기본 기능으로 제공하는 방법
  - `package-private`으로 선언된 [mutable companion class(가변 동반 클래스)](../../Terms/Terms.md#mutable-companion-class) 이용
  - ex) `BigInteger`
- 대처방법 2. 복잡한 연산이 예측 불가능 할 때는 companioni class를 public으로 제공
  - ex) `String` -> `StringBuilder`

### Serialize 할 때
- 불변 클래스 내부에 가변 객체를 참조하는 필드가 있다면
- `readObject`, `readResolve`, `ObjectOutputStream.writeUnshared`, `ObjectInputStream.readUnshared` method 중 하나를 제공해야 함
- 그렇지 않으면 공격자가 이 클래스로부터 가변 인스턴스를 만들어낼 수 있음 (아이템 88)

### 원칙

**클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.**

- 특정 상황에서의 잠재적 성능 저하를 제외하고는 장점만 존재
- 어쩔 수 없다면 [mutable companion class](../../Terms/Terms.md#mutable-companion-class) 이용

**불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분은 최소한으로 줄이자.**
- 객체가 가질 수 있는 상태의 수를 줄이면, 예측이 쉬워지고, 오류 가능성 적어짐
**다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.**

**생성자는 불변식 설정이 모두 완료된, 초기확가 완벽히 끝난 상태의 객체를 생성해야 한다.**


## 18. 상속보다는 컴포지션을 활용하라
- 상속은 코드를 재사용하는 강력한 수단 but 항상 최선은 아님
- 다른 패키지의 구체 클래스를 상속하는 일은 위험

**메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.**

- 상위클래스의 구현에 따라 하위클래스의 동작에 이상이 생길 수 있음
- 상위클래스의 릴리스에 따른 내부 구현의 변화로 하위클래스에 문제 발생 가능
  - 상위 클래스의 새롭게 생긴 구현으로 인한 중복된 member변수 조작
  - 상위 클래스의 새로운 메서드 추가 + 상속 후 구현하지 못하여 허용되지 않은 원소 추가
  - 하위클래스에서 새롭게 정의한 메서드가 공교롭게 상위 클래스에 추가되는 경우

### 해결책

- 확장하는 대신, 새로운 클래스를 만들고, private 필드로 기존 클래스의 인스턴스를 참조하게 함
- composition: 기존 클래스가 새로운 클래스의 구성요소로 쓰임
- forwarding: 새 클래스의 메서드(forwarding method)는 기존 클래스의 대응하는 메서드를 호출
- 예시: 아래 클래스를 wrapping해서 사용

    ```java
    public class ForwardingSet<E> implements Set<E> {
        private final Set<E> s;
        public ForwardingSet(Set<E> s) { this.s = s; }

        public void clear() { s.clear(); }
        ...
    }

    public class InstrumentedSet<E> extends ForwardingSet<E> {  // Wrapper class
        private int addCount = 0;

        public InstrumentedSet(Set<E> s) {
            super(s);           
        }

        @Override public boolean add(E e) {
            addCount++;
            return super.add(e);
        }

        ...
    }
    ```

  - wrapper class: [callback framework](../../Terms/Terms.md#callback-framework)와 어울리지 않는다는 것 빼곤 단점 없음

- 상속은 반드시 하위 클래스가 상위 클래스의 하위 타입일 때만 쓰임
  - B is a A (`B extends A`)
  - 상속을 하기 전, 상위 클래스의 API에는 아무런 결함이 없거나 해당 결함이 하위 클래스로 전파되어도 괜찮은지 판단해야 함

## 19. 상속을 고려해 설계하고 문서화하라. 그렇지 않았다면 상속을 금지하라

### 원칙

#### 1. 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다

- 좋은 API 문서란 '어떻게'가 아닌 '무엇'을 하는지 설명하는 것을 위반
- 상속이 캡슐화를 해치기 때문에 발생하는 현상
- `@implSpec` tag를 통해 선택적으로 구현방식 설명

#### 2. 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다

- 효율적인 하위 클래스 제작을 위함
- `protected`로 노출할 메서드는 예측/테스트를 통해 정하는 것이 최선

#### 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 '유일'하다

- 하위 클래스를 작성할 때, 필요한 `protected` 멤버가 생김

#### 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야 한다

- `protected` method 및 field 구현 시, 선택한 결정에 영원히 책임져야 함

#### 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다

- 이 규칙을 어기면 프로그램 오동작
- 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행

#### `clone`과 `readObject` 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다

- `clone`: 복제본의 상태를 올바르게 수정하기 전, 재정의한 메서드부터 호출하게 됨, 오류 중 원본 객체 참조가 발생하면 원본 객체에도 피해
- `readObject`: 하위 클래스의 상태가 역직렬화되기 전, 재정의한 메서드부터 호출하게 됨

### 정리

- **클래스를 상속용으로 설계하려면 엄청난 노력이 들고 그 클래스에 안기는 제약도 상당함**
- **상속용으로 설계하지 않은 클래스는 상속을 금지**
  - 일반적인 구체 클래스는 상속용으로 설계 X
  - 문서화 X
  - 인터페이스가 잘 설계되어있다면 상속 없이도 개발 충분히 가능
  - Wrapper class Pattern을 통해 상속 대체 가능
  - 굳이 하겠다면 재정의 가능 메서드를 호출하는 자기 사용 코드를 완벽히 제거

## 20. 추상 클래스보다는 인터페이스를 우선하라

### Java가 제공하는 다중 구현 메커니즘

- Interface
  - java 8 부터 [default method](./Java.md#default-method) 제공 가능
  - 어떤 class를 상속했던, inteface 규약을 지켰으면 같은 타입으로 취급
- Abstract class
  - 구현하는 class는 반드시 abstract class의 sub-class 가 되어야 함

### Interface의 장점

1. **기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있음**
  - 아직 없다면, class 선언에 `implements` 구문만 추가해주면 됨
2. **[mixin](../../Terms/Terms.md#mixin) 정의에 안성맞춤**
  - '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과
  - 추상 클래스는 기존 class에 덧씌울 수 없음
3. **interface로는 계층구조가 없는 type framework를 만들어낼 수 있음**
  - 두 개 이상의 interface로 새로운 interface를 만들 수 있음
  - 예시: singer songwriter
    ```java
    public interface SingerSongwriter extends Singer, Songwriter {}
    ```
  - 추상 클래스 사용 시, 모든 경우의 수를 고려하여 상속해야 함
    - n개 class를 이용해 여러 `type` 제작 시, `n^2`개의 class를 생성해야 함
4. Wrapper class 관용구와 함께 사용하면 **interface는 기능을 향상시키고 안전하고 강력한 수단이 된다.**
  - 추상 클래스로 정의 시, 해당 type에 기능을 추가하는 방법은 상속 뿐
5. 구현 방법이 명백한 것이 있다면, [default method](./Java.md#default-method) 이용하면 됨
  - `@implSpec` javadoc tag를 통해 문서화해야 함
  - `equals`, `hashCode` 등의 method는 default method로 제공해서는 안 됨 (instance field 가질 수 없고, public이 아닌 static member도 가질 수 없음)
  - 다른 사람이 만든 interface에는 default method 추가 불가

### [template method pattern](../../Terms/Terms.md#Template-method-pattern)
- interface와 abstract skeletal implementation class(추상 골격 클래스)를 조합하면 모두의 장점 취할 수 있음
  - interface로 type 및 default method 정의/제공
  - abstract skeletal implementation class: 나머지 method 구현
  - interface 이름이 `A`라면, 골격 구현 클래스 이름은 `AbstractA`라고 지음 (관례)
- 예시
  ```java
  public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
    // 이 안에서 interface의 method 구현
  }
  ```

### 구현

- 방법 1. [default method](./Java.md#default-method) 사용
- 방법 2. abstract skeletal implementation class (추상 골격 구현 클래스) 사용
- 방법 3. Simple implementation(단순 구현) 사용
  - **Simple implementation(단순 구현)은 골격 구현의 작은 변종**
  - 상속을 위해 interface를 구현한 것이지만, abstract class가 아님
  - 동작하는 가장 단순한 구현

## 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

- Java 8 이전에는 interface에 method 추가 시, 구현 class에 우연히 해당 method가 있지 않는 한 compile error 발생
- Java 8 이후부터 [default method](./Java.md#default-method)를 통해 해결할 수 있음
  - 매끄럽게 연동된다는 보장은 없음
  - interface에 추가되는 method는 구현 클래스에 대해 아는 바가 없음
- lambda 식을 위해 핵심 collection interface들에 [default method](./Java.md#default-method) 추가
  - **생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법이다.**
- **[default method](./Java.md#default-method)는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있다.**
  - 기존 구현체와 충돌하는 경우가 흔하지 않게 존재
- 기존 method 제거 혹은 signature를 수정하는 용도가 아님

#### 핵심: [default method](./Java.md#default-method)가 생겼더라도 interface를 설계할 때는 여전히 세심한 주의 필요

- 반드시 테스트 필요
  - 서로 다른 방식으로 최소한 세 가지는 구현해봐야 함
  - interface의 instance를 활용하는 클라이언트도 구현해봐야 함
  - **intreface를 release한 후라도 결함을 수정하는 게 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안 된다.**

## 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

### interface란
- type 역할
- interface 구현 == 자신의 instance로 무엇을 할 수 있는지 이야기하는 것
- 지침에 맞지 않는 예: 상수 인터페이스 (static final 필드로만 가득 차있음)
  - **상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예다.**
  - 상수는 내부 구현인데, interface는 그것을 API로 노출해버리는 것
  - 사용자에게 혼란
  - client code가 이 상수에 종속되게 할 수도 있음

### 상수를 공개하고 싶다면?
- class나 interface 자체에 추가하는 방법
  - 특정 class나 interface와 강하게 연관되었을 경우
  - ex) `Integer.MAX_VALUE`
- Enum Type(열거 타입)으로 만들어 공개하는 방법
  - 열거 타입으로 나타내기 적합할 경우
- instance화 할 수 없는 util class에 넣어 공개하는 방법


## 23. 태그달린 클래스보다는 클래스 계층구조를 활용하라

### 태그달린 클래스

- 두 개 이상의 의미를 표현할 수 있고, 그중 현재 표현하는 의미를 태그값으로 알려주는 class
- **장황하고, 오류를 내기 쉽고, 비효율적**
- **태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류**
  - subtyping으로 상속하면 됨

## 24. 멤버 클래스는 되도록 static으로 만들라

### nested class (중첩 클래스)

- 다른 클래스 안에 정의된 클래스
- 자신을 감싼 class 안에서만 사용

#### 종류: static member class를 제외한 나머지는 내부 class

- static member class
  - 다른 class 안에 선언
  - outer class의 private member에도 접근할 수 있음
  - 다른 정적 member와 똑같은 접근 규칙을 적용받음
  - outer class와 함께 쓰일 때만 유용한 `public` 도우미 class로 쓰임
- (non-static) member class
  - member class의 instance는 암묵적으로 outer class의 instance와 연결 (숨은 외부참조 - 시간/공간 소비)
  - 정규화된 `this` (`OuterClassName.this`)를 이용해 outer instance의 method 및 참조를 가져올 수 있음
  - 생성
    - 보통은 member class의 생성자 호출할 때 만들어짐
    - 드물게 `OuterClassName.new MemberClass(args)`를 호출해 수동으로 생성
  - Adapter를 정의할 때 자주 쓰임: 어떤 class의 instance를 감싸 다른 class의 instance처럼 보이게 하는 뷰로 사용
  - 자신의 collection view를 구현할 때 주로 사용
- anonymous class
  - 쓰이는 시점에 선언과 동시에 instance화
  - 코드의 어디서든 만들 수 있음
  - non-static한 context에서 사용될 때만 outer class의 instance 참조 가능
  - lambda 등장 전에 작은 함수 객체나 처리 객체를 만드는 데 주로 사용
  - 정적 팩터리 메서드 구현할 때 쓰임
- local class
  - 가장 드물게 사용
  - 지역 변수 선언할 수 있는 어디서든 선언 가능
  - 이름이 있고 반복 사용 가능
  - non-static context에서만 outer instance 참조 가능

### static member class로 만들어야하는 이유

- **member class에서 바깥 instance에 접근할 일이 없다면 무조건 static을 붙여서 static member class로 만들자.**
- static이 아니면 숨은 외부참조 발생
  - 시간/공간 소비
  - GC가 outer class를 수거하지 못하는 memory leak 발생 가능

## 25. 톱레벨 클래스는 한 파일에 하나만 담아라

### 여러 top-level class 선언할 경우

- compiler상 제약은 없음
- 한 class를 여러 가지로 정의할 수 있게 됨
- compile 순서에 따라 오류 혹은 결과가 달라지는 상황 발생
- 굳이 사용하고싶다면 static member class를 고려하라

# 제네릭

## 26. raw type은 사용하지 말라

## 27. 비검사 경고를 제거하라