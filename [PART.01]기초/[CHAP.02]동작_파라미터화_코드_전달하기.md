# [CHAP.02] 동작 파라미터화 코드 전달하기
- 새로 추가한 기능은 쉽게 구현할 수 있어야 하며,
  - 장기적인 관점에서 **유지보수**가 쉬워야 함

### 동작 파라미터화
- behavior parameterization
- 자주 바뀌는 **요구사항**대응이 가능
- 아직 어떻게 실행할지 `결정하지 않은 코드 블록`
  - 이 코드블록은, 나중에 프로그램에서 호출
  - 실행은 나중으로 미뤄짐
- 메서드의 **인수**로 코드 볼록 전달 가능
- 컬렉션 처리시 다음과 같은 메서드 구현 가능
  - 리스트의 모든 요소에 대해 `어떤 동작`을 수행할 수 있음
  - 리스트 관련 작업을 끝낸 다음에 `어떤 다른 동작`을 수행할 수 있음
  - 에러가 발생하면 `정해진 어떤 다른 동작`을 수행할 수 있음
- 동작 파라미터화를 추가하면 **쓸데 없는 코드**가 늘어남
  - 다만, java 8에서는 **람다 표현식**으로 해결

## 2.1. 변화하는 요구사항에 대응하기

#### [CODE.2.1.1] 첫번째 시도 : 녹색 사과 필터링
```java
enum Color {RED, GREEN}
public static List<Apple> filterGreenApples(list<Apple> inventory) {
  List<Apple> result = new ArrayList<>(); // 사과 누적 리스트
  for(Apple apple: inventory) {
    if (GREEN.equals(apple.getColor())) { // 녹색 사과만 선택
      result.add(apple);
    }
  }
  return result;
}
```
- 다양한 색 변화에 취약함
- 비슷한 코드가 존재한다면, 그 코드를 추상화 해야함

