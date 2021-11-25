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

## 9.2. 람다로 객체지향 디자인 패턴 리팩터링 하기
- 다양한 패턴을 유형별로 정리한 것이 **디자인 패턴**
  - 공통적인 소프트웨어 문제를 설계할 때 재사용 가능한, 검증된 청사진 제공

### 9.2.1. 전략
- 전략 패턴은
  - `한 유형의 알고리즘`을 보유한 상태에서
  - `Runtime`에 적절한 알고리즘을 선택하는 기법
- 다양한 기준은 갖는 **입력값**을 검증하거나
  - 다양한 **파싱 방법**을 사용하거나
  - **입력 형식**을 설정하는 등
  - 다양한 시나리오에서 사용 가능
- 전략 패턴의 구성
  - 알고리즘을 나타내는 Interface(`Strategy Interface`)
  - 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현(`ConcreteStrategyA`, `ConcreteStrategyB`)
  - 전략 객체를 사용하는 한 개 이상의 `클라이언트`

#### 예시: 오직 소문자 또는 숫자로 이루어져야하는 등, 텍스트 입력이 다양한 조건에 맞게 포맷되어 있는지 검증
- `String` 문자열을 검증하는 인터페이스
  ```java
  public interface ValidationStrategy {
    boolean execute(String s);
  }
  ```
- 위에서 정의한 인터페이스를 구현하는 `클래스`를 하나 이상 정의
  ```java
  public class IsAllLowerCase implements ValidationStrategy {
    public boolean execute(String s) {
      return s.matches("[a-z]+");
    }
  }

  public class IsNumeric implements ValidationStrategy {
    public boolean execute(String s) {
      return s.matches("\\d+");
    }
  }
  ```
- 구현한 클래스를 `다양한 검증 전략`으로 활용
  ```java
  public class Validator {
    private final ValidationStrategy strategy;
    
    public Validator(ValidationStrategy v) {
      this.strategy = v;
    }

    public boolean validate(String s) {
      return stategy.execute(s);
    }
  }

  Validator numericValidator = new Validator(new IsNumeric());
  boolean b1 = numericValidator.validate("aaaa"); // false
  Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
  boolean b2 = lowerCaseValidator.validate("bbbb"); // true
  ```

#### 람다 표현식 사용
- `ValidationStrategy`는 **함수형 인터페이스**며
  - `Predicate<String>`과 같은 **함수 디스크립터**를 갖고 있음을 파악
- 다양한 전략을 구현하는 새로운 클래스 구현 없이, **람다 표현식**을 전달하면 코드가 간단해짐
- 개선 코드
  ```java
  Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));
  boolean b1 = numericValidator.validate("aaaa");
  Validator lowerCaseValidator = new Validator((String s) -> s.matches("\\d+"));
  boolean b2 = lowerCaseValidator.validate("bbbb");
  ```
- 람다 표현식을 활용하면 **전략 디자인 패턴**에서 발생하는 자잘한 코드 제거 가능
- 람다 표현식은 **코드 조각**(전략)을 **캡슐화**
- **람다 표현식**으로 **전략 디자인 패턴**을 대체할 수 있음
  - 위와 비슷한 문제에서는 **람다 표현식**을 사용할 것

### 9.2.2. 템플릿 메서드
- 알고리즘 개요를 제시한 다음
  - 알고리즘의 일부를 고칠 수 있는 **유연함**을 제공해야할 때
  - **템플릿 메서드 디자인 패턴**을 사용
- `이 알고리즘을 사용하고 싶은데, 그대로는 안 되고 조금 고쳐야 하는` 상황
- 예시: 온라인 뱅킹 어플리케이션 구현
- 추상 클래스: 온라인 뱅킹 어플리케이션의 동작 정의
  ```java
  abstract class OnlineBanking {
    public void processCustomer(int id) {
      Customer c = Database.getCustomerWithId(id);
      makeCustomerHappy(c);
    }
    abstract void makeCustomerHappy(Customer c);
  }
  ```
  - `processCustomer` 메서드: 온라인 뱅킹 알고리즘이 해야할 일
    - 주어진 고객 ID를 이용해 고객 만족
  - 각각의 지점은 `OnlineBanking` 클래스를 상속받아 `makeCustomerHappy` 메서드가 원하는 동작을 수행하도록 구현

