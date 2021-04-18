# [CHAP.03] 람다 표현식
- **익명 클래스**처럼 이름이 없는 함수면서,
  - **메서드**를 **인자**로 전달할 수 있으므로,
  - 일단은 **람다 표현식**이 **익명 클래스**와 유사하다고 생각

## 3.1. 람다란 무엇인가
### 람다 표현식
- 메서드로 전달 할 수 있는 **익명 함수**를 단순화 한 것
- 이름은 없으나, **파라미터 리스트**, **바디**, **변환 형식**, 발생할 수 있는 **예외 리스트**를 가질 수 있음

### 람다의 특징
#### 익명
- 보통의 메서드와 달리, 이름이 없음
#### 함수
- 메서드처럼 특정 클래스에 종속되지 않음
- 하지만, 메서드처럼 `파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트`를 반환
#### 전달
- 람다 표현식을 **메서드 인수**로 전달하거나 **변수**로 저장할 수 있음
#### 간결성
- **익명 클래스**처럼 많은 자질구레한 코드를 구현할 필요 없음

### 람다의 특징
- 람다 **미적분학** 학계에서 개발한 시스템에서 유래
- **2장**에서 사용한 **동적 파라미터 형식**의 코드를 더 쉽게 구현 가능
- 기존 코드와 람다 비교
  ```java
  Comparator<Apple> beWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
      return a1.getWeight().compareTo(a2.getWeight());
    }
  };

  // 람다를 사용한 코드
  Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a2.getWeight().compareTo(a2.getWeight());
  ```

### 람다의 구성
- 세부분으로 이루어짐
```java
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

// (Apple a1, Appl2 a2) : 람다 파라미터, `Compator::compare` 메서드 파라미터
// -> : 화살표, 람다의 파라미터 리스트와 바디 구분
// a2.getWeight().compareTo(a2.getWeight()) : 람다 바디, 람다의 반환값에 해당하는 표현식
```
- 람다의 표현법
  ```java
  // expression style
  (parameters) -> expression

  // block style
  (parameters) -> { statements; }
  ```

## 3.2. 어디에, 어떻게 람다를 사용할까?
- 함수형 인터페이스라는 문맥에서 **람다 표현식**을 사용할 수 있음

### 3.2.1. 함수형 인터페이스
- `Predicate<T>`
  ```java
  public interface Predicate<T> {
    boolean test(T t);
  }
  ```
- 함수형 인터페이스
  - 정확히 **하나의 추상 메서드**를 지정하는 **인터페이스**
- 예시
  ```java
  public interface Comparattor<T> {
    int compare(T o1, T o2);
  }

  public interface Runnable {
    void run();
  }

  public interface ActionListener extends EventListener {
    void actionPerformed(ActionEvent e);
  }

  public interface Callable<V> {
    V call() throws Exception;
  }

  public interface PrivilegedAction<T> {
    T run();
  }
  ```
- `defatlt method`를 포함하더라도, 추상 메서드가 오직 하나면 **함수형 인터페이스**
- 람다 표현식으로 **함수형 인터페이스**의 **추상 메서드 구현**을 직접 전달할 수 있으므로
- **전체 표현식**을 **함수형 인터페이스의 인스턴스**로 취급할 수 있음
  - `= 함수형 인터페이스를 구현한 클래스의 인스턴스`
- `Runnable` 예시
  ```java
  // lambda 
  Runnable r1 = () -> System.out.println("Hello World 1");

  public static void process(Runnable r) {
    r.run();
  }

  process(r1); // Hello World 1
  process(() -> System.out.println("Hello World 3")); // 직접 전달된 lambda 표현식으로 Hello World 3 출력
  ```

### 3.2.2. 함수 디스크립터
- 함수형 인터페이스의 **추상 메서드 시그니처**는
  - **람다 표현식의 시그니처**를 의미
- 람다 표현식의 **시그니처**를 서술하는 메서드를
  - **함수 디스크립터**(function descriptor)라고 부름
- 예를 들어 `Runnable` interface의 유일한 추상 메서드 `run`은
  - 인수와 반환값이 없으므로(`void`)
  - `Runnable` 인터페이스는 **인수**와 **반환값**이 없는 시그니처로 생각할 수 있음