### [CODE.2.1.2] 두번째 시도 : 색을 파라미터화
- `filterGreenApples`의 코드를 반복사용하지 않고, 구현하기
```java
enum Color {RED, GREEN}

// Color를 추가하여 대응
public static List<Apple> filterApplesByColor(list<Apple> inventory, Color color) {
  List<Apple> result = new ArrayList<>(); // 사과 누적 리스트
  for(Apple apple: inventory) {
    if (apple.getColor().equals(color)) {
      result.add(apple);
    }
  }
  return result;
}

// 다음과 같이 호출 가능
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```
- 색에 대해 동적으로 대응 가능하나, **무게**와 같은 특정 범주로 필터링이 필요할 경우
```java
// Color를 추가하여 대응
public static List<Apple> filterApplesByWeight(list<Apple> inventory, int weight) {
  List<Apple> result = new ArrayList<>(); // 사과 누적 리스트
  for(Apple apple: inventory) {
    if (apple.getWeight() > weight) {
      result.add(apple);
    }
  }
  return result;
}
```
- 하지만 **색**과 **무게**의 다수가 대부분 중복함
  - 소프트웨어 공학의 **DRY**(Don't Repeat Yourself)을 위반한 사례(같은 일을 반복하는 것)
- 어떤 기준으로 필터링 할지, **플래그**를 추가할 수 있는 있으나, 실전에서 사용하지 말 것

### [CODE.2.1.3] 세번째 시도 : 가능한 모든 속성으로 필터링
- 잘못된 예시
```java
// Color를 추가하여 대응
public static List<Apple> filterApples(list<Apple> inventory, Color color, int weight, boolean flag) {
  List<Apple> result = new ArrayList<>(); // 사과 누적 리스트
  for(Apple apple: inventory) {
    if((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) { // 색이나 무게를 선택하는 방법이 맘에 들지 않음
      result.add(apple);
    }
  }
  return result;
}

List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```
- `true`, `false`가 무얼 의미하는지 모호함
- 요구사항이 추가되었을 때 유연한 대응이 불가능
- **동작 파라미터화**로 유연성이 필요

## 2.2. 동작 파라미터화
- 사과의 특정 속성에 기초하여 `boolean`값을 반환하는 방법
- `true | false`를 반환하는 함수를 `predicate`라고 함
- **선택조건을 결정하는 인터페이스 정의**
  ```java
  public interface ApplePredicate {
    boolean test (Apple apple);
  }
  ```
- 다양한 선택 조건을 대표하는 여러 `ApplePredicate` 정의
  ```java
  public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
      return apple.getWeigt() > 150;
    }
  }

  public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
      return GREEN.equals(apple.getColor());
    }
  }
  ```
- 위 조건에 따라 `filter` 메서드가 다르게 동작
  - **전략 디자인 패턴**(Strategy Design Pattern)
    - 각 알고리즘(=전략)을 **캡슐화**하는 **알고리즘 패밀리**를 정의해둔 뒤
    - `Runtime`에 알고리즘을 선택하는 기법
- 위 예시에서
  - `ApplePredicate`가 알고리즘 패밀리
  - `AppleHeavyWeightPredicate`, `AppleGreenColorPredicate`가 전략
- `filterApples`에서 `ApplePredicate` 객체를 받아, 애플의 조건을 검사하도록 메서드를 고쳐야 함
- 동작 파라미터화
  - 메서드가 다양한 동작(전략)을 **받아**
  - 내부적으로 다양한 동작을 **수행**

### [CODE.2.2.1] 네번째 시도 : 추상적 조건으로 필터링
```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple: inventory) {
    if(p.test(apple)) { // predicate 객체로 사과 검사 조건을 캡슐화
      result.add(apple);
    }
  }
  return result;
}
```
- 동적으로 처리 될수는 있으나, `test`메서드를 `ApplePredicate`객체로 감싸서 전달해야 함 
- 람다를 사용하여 여러개의 `ApplePredicate` 클래스를 정의하지 않고 전달 가능

## 2.3. 복잡한 과정 간소화
- 익명 클래스를 통해 해결 가능

### 2.3.1. 익명 클래스
- 자바의 지역 클래스(`local class`)와 비슷한 개념
- 클래스 선언과 인스턴스화를 동시에 가능함
- 즉석에서 필요한 구현 가능 

### [CODE.2.3.2] 다섯 번째 시도 : 익명 클래스 사용
```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() { // filterApples 메서드의 동작을 직접 파라미터화
  public boolean test(Apple apple){
    return RED.equals(apple.getColor());
  }
});
```
- GUI 앱에서 많이 사용하는 방식
- 하지만 아직도 많이 장황한 방식
  - 코드의 가독성이 떨어짐

### [CODE.2.3.3] 여섯 번째 시도 : 람다 표현식 사용
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

### [CODE.2.3.4] 일곱 번째 시도 : 리스트 형식으로 추상화
```java
public interface Predicate<T> {
  boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
  List<T> result = new ArrayList<>();
  for(T e: list) {
    if(p.test(e)) {
      result.add(e);
    }
  }
  return result;
}

// 이후 다음과 같이 사용
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```

## 2.4. 실전 예제

### 2.4.1. Comparator로 정렬하기
- 컬렉션 정렬시 사용
- `java.util.Comparator`객체를 이용하여 `sort` 동작을 파라미터화
  ```java
  // java.util.Comparator
  public interface Comparator<T> {
    int comapre(T o1, T o2);
  }
  ```
- 예시 : 사과의 무게가 적은 순으로 정렬하기
  ```java
  inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
      return a1.getWeight().compareTo(a2.getWeight());
    }
  });

  // lambda를 사용하여 구현
  inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
  ```

### 2.4.2. Runnable로 코드 블록 실행
- `java thread`를 이용하여 병렬로 코드 블록 실행 가능
- `Runnable`을 사용하여, 각 스레드별 별도 실행 블록 지정 가능
  ```java
  // java.lang.Runnable
  public interface Runnable {
    void run();
  }
  ```
- 예시 : 다양한 동작 스레드로 실행하기
  ```java
  Thread t = new Thread(new Runnable() {
    public void run() {
      System.out.println("Hello world");
    }
  });
  
  // with lambda
  Thread t = new Thread(() -> System.out.println("Hello World"));
  ```

### 2.4.3. Callable을 결과로 반환하기
- java 5부터 `ExecutorService` 추상화 개념 지원
- `ExecutorService` 인터페이스
  - **task 제출**과, **실행 과정**의 연관성을 끊어줌
  - 이를 사용하면
    - `task`를 `thread pool`로 보내고, 결과를 `Future`로 저장할 수 있음
    - Thread/Runnable과는 다른 방식
- `Callable`인터페이스를 이용해, 결과를 반환하는 `task`를 만든다 -> `Runnable`의 업그레이드 버전
  ```java
  // java.util.concurrent.Callable
  public interface Callable<V> {
    V call();
  }
  ```
- `ExecutorService`에 `task`를 제출하여 코드 활용 가능
- 예시 : `task`를 실행하는 `thread name` 반환
  ```java
  ExecutorService executorService = Executors.newCachedThreadPool();
  Future<String> threadName = executorService.submit(new Callable<String>() {
    @Override public String call() throws Exception {
      return Thread.currentThread().getName();
    }
  });

  // lambda
  Future<String> threadName = executorService.submit(() -> Thread.currentThread().getName());
  ```

### 2.4.4. GUI 이벤트 처리하기
- 일반적으로 **GUI 프로그래밍**은
  - 마우스 클릭 / 문자열 위로 이동 등의 이벤트에 대응하는 동작을 수행하는 식으로 동작
- 예시 : `setOnAction` 메서드에 `EventHandler`를 전달하여, 이벤트에 어떻게 반응할지 지정
  ```java
  Button button = new Button("Send");
  button.setOnAction(new EventHandler<ActionEvent>(){
    public void handle(ActionEvent event) {
      label.setText("Sent!!");
    }
  });

  // lambda
  button.setOnAction((ActionEvent event) -> label.setText("Sent!!"));
  ```
- `EventHandler`는 `setOnAction` 메서드의 동작을 **파라미터화**

### 정리
- **동작 파라미터화**에서는
  - **메서드 내부적**으로 다양한 동작을 수행할 수 있도록, 코드를 **메서드 인수**로 전달
  - 변화하는 요구사항에 더 잘 대응할 수 있음
- **코드 전달 기법**을 이용하면, 동작을 **메서드의 인수**로 전달할 수 있음
  - java 8부터는
    - **익명 클래스**로도 어느 정도 코드를 정리할 수 있음
    - `interface`를 상속 받아, 여러 클래스를 구현하는 수고를 없앨 수 있음
- java api의 많은 메서드는
  - 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 **파라미터화** 가능