#### 람다 표현식 사용
- 알고리즘의 개요를 만든 이후, 구현자가 원하는 기능을 추가하기
- `makeCustomerHappy`의 메서드 시그니처와 일치하도록 
  - `Consumer<Customer>` 형식을 갖는 두번째 인수를 `processCustomer`에 추가
  ```java
  public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
  }
  ```
- 이제 `onlineBanking` 클래스를 상속받지 않고, 직접 람다 표현식 전달 가능
  ```java
  new OnlineBankingLambda().processCustomer(1337, (Customer c) -> System.out.println("Hello " + c.getName()));
  ```

### 9.2.3. 옵저버
- 여러 이벤트가 발생했을 때
  - 한 객체(`subject`, 주체)가 다른 객체 리스트(`observer`)에
  - 자동으로 **알림**을 보내야하는 상황에서
    - **옵저버 디자인 패턴**을 사용
- `GUI` 어플리케이션에서 자주 등장
- 버튼 같은 `GUI` 컴포넌트에 옵저버를 설정할 수 있음
- 그리고 사용자가 **버튼**을 클릭하면, **옵저버**에게 알림이 전달되고
  - 정해진 동작이 수행됨
- 예시로, 주식의 가격 변동에 반응하는 다수에 거래자 예제에서도 옵저버 패턴 사용 가능
- 옵저버 패턴 동작 원리
  - 트위터 같은 **커스터마이즈된 알림 시스템**을 설계하고 구현 가능
- 옵저버를 그룹화할 `Observer` 인터페이스 필요
  - 새로운 트윗이 있을 때 `Feed`가 호출할 수 있도록 `notify` 메서드 제공
  ```java
  interface Observer {
    void notify(String tweet);
  }
  ```
- 트윗에 포함된 다양한 키워드에 대한 다른 동작을 수행할 `여러 옵저버 정의`
  ```java
  class NYTimes implements Observer {
    public void notify(String tweet) {
      if(tweet != null && tweet.contains("money")) {
        System.out.println("Breaking news in NY! " + tweet);
      }
    }
  }
  
  class Guardian implements Observer {
    public void notify(String tweet) {
      if(tweet != null && tweet.contains("queen")) {
        System.out.println("Yet more news from London... " + tweet);
      }
    }
  }
  ```
- 주제도 구현, `Subject` 인터페이스 정의
  ```java
  interface Subject {
    void registerObserver(Observer o);
    void notifyObservers(String tweet);
  }
  ```
- 주제는 `registerObserver` 메서드로, 새로운 옵저버를 등록한 다음
  - `notifyObservers` 메서드로 트윗의 옵저버에 이를 알림
  ```java
  class Feed implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    
    public void registerObserver(Observer o) {
      this.observers.add(o);
    }

    public void notifyObservers(String tweet) {
      observers.forEach(o -> o.notify(tweet));
    }
  }
  ```
- 구현은 간단
  - `Feed`는 트윗을 받았을 때, 알림을 보낼 옵저버 리스트 유지
- 주제와 옵저버를 연결하는 데모 어플리케이션 제작 가능
  ```java
  Feed f = new Feed();
  f.registerObserver(new NYTimes());
  f.registerObserver(new Guardian());
  f.notifyObservers("The queen ...");
  ```

#### 람다 표현식 사용하기
- `Observer` 인터페이스를 구현하는 **모든 클래스**는, 하나의 메서드 `notify`를 구현
- 트윗이 도착했을 때, 어떤 동작을 수행할지 감싸는 코드를 구현
- 세 개의 옵저버를 명시적으로 인스턴스화 하지 않고
  - 람다 표현식을 직접 전달하여, 실행할 동작을 지정
- 코드
  ```java
  f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains(" money ")) {
      System.out.println("Breaking news in NY! " + tweet);
    }
  })
  f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains(" queen ")) {
      System.out.println("Yet more new from London..." + tweet);
    }
  });
  ```
- 람다 표현식을 항상 사용해야 하는가 -> No
  - 위와 같은 상황에서는, 실행할 동작이 `비교적 간단`하므로
  - 람다표현식으로 불필요한 코드를 제거하는 것이 바람직
- 하지만 `Observer`가 상태를 가지며, 여러 메서드를 정의하는 등 복잡할 경우
  - 람다 표현식보다 **기본의 클래스 구현방식**을 구현하는 게 더 바람직 할 수 있음

