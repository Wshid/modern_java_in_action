# [CHAP.09] 리팩터링, 테스팅, 디버깅
- 기존 코드를 이용해서, 새로운 프로젝트를 시작하는 상황
- **람다 표현식**을 이용하여 **가독성**과 **유연성**을 높이려면
  - 기존 코드를 어떻게 리팩터링 해야하는가?
- 람다 표현식으로
  - `strategy`, `template method`, `observer`, `chain of responsibility`, `factory` 등의
    - 객체지향 디자인 패턴을 어떻게 간소화 할지 확인
- 람다 표현식와 스트림 API를 사용하는 코드를
  - 테스트하고, 디버깅 하는 모습 설명

## 9.1. 가독성과 유연성을 개선하는 리팩터링
- 인수로 전달하려는 메서드가 이미 있을 때는,
  - 메서드 참조를 통해 람다보다 간결한 코드 작성 가능
- 람다 표현식은 **동적 파라미터화 형식**을 지원
  - 코드의 다양한 요구사항 변화에 대응할 수 있도록 **동작을 파라미터화**

### 9.1.1. 코드 가독성 개선
- 코드 가독성 이란?
  - 어떤 코드를 다른 사람도 쉽게 이해할 수 있음
- 코드 가독성 개선
  - 우리가 구현한 코드를 다른 사람이 쉽게 이해 및 유지보수 가능
- `코드의 문서화`를 잘하고, `코딩 규칙 준수`
- 세가지 방법
  - `익명 클래스`를 `람다 표현식`으로 리팩터링
  - `람다 표현식`을 `메서드 참조`로 리팩터링
  - `명령형 데이터 처리`를 `스트림`으로 리팩터링

### 9.1.2. 익명 클래스를 람다 표현식으로 리팩터링
- 익명 클래스가
  - 코드를 장황하게 만들며
  - 쉽게 에러를 발생시킬 수 있음
- `Runnable` 객체 생성 비교
  ```java
  // 익명 클래스 코드
  Runnable r1 = new Runnable() {
    public void run() {
      System.out.println("Hello");
    }
  };
  // 람다 표현식으로 개선
  Runnable r2 = () -> System.out.println("Hello");
  ```
- 물론, 모든 익명 클래스를, 람다 표현식으로 변환할 수 있는 것은 아님
- 조건
  - 익명 클래스에서 사용한 `this`와 `super`는 **람다 표현식**에서 다른 의미를 가짐
    - 익명 클래스의 `this`; 익명 클래스 자신을 가리킴
    - 람다의 `this`; 람다를 감싸는 클래스
  - 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있음 -> `shadow variable`
  - 람다 표현식으로는 변수를 가릴 수 없음(아래 코드는 컴파일 불가)
    ```java
    int a = 10;
    Runnable r1 = () -> {
      int a = 2; // compile error
      System.out.println(a);
    }

    Runnable r2 = new Runnable() {
      public void run() {
        int a = 2; // 모든 것이 잘 작동한다
        System.out.println(a);
      }
    }
    ```
- 익명 클래스를 **람다 표현식**으로 바꾸면
  - `Context Overloading`에 따른 모호함이 초래될 수 있음
- 인스턴스화 할 때 명시적으로 **형식**이 정해지는 반면,
  - 람다의 형식은 콘텍스트에 따라 달라지기 때문
- `Task`라는 `Runnable` 같은 시그니처를 갖는 **함수적 인터페이스**를 선언
  ```java
  interface Task {
    public void execute();
  }
  public static void doSomething(Runnable r) { r. run(); }
  public static void doSomething(Task a) { r.execute(); }

  // Task를 구현하는 익명 클래스 전달
  doSomething(new Task() {
    public void execute() {
      System.out.println("Danger danger!!");
    }
  })
  ```