- `() -> void` 표기
  - 파라미터 리스트가 없으며, `void`를 반환하는 함수
- `(Apple, Apple) -> int`
  - 두개의 `Apple`을 받아, `int`를 반환하는 함수
- 한 개의 `void`메서드 호출은 중괄호로 감쌀 필요 없음
  ```java
  process(() -> System.out.println("This is awesome"));
  ```
- `@FunctionalInterface`
  - 함수형 인터페이스를 가리키는 어노테이션
  - 해당 어노테이션을 선언했지만, 실제 함수형 인터페이스가 아닐경우, **컴파일 에러** 발생

## 3.3. 람다 활용 : 실행 어라운드 패턴
- 자원 처리에 사용하는 **순환 패턴**(recurrent pattern)
  - 자원을 열고, 처리한 이후, 자원을 닫는 순서로 이루어짐
- `setup`과 `cleanup`과정은 대부분 비슷
- 예시 : `try-with-resource`
  ```java
  public String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
      return br.readLine();
    }
  }
  ```

### 3.3.1. 1단계 : 동작 파리미터화를 기억하라
- 위 예시 코드는 `한번에 한줄만 읽을 수 있음`
- `한 번에 두줄` 혹은 `가장 자주 사용되는 단어 반환`등을 하려면?
- 기존의 설정, 정리 과정은 재사용하고, `processFile`만 다른 동작을 수행하도록 해야 하므로
  - **동작 파라미터화**가 필요함
- `BufferedReader`를 이용해서 다른 동작을 수행할 수 있도록
  - `processFile` 메서드로 동작을 전달해야 함
- `processFile` 메서드가 한번에 두행을 읽게 하려면
  - 우선 `BufferedReader`를 인수로 받아 `String`을 반환하는 람다가 필요함
  - 예시
    ```java
    String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
    ```

### 3.3.2. 2단계 : 함수형 인터페이스를 이용해서 동작 전달
- **함수형 인터페이스** 자리에 **람다**를 사용할 수 있음
- `BufferedReader -> String`과 `IOException`을 throw할 수 있는 시그니처와 일치하는 **함수형 인터페이스** 구성 필요
- 이를 `BufferedReaderProcessor`로 정의
  ```java
  @FunctionalInterface
  public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
  }

  // 이 인터페이스를 사용하여, processFile 메서드의 인수로 전달 가능
  public String processFile(BufferedReaderProcessor p) throws IOException {
    ...
  }
  ```

### 3.3.3. 3단계 : 동작 실행
- `BufferedReaderProcessor`에 정의된 `process` 메서드의 시그니처(`BufferedReader -> String`)와 일치하는 람다 전달 가능
- `processFile` 바디 내에서 `BufferedReaderProcessor::process`를 호출할 수 있음
- 코드
  ```java
  public String processFile(BufferedReaderProcessor p) throws IOException {
    try ( BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
      return p.process(br); // BufferedReader 객체 처리
    }
  }
  ```

### 3.3.4. 4단계 람다 전달
- 람다를 이용하여 `processFile`의 메서드로 전달 가능
- 코드
  ```java
  // 한 행 처리 코드
  String oneLine = processFile((BufferedReader br) -> br.readLine());
  // 두 행 처리 코드
  String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
  ```

## 3.4. 함수형 인터페이스 사용
- **함수형 인터페이스**는 **오직 하나의 추상 메서드**
- 함수형 인터페이스의 추상 메서드는
  - **람다 표현식**의 **시그니쳐**를 묘사
- 함수형 인터페이스의 **추상 메서드 시그니처**를 **function descriptor**라고 함
- 다양한 람다 표현식을 사용하려면
  - 공통의 함수 디스크립터를 기술하는 **함수형 인터페이스 집합**이 필요함
- 이미 `java`에서는 `Comparable, Runnable, Callable` 등의 다양한 함수형 인터페이스를 포함
- `java.util.function` 패키지로 여러 가지 새로운 **함수형 인터페이스** 제공

