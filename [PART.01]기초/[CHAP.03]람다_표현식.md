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