### 9.2.4. 의무 체인
- 작업 처리 객체의 **체인**(동작 체인 등)을 만들 때는 **의무 체인 패턴**을 사용
- 한 객체가 어떤 작업을 처리한 이후 `다른 객체로 결과 전달`하고,
  - 다른 객체도 `작업 처리`이후, 다른 객체로 `전달`하는 식
- 일반적으로 `다음으로 처리할 객체 정보`를 유지하는 필드를 포함하는 **작업 처리 추상 클래스**로
  - **의무 체인 패턴**을 구상
- 작업 처리 객체가 자신의 작업을 끝냈으면, 다음 처리 작업 객체로 결과 전달
- 코드
  ```java
  public abstract class ProcessingObject<T> {
    protected ProcessingObject<T> successsor;
    public void setSuccessor(ProcessingObject<T> sucessor) {
      this.successor = successor;
    }
    public T handle(T input) {
      T r = handleWork(input);
      if(successor != null) {
        return successor.handle(r);
      }
      return r;
    }
    abstract protected T handleWork(T input);
  }
  ```
- **9.2.2**의 **템플릿 메서드 디자인 패턴**이 사용됨
- `handle`은 일부 작업을 어떻게 처리해야할 지 전체적으로 기술
- `ProcessingObject` 클래스를 상속받아 `handleWork` 메서드를 구현하여, 다양한 종류의 작업 처리 객체 생성 가능
- 예시: 텍스트를 처리하는 객체
  ```java
  public class HeaderTextProcessing extends ProcessingObject<String> {
    public String handleWork(String text) {
      return "From Raoul, Mario and Alan: " + text;
    }
  }

  public class SpellCheckerProcessing extends ProcessingObject<String> {
    public String handleWork(String text) {
      return text.replaceAll("labda", "lambda");
    }
  }

  // 두 작업 처리 객체를 연결하여 작업 체인 생성 가능
  ProcessingObject<String> p1 = new HeaderTextProcessing();
  ProcessingObject<String> p2 = new SpellCheckerProcessing();
  p1.setSuccessor(p2); // 두 작업 처리 객체 연결
  String result = p1.handle("Aren't labdas really sexy?!!");
  System.out.println(result);
  ```

#### 람다 표현식 사용
- **함수 체인**과 유사함
- 작업 처리 객체를 `Function<String, String>`
  - 더 정확히 표현하자면 `UnaryOperator<String>` 형식의 **인스턴스**로 표현 가능
- `andThen` 메서드로 이들을 조합하여 체인 구성 가능
- 코드
  ```java
  UnaryOperator<String> headerProcessing =
    (String text) -> "From Raoul, Mario and Alan: "+ text; // 첫 번째 작업 처리 객체
  UnaryOperator<String> spellCheckerProcessing =
    (String text) -> text.replaceAll("labda", "lambda"); // 두 번째 작업 처리 객체
  Function<String, String> pipeline =
    headerProcessing.andThen(spellCheckerProcessing); // 동적 체인으로 두 함수 조합
  String result = pipeline.apply("Aren't labdas really sexy?!!");
  ```

### 9.2.5. 팩토리
- `인스턴스화 로직`을 클라이언트에 노출하지 않고, 객체를 만들 때
  - **팩토리 디자인 패턴**을 사용
- 코드
  ```java
  public class ProductFactory {
    public static Product createProduct(String name) {
      switch(name){
        case "loan": return new Loan();
        case "stock": return new Stock();
        case "bond": return new Bond();
        default: throw new RuntimeException("No such product" + name);
      }
    }
  }

  // 클라이언트 호출 코드
  Product p = ProductFactory.createProduct("loan");
  ```
  - `Loan, Stock, Bond` 모두 `Product`의 `Subtype`
  - `createProduct` 메서드는 메서드의 **생산된 상품**을 설정하는 로직 포함 가능