### 3.4.1. Predicate
- `java.util.function.Predicate<T>` 인터페이스
- `test`라는 추상 메서드르 정의
- `T`의 객체를 인수로 받아 `boolean`을 반환
- `T`형식의 객체를 사용하는 `boolean`표현식이 필요한 상황에서 `Predicate` 인터페이스 사용 가능
- 예제
  ```java
  @FunctionalInterface
  public interface Predicate<T> { 
    boolean test(T t);
  }
  
  public <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> results = new ArrayList<>();
    for(T t : list) {
      if(p.test(t)) {
        results.add(t);
      }
    }
    return results;
  }
  Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
  List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
  ```

### 3.4.2. Consumer
- `java.util.function.Consumer<T>`
- `T`객체를 받아 `void`를 반환하는 `accept` 추상 메서드 정의
- `T`형식의 객체를 **인수**로 받아 어떤 동작을 **수행**하고 싶을 때 사용
- 예를 들어
  - `Integer` 리스트를 인수로 받아, 각 항목에 어떤 동작을 수행하는 `forEach` 메서드를 정의
- 예시
  ```java
  @FunctionalInterface
  public interface Consumer<T> {
    void accept(T t);
  }

  public <T> void forEach(List<T> list, consumer<T> c) {
    for(T t : list) {
      c.accept(t);
    }
  }
  forEach(
      Arrays.asList(1,2,3,4,5),
      (Integer i) -> System.out.println(i) // Consumer의 accept 메서드를 구현하는 람다
  );
  ```

### 3.4.3. Function
- `java.util.function.Function<T, R>`
- `T`를 인수로 받아, `R` 객체를 반환하는 추상 메서드 `apply`를 정의
- 입력을 **출력으로 매핑**하는 람다를 정의할 때 사용
- 예시
  - `String`의 길이를 포함하는 `Integer`리스트로 변환하는 `map` 메서드를 정의하는 예제
  ```java
  @FunctionalInterface
  public interface Function<T, R> {
    R apply(T t);
  }

  public <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for(T t : list) {
      result.add(f.apply(t));
    }
    return result;
  }
  
  // [7, 2, 6]
  List<Integer> l = map(
    Arrays.asList("lambdas", "in", "action"),
    (String s) -> s.length() // Function의 apply를 구현하는 람다
  );
  ```

### 기본형 특화
- `Predicate<T>, Consumer<T>, Function<T,R>`외에 특화된 형식의 함수형 인터페이스
- 자바의 모든 형식은
  - 참조형(`reference type`, `Byte, Integer, Object, List`)
  - 기본형(`primitive type`, `int, double, type, char`)
- 하지만 **제네릭 파라미터**에는 **참조형**만 사용가능
  - 제네릭의 내부 구현때문에 어쩔 수 없음
- 자바에서는 `기본형 -> 참조형`으로 변환하는 기본 제공 -> `Boxing`
- `Autoboxing`
  ```java
  List<Integer> list = new ArrayList<>();
  for (int i = 300 ; i < 400; i++) {
    list.add(i);
  }
  ```
  - 위와 같은 변환이 가능하나, 비용이 소모됨
    - 박싱한 값은, **기본형을 감싸는 래퍼**며, **힙에 저장**
    - 박싱한 값은 **메모리를 더 소비**, 가져올 때도 메모리를 탐색하는 과정 필요