- **익명 클래스**를 **람다 표현식**으로 바꾸면
  - 메서드를 호출할 때 `Runnable`과 `Task` 모두 대상 형식이 될 수 있으므로 문제 발생
    ```java
    // 모호함 발생
    doSomething(() -> System.out.println("Danger danger!!"));

    // 명시적 형변환을 이용하여 모호함 해결
    doSomething((Task)() -> System.out.println("Danger danger!!"));
    ```
- 위와 같은 `Netbeans`나 `Intellij` 등을 포함한 대부분의 `IDE`에서 제공하는
  - 리팩터링 기능을 이용하면, 위 문제가 해결됨

### 9.1.3. 람다 표현식을 메서드 참조로 리팩터링 하기
- **람다 표현식** 대신 **메서드 참조**를 사용하면 **가독성**을 높일 수 있음
  - 메서드 참조의 **메서드명**으로 코드의 참조 의도를 알릴 수 있음
- 예시 코드
  ```java
  Map<CaloricLevel, List<Dish>> dishesByCaloricLevel =
      menu.stream().collect(groupingBy(Dish::getCaloricLevel)); // 람다 표현식을 메서드로 추출

  public class Dish {
    public CaloricLevel getCaloricLevel() {
      if (this.getCalories() <= 400) return CaloricLevel.DIET;
      else if (this.getCalories() <= 700) return CaloricLevel.NORMAL;
      else return CaloricLevel.FAT;
    }
  }
  ```
- `comparing`과 `maxBy` 같은 **정적 핼포 메서드**를 활용해도 좋음
  - 이들은 **메서드 참조**와 조화를 이루도록 설계됨
  ```java
  inventory.sort(
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())); // 비교 구현에 신경써야 함
  inventory.sort(comparing(Apple::getWeight)); // 코드가 문제 자체 설명
  ```
- `sum, maximum`등 자주 사용하는 **리듀싱 연산**은
  - 메서드 참조와 함께 사용할 수 있는 **내장 헬퍼 메서드**를 제공
- 최댓값이나, 합계를 사용할 때 **람다 표현식**과 **저수준 리듀싱 연산**을 조합하는 것보다
  - `Collectors API`를 사용하면 코드의 의도가 명확해짐
  ```java
  int totalCalories =
    menu.stream().map(Dish::getCalories)
                  .reduce(0, (c1, c2) -> c1 + c2);
  
  // 내장 컬렉터 사용, 코드 자체로 명확히 설명 가능
  // summingInt 사용(자신이 어떤 동작을 수행하는지 메서드 이름으로 설명 가능)
  int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
  ```

### 9.1.4. 명령형 데이터 처리를 스트림으로 리팩터링하기
- 이론적으로는 `반복자`를 이용한 모든 컬렉션 처리 코드를
  - **스트림 API**로 변경하여야 함
    - 데이터 처리 파이프라인의 의도를 더 명확히 보여주기 때문
- 스트림은
  - `short-circuit`, `lazyness`와 `multicore`에서 사용할 수 있는 특징을 제공
- 두가지 패턴(`필터링`과 `추출`)를 혼합한 코드
  - 전체 구현을 살펴야, 의도 파악 가능
  - 또한 코드를 병렬시키기에 매우 어려움
    ```java
    List<String> dishNames = new ArrayList<>();
    for(Dish dish: menu) {
      if(dish.getCalories() > 300) {
        dishNames.add(dish.getName());
      }
    }
    
    // Stream API로 작성, 직접적으로 기술 및 병렬화 가능
    map.parallelStream()
        .filter(d -> d.getCalories() > 300)
        .map(Dish::getName)
        .collect(toList());
    ```
- 명령형 코드의 `break`, `continue`, `return` 등의 **제어 흐름문**을 모두 분석해서
  - 같은 기능을 수행하는 스트림 연산으로 유추해야 하므로
  - **명령형 코드 -> 스트림 API** 작업은, 쉬운일이 아님
- 이 작업을 가능하도록 하는 몇가지 도구 등이 존재
  - `LambdaFicator` 등