#### 람다 표현식 사용
- 생성자도 **메서드 참조**처럼 접근 가능
- 예시: `Loan`의 생성자 코드
  ```java
  Supplier<Product> loanSupplier = Loan::new;
  Loan loan = loanSupplier.get();

  // 상품명을 생성자로 연결하는 Map을 만들어 코드 재구현 가능
  final static Map<String, Supplier<Product>> map = new HashMap<>();
  static {
    map.put("loan", Loan::new);
    map.put("stock", Stock::new);
    map.put("bond", Bond::new);
  }

  // `Map`을 이용하여 **팩토리 디자인 패턴**에서 했던 것처럼, 다양한 상품의 인스턴스화 가능
  public static Product createProduct(String name) {
    Supplier<Product> p = map.get(name);
    if(p != null) return p.get();
    throw new IllegalArgumentException("No such product " + name);
  }
  ```
- 단, 팩토리 메서드 `createProduct`가 **상품 생성자**로
  - **여러 인수를 전달하는 과정에서는 이 기법 적용 어려움**
  - 단순한 `Supplier` 함수형 인터페이스로는 문제 해결 불가
- 예시로, 세가지의 인수(`Integer 2, String 1`)을 받는 **상품의 생성자**가 있다면,
  - 세 인수를 지원하려면 `TriFunction`이라는 특별한 **함수형 인터페이스**를 만들어야 함
  - `Map`의 시그니처가 복잡해짐
  - 코드
    ```java
    public interface TriFunction<T, U, V, R> {
      R apply(T t, U u, V v);
    }
    Map<String, TriFunction<Integer, Integer, String, Product>> map = new HashMap<>();
    ```

## 9.3. 람다 테스팅
- **단위 테스팅**(unit testing)
  - 소스코드의 일부가 예상된 결과를 도출할 것인지에 대한 테스트 케이스 작성
- 예시: 그래픽 어플리케이션의 일부인 `Point`
  ```java
  public class Point {
    private final int x;
    private final int y;

    private Point(int x, int y) {
      this.x = x;
      this.y = y;
    }
    public int getX() { return x; }
    public int getY() { return y; }
    public Point moveRightBy(int x) {
      return new Point(this.x + x, this.y);
    }
  }
  ```
- 단위 테스트 코드
  ```java
  @Test
  public void testMoveRightBy() throws Exception {
    Point p1 = new Point(5, 5);
    Point p2 = p1.moveRightBy(10);
    assertEquals(15, p2.getX());
    assertEquals(5, p2.getY());
  }
  ```

### 9.3.1. 보이는 람다 표현식의 동작 테스팅
- `moveRightBy`는 `public`이므로, 위 코드는 문제 없이 작동
- 단, 람다는 **익명**이므로,
  - 테스트 코드 이름을 호출할 수 없음
- 필요하다면, 람다를 **필드**에 저장해서 **재사용**할 수 있으며,
  - 람다의 로직을 **테스트**할 수 있음
- 메서드를 호출하는 것처럼 **람다**를 사용할 수 있음
- 예시: `Point` 클래스에 `compareByXAndThenY`라는 **정적 필드** 추가
  ```java
  public class Point {
    public final static Comparator<Point> compareByXAndThenY =
      comparing(Point::getX).thenComparing(Point::getY);
      ...
  }
  ```
- 람다 표현식은 **함수형 인터페이스의 인스턴스**를 생성
- 따라서 생성된 **인스턴스**의 동작으로 **람다 표현식**을 테스트할 수 있음
- 예시: `Comparator` 객체 `compareByXAndThenY`에 다양한 인수로 `compare` 메서드를 호출하면서 
  - 잘 동작하는지 테스트하는 코드
  ```java
  @Test
  public void testComparingTwoPoints() throws Exception {
    Point p1 = new Point(10, 15);
    Point p2 = new Point(10, 20);
    int result = Point.compareByXAndThenY.compare(p1, p2);
    assertTrue(result < 0);
  }
  ```

### 9.3.2. 람다를 사용하는 메서드의 동착에 집중하라
- 람다의 목표는 **정해진 동작을 다른 메서드에서 사용**할 수 있도록
  - 하나의 조각으로 **캡슐화**하는 것
- 그러려면, 세부 구현을 포함하는 **람다 표현식**을 공개하지 말아야 함
- 람다 표현식을 사용하는 메서드의 **동작**을 테스트 함으로써,
  - 람다를 공개하지 않으면서도, **람다 표현식 검증**을 할 수 있음
- 예시: `moveAllPointsRightBy`
  ```java
  public static List<Point> moveAllPointsRightBy(List<Point> points, int x) {
    return points.stream()
                  // 아래 람다 표현식을 테스트하는 부분이 없음
                  .map(p -> new Point(p.getX() + x, p.getY())
                  .collect(toList());
  }
  ```