- `java 8`에서는 **기본형**을 입출력으로 사용하는 상황에서
  - 오토박싱을 피할 수 있도록, 특수한 함수형 인터페이스 제공
  - `IntPredicate`는 값을 파싱하지 않지만, `Predicate<Integer>`는 `1000`이라는 `Integer` 객체로 박싱
  ```java
  public interfcae IntPredicate {
    boolean test(int t);
  }

  IntPredicate evenNumbers = (int i) -> i % 2 == 0;
  eventNumbers.test(1000); // 박싱 없이 'true` 반환

  Predicate<Integer> oddNumbers = (Integer i) -> i % 2 != 0;
  ```
- 특정 형식을 입력으로 받는 함수형 인터페이스 앞에는 **형식명**이 붙음
  - `DoublePredicate, IntConsumer, LongBinaryOperator, IntFunction`
- `Function` 인터페이스는 `ToIntFunction<T>, IntToDoubleFunction`등의 다양한 **출력 파라미터** 제공
  

### 람다와 함수형 인터페이스 예제

**사용 사례**|**람다 예제**|**대응하는 함수형 인터페이스**
-----|-----|-----
boolean|(List<String> list) -> list.isEmpty()|Predicate<List<String>>
객체 생성|() -> new Apple(10)|Supplier<Apple>
객체 소비|(Apple a) -> System.out.println(a.getWeight())|Consumer<Apple>
객체 선택/추출|(String s) -> s.length()|Function<String, Integer> <br/> ToIntFunction<String>
두 값 조합|(int a, int b) -> a + b|IntBinaryOperator
두 객체 비교|(Apple a1, Apple a2) -> a1.getWeight().compreTo(a2.getWeight())|Comparator<Apple><br /> BiFunction<Apple, Apple, Integer> <br /> ToIntBiFunction<Apple, Apple>

### 예외, 람다, 함수형 인터페이스의 관계
- 함수형 인터페이스는
  - 확인된 **예외**를 던지는 동작을 허용하지 않음
- 예외를 던지는 **람다 표현식**을 만들려면,
  - 확인된 예외를 선언하는 **함수형 인터페이스 직접 정의**하거나
  - 람다를 `try/catch`블록으로 감싸야 함 
- 예시
  ```java
  @FunctionalInterface
  public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException; // IOException 명시적 정의
  }

  // 확인된 예외를 try, catch로 감싸기
  Function<BufferedReader, String> f = (BufferedReader b) -> {
    try {
      return b.readLine();
    } catch(IOException e) {
      throw new RuntimeException(e);
    }
  };
  ```
## 3.5. 형식 검사, 형식 추론, 제약

### 3.5.1. 형식 검사
- 람다가 사용하는 context를 이용하여, 람다의 `type` 추론이 가능함
  - context : 람다가 전달될 **메서드 파라미터**나 **람다가 할당되는 변수** 등
- context에서 기대되는 **람다 표현식**의 형식을 **대상 형식**(target type)이라고 함
- 예시
  ```java
  List<Apple> heavierThan150g = 
    filter(inventory, (Apple apple) -> apple.getWeight() > 150);
  ```
  - 순서
    - `filter`메서드의 선언 확인
    - `filter` 메서드는 두번째 파라미터로 `Predicate<Apple>`의 형식(대상 형식) 기대
    - `Predicate<Apple>`은 `test`라는 **한 개의 추상 메서드**를 정의하는 **함수형 인터페이스
    - `test`메서드는 `Apple`을 받아 `boolean`을 반환하는 **함수 디스크립터**를 묘사
    - `filter`메서드로 전달된 인수는 이와 같은 요구사항 만족
  - 위 예제에서, `Apple`을 인수로 받아 `boolean`을 반환하므로 유효한 코드
  - 람다 표현식이 예외를 던질 수 있다면
    - **추상 메서드**도 같은 예외를 던질 수 있도록 `throws`로 선언해야 함

### 3.5.2. 같은 람다, 다른 함수형 인터페이스
- `target type`이라는 특징 때문에
  - 같은 람다 표현식이더라도, 호환되는 **추상 메서드**를 가진 **다른 함수형 인터페이스**로 사용될 수 있음
- `Callable`, `PrivilegedAction` 인터페이스는 인수를 받지 않고 `T`를 반환하는 함수 정의
  ```java
  Callable<Integer> c = () -> 42; // Callable<Integer>
  PrivilegedAction<Integer> p = () -> 42; // PrivilegedAction<Integer>

  Comparator<Apple> c1 = 
    (Apple a1, Apple a2) -> a.getWeight().compareTo(a2.getWeight());
  ToIntBiFunction<Apple, Apple> c2 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
  ```

#### 다이아몬드 연삱다
- `<>`를 사용하여 제네릭 형식을 추론할 수 있음
  ```java
  List<String> listOfStrings = new ArrayList<>();
  ```

#### 특별한 void 호환 규칙
- 람다의 바디에 **일반 표현식**이 있으면 `void`를 반환하는 함수 디스크립터와 호환
- `List::add` 메서드는 `Consumer` 콘텍스트(`T -> void`)가 기대하는 `void` 대신 `boolean`을 반환하지만 유효환 코드
  ```java
  // Predicate는 boolean 반환 값
  Predicate<String> p = s -> list.add(s);
  // Consumer는 void 반환 값
  Consumer<String> b = s -> list.add(s);
  ```