### 9.1.5. 코드 유연성 개선
- 람다 표현식을 이용하면 **동작 파라미터화**(behavior parameterization)을 쉽게 구현할 수 있음
- 다양한 람다를 전달하여 **다양한 동작**을 표현할 수 잇음
- 따라서 **변화하는 요구사항**에 대응할 수 있는 코드 구현 가능

#### 함수형 인터페이스 적용
- 람다 표현식을 이용하려면 **함수형 인터페이스**가 필요
- **조건부 연기 실행**(conditional deferred execution)과 **실행 어라운드**(execute around)
  - 두 가지 패턴으로 람다 표현식 리팩터링

#### 조건부 연기실행
- 실제 작업 코드 내부에 **제어 흐름문**이 복잡하게 얽힌 코드를 흔히 볼 수 있음
- 예시 : 내장 자바 Logger 클래스 예시
  ```java
  if (logger.isLoggable(Log.FINER)) {
    logger.finer("Problem: " + generateDiagnostic());
  }
  ```
- 위 코드의 문제
  - `logger`의 상태가 `isLoggable`이라는 메서드에 의해 클라이언트 코드로 노출
  - 메세지를 로깅할 때마다 `logger`의 객체의 상태를 매번 확인해야할까?
    - 오히려 코드 복잡성 증가
- 해결책: 작업 메세지를 로깅하기 전에, `logger`객체가 적절한 수준으로 설정되었는지
  - 내부적으로 확인하는 `log` 메서드를 활용
    ```java
    logger.log(Level.FINER, "Problem: " + generateDiagnostic());
    ```
- 불필요한 `if`문 제거 가능
  - `logger`의 상태 노출 필요 없음
  - 단, 위 코드에는
    - 인수로 전달된 **메세지 수준**에서 `logger`가 활성화되어 있지 않더라도
    - 항상 로깅 메세지를 평가해야 함
- 람다를 이용하면 쉽게 문제 해결 가능
  - **특정 조건** 에서만 메세지가 생성될 수 있도록
  - **메세지 생성과정**을 연기(defer)할 수 있어야 함
- 이 문제를 해결하도록 `Supplier`를 인수로 갖는, **오버 로드된 log 메서드** 활용
  ```java
  public void log(Level level, Supplier<String> msgSupplier) {
    if(logger.isLoggable(level)) {
      log(level, msgSupplier.get()); // 람다 실행
    }
  }
  ```
- 위 코드를 통해 문제 해결 가능
  - C 코드에서 **객체 상태**를 자주 확인하거나(`e.g. logger의 상태`)
  - 객체의 일부 메서드를 호출하는 상황(`e.g. 메세지 로깅`)
- 내부적으로 객체 상태를 확인한 다음에 메서드를 호출하도록 새로운 메서드를 구현하는 것이 좋음
  - **코드 가독성** 및 **캡슐화**(객체 상태가 C로 노출되지 않음)

#### 실행 어라운드
- 매번 같은 준비, 종료 과정이 있다면
  - 이를 **람다**로 표현
- `준비, 종료`과정을 처리하는 로직을 **재사용**함으로써
  - **코드 중복**을 줄일 수 있음
- 예시 코드
  ```java
  String oneLine = processFile((BufferedReader b) -> b.readline()); // 람다 전달
  String twoLines = processFile((BufferedReader b) -> b.readLine() + b.readLine()); // 다른 람다 전달
  
  public static String processFile(BufferedReaderProcessor p) throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader("ModernJavaInAction/chap9/data.txt"))) {
      return p.process(br); // 인수로 전달된 BufferedReaderProcessor를 실행
    }
  } // IOException을 던질 수 있는 람다의 함수형 인터페이스

  public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
  }
  ```
- 람다로 `BufferedReader` 객체의 동작을 결정할 수 있는 것은
  - **함수형 인터페이스** `BufferedReaderProcessor` 덕분

#### 결론
- `조건부 연기실행`과 `실행 어라운드`를 통해
  - 기존 코드의 **가독성**과 **유연성**을 개선하는 다양한 기법 확인