- 위 코드에 **람다 표현식**을 테스트 하는 부분은 없음
  - 단순히 `moveAllPointsRightBy`를 구현한 메서드
- 이제 테스트가 가능
  ```java
  @Test
  public void testMoveAllPointsRightBy() throws Exception {
    List<Point> points = Arrays.asList(new Point(5, 5), new Point(10, 5));
    List<Point> expectedPoints = Arrays.asList(new Point(15, 5), new Point(20, 5));
    List<Point> newPoints = Point.moveAllPointsRightBy(points, 10);
    assertEquals(expectedPoints, newPoints);
  }
  ```
- 위 단위 테스트에서 보여주듯
  - `Point::equals`는 중요한 메서드
- 따라서 `Object::equals`를 그대로 사용하지 않으려면
  - `equals` 메서드를 적절히 구현해야 함

### 9.3.3. 복잡한 람다를 개별 메서드로 분할하기
- **9.1.3**처럼 메서드 참조로 바꾸는 것
- 일반 메서드를 테스트 하듯이 람다 표현식을 테스트 할 수 있음

### 9.3.4. 고차원 함수 테스팅
- 고차원 함수(`higher-order functions`)
  - 함수를 인수로 받거나, 다른 함수를 반환하는 메서드
- 메서드가 `람다를 인수로` 받는다면, 다른 람다로 메서드의 동작을 테스트 할 수 있음
- 예를 들어, 다양한 `predicate`로 2장에서 만든 `filter` 메서드를 테스트할 수 있음
  ```java
  @Test
  public void testFilter() throws Exception {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    List<Integer> even = filter(numbers, i -> i % 2 == 0);
    List<Integer> smallerThenThree = filter(numbers, i -> i < 3));
    assertEquals(Arrays.asList(2, 4), even);
    assertEquals(Arrays.asList(1, 2), smallerThanThree);
  }
  ```
- 테스트 해야할 메서드가 **다른 함수**를 반환한다면?
  - `Comparator`에서 살펴봤던 것처럼
  - **함수형 인터페이스**의 **인스턴스**로 간주하고, 함수의 동작을 테스트할 수 있음

## 9.4. 디버깅
- 문제가 발생하였을 때 아래 내용 확인 필요
  - stacktrace, logging
- 람다 표현식과 스트림은 기존의 디버깅 기법을 무력화

### 9.4.1. 스택 트레이스 확인
- 예외 발생으로 프로그램 실행이 갑자기 중단되었다면
  - `stack frame`에서 이 정보를 얻을 수 있음
- 프로그램이 메서드를 호출할 때마다
  - 프로그램에서의 호출 위치, 호출할 때의 인수값, 호출된 메서드의 지역 변수 등을 포함한 호출 정보
- 프로그램이이 멈췄다면 프로그램이 어떻게 멈추게 되었는지
  - `stack trace`를 얻을 수 있음
- 문제가 발생한 지점에 이르된 메서드 호출 리스트를 얻을 수 있음

#### 람다와 스택 트레이스
- 유감스럽게도 람다 표현식은 이름이 없기 때문에 조금 복잡한 스택 트레이스가 생성됨
  ```java
  // 문제가 발생하는 코드 예시
  public class Debugging {
    public static void main(String[] args) {
      List<Point> points = Arrays.asList(new Point(12, 2), null);
      points.stream().map(p -> p.getX()).forEach(System.out::println)
    }
  }
  ```
- 위 코드의 스택 트레이스 코드
  ```java
  Exception in thread "main" java.lang.NullPointerException
    at Debugging.lambda$main$0(Debugging.java:6) // $()은 어떤 의미?
    ...
  ```
- `points` 리스트의 둘째 인수가 `null`이므로 프로그램의 실행이 멈췄다
- 스트림 파이프라인에서 에러가 발생하였으므로
  - `스트림 파이프라인 작업`과 관련된 **전체 메서드 호출 리스트**가 출력
- 메서드 호출 리스트에 `$`가 포함된 정보도 존재
  - **람다 표현식** 내부에서 **에러**가 발생했음을 의미
