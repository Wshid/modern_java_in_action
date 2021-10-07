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