- 람다 표현식은 이름이 없으므로 **컴파일러**가 **람다**를 참조하는 이름을 만들어냄
- `lmabda$main$0`은 다소 생소한 이름
  - 클래스에 여러 **람다 표현식**이 있을 때는 꽤 골치 아픈 일이 벌어짐
- **메서드 참조**를 사용해도 기존의 **람다 표현식**
  - `p -> p.getX()`를 메서드 참조 `Point::getX`로 고쳐도 `stack trace`로는 이상한 정보 출력
    ```java
    points.stream().map(Point::getX).forEach(System.out::println);

    Exception in thread "main" java.lang.NullPointerException
      at Debugging$$Lambda$5/284720968.apply(Unknown Source) // 어떤 의미?
    ```
- 메서드 참조를 사용하는 **클래스**와 같은 곳은 선언되어 있는 메서드를 참조할 때는
  - 메서드 참조 이름이 stack trace에 나타냄
    ```java
    public class Debugging{
      public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3);
        numbers.stream().map(Debugging::divideByZero).forEach(System.out::println);
      }
      public static int divideByZero(int n){
        return n / 0;
      }
    }
    ```
- `divideByZero` 메서드는 `stackstrace`에 제대로 표시됨
- 어쨌든 **람다 표현식**과 관련한 `stack trace`는 이해하기 어려울 수 있음

### 9.4.2. 정보 로깅
- 스트림 파이프라인의 연산을 **디버깅**한다면?
- 다음처럼 `foreEach`로 **스트림 결과**를 출력하거나 로깅 가능
  ```java
  List<Integer> numbers = Arrays.asList(2, 3, 4, 5);

  numbers.stream()
          .map(x -> x + 17)
          .filter(x -> x % 2 == 0)
          .limit(3)
          .forEach(System.out.println);
  ```
- `forEach`를 사용하는 순간, **전체 스트림이 소비**
- 스트림 파이프라인에 적용된 각각의 연산(`map, filter, limit`)이 어떤 결과를 도출하는지 확인할 수 있다면 좋음
- `peek` 스트림 연산 사용 가능
- `peek`는 스트림의 각 요소를 소비한 것처럼 **동작 싫행**
  - `forEach`처럼 스트림의 요소를 소비하지는 않음
- `peek`는 자신이 확인한 요소를 파이프라인의 다음 연산으로 그대로 전달
  - `peek`으로 스트림 파이프라인의 `각 동작 전후의 중간값을 출력`
- 예시 코드
  ```java
  List<Integer> result =
      numbers.stream()
              .peek(x -> System.out.println("from stream: " + x)) // 소스에서 처음 소비한 요소를 출력
              .map(x -> x + 17)
              .peek(x -> System.out.println("after map: " + x)) // map 동작 실행 결과를 출력
              .filter(x -> x % 2 == 0)
              .peek(x -> System.out.println("after filter: " + x)) // filter 동작 후 선택된 숫자 출력
              .limit(3)
              .peek(x -> System.out.println("after limit: " + x)) // limit 동작 후 선택된 숫자를 출력
              .collect(toList());
  ```

## 9.5. 마치며
- **람다 표현식**으로 가독성이 좋고 더 유연한 코드를 만들 수 있음
- `익명 클래스 -> 람다 표현식`으로 바꾸는 것이 좋음
  - 이때 `this`, 변수 섀도 등 미묘하게 의미상 다른 내용 존재
- **메서드 참조**로 람다 표현식보다 더 가독성이 좋은 코드를 구현할 수 있음
- 반복적으로 **컬렉션** 처리하는 루틴은 **스트림 API**로 대체할 수 있을지 고려하는 것이 좋음
- 람다 표현식으로 `전략, 템플릿 메서드, 옵저버, 의무 체인, 팩토리` 등의
  - **객체지향 디자인 패턴**에서 발생하는 불필요한 코드를 제거할 수 있음
- 람다 표현식도 **단위 테스트**를 수행할 수 있음
  - 단, 람다 표현식 자체를 테스트하는 것 보다는
  - 람다 표현식이 사용되는 **메서드의 동작**을 테스트하는 것이 바람직
- 복잡한 람다 표현식은 **일반 메서드**로 **재구현** 가능
- 람다 표현식을 사용하면 `stack trace`를 이해하기 어려워짐
- 스트림 파이프라인에서 요소를 처리할 때 `peek 메서드`로 중간값을 확인할 수 있음