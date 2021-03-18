# [CHAP.01] 자바8_9_10_11_무슨일이_일어나고_있는가

## 1.1. 역사의 흐름은 무엇인가
- java 8을 이용한 코드
  ```java
  inventory.sort(comparing(Apple::getWeight)); // 자연어에 더 가깝게 표현
  ```
- 하드웨어 적인 변화
  - 멀티 쓰레드, 멀티 프로세스
- 스레드의 활용
  - 관리가 어려우며, 문제가 발생할 수 있음
- 병렬 실행 환경을 쉽게 관리, 에러가 덜 발생하는 방향으로 진화
- java version별 특징
    - java 1.0 : thread, lock, memory model
    - java 5 : thread pool, cocurrent collection
    - java 7 : fork/join framework
    - java 8 : ?
    - java 9 : reactive programming
      - RxJava(Reactive Stream Toolkit)

### java 8의 특징
#### stream api
- 병렬 연산을 지원하는 API
- db의 query language로 원하는 동작을 표현하면
  - 구현(stream lib in java), 최적의 저수준 실행 방법 선택
  - 에러를 자주 일으키며, 멀티코어 CPU를 이용하는 것보다, 비용이 훨씬 비싼 synchronized 키워드를 사용하지 x
#### method에 code를 전달하는 기법
- 메서드 참조와 람다
#### interface의 default method

### 유의점
- **stream api 때문에 메서드에 코드를 전달하는 기법**이 생겼다고 추리하는 것은
  - 메서드에 코드를 전달하는 기법의 활용성을 제한할 수 있는 **위험한 생각**
- 동작 파라미터화(behavior parameterization)
  - 인수를 이용하여 다른 동작을 하는 두 메서드를 합칠 수 있음
- 함수형 프로그래밍(functional-style programming)

## 1.2. 왜 아직도 자바는 변화하는가
### 1.2.1. 프로그래밍 언어 생태계에서 자바의 위치
- 코드를 **JVM 바이트 코드**로 컴파일 하는 특징
  - 그리고 모든 브라우저에서 가상 머신 코드를 지원하기
  - 인터넷 애플릿 프로그램의 주요 언어가 된 이유
- JDK7에는 invokedynamic이라는 바이트 코드 추가
  - 경쟁언어를 JVM에서 더 잘 실행시키며, 상호 동작이 가능해짐
- 다양한 임베디드 컴퓨팅 분야에서 쓰임
  - 스마트카드, 포스터, 셋톱박스, 자동차 브레이크 시스템
- 빅데이터가 도래하면서, 멀티 코어 컴퓨터나, 컴퓨팅 클러스터를 사용해야 함
  - 병렬 프로세싱을 활용하는데, 현재 자바로는 대응 불가
- java 8의 기능
  - 병렬성을 활용하는 코드
  - 간결한 코드를 구현할 수 있도록 세가지 프로그래밍 개념 존재

### 1.2.2. 스트림 처리
- 스트림 : 한 번에 한 개씩 만들어지는 **연속적인 데이터 항목들의 모임**
- 특정 출력 스트림은, 다른 프로그램의 입력 스트림이 될 수 있음
- 유닉스 명령어의 `|`와 비슷
- `java.util.stream` 패키지에 **스트림 API**가 추가됨
  - `Stream<T>`
- 특징
  - 고수준으로 추상화하여, 일련의 스트림으로 만들어 처리 가능
  - 입력 부분을 **여러 CPU 코어에 할당**할 수 있음
  - 복잡한 작업을 사용하지 않으면서, **공짜**로 **병렬성**을 얻을 수 있음

### 1.2.3. 동작 파라미터화로 메서드에 코드 전달하기
- 코드 일부를 API로 전달하는 기능
- 함수형 프로그래밍 커뮤니티에서 실행하는 기술과 연관

### 1.2.4. 병렬성과 공유 가변 데이터
- **병렬성**을 공짜로 얻을 수 있다?
- 스트림 메서드로 전달하는 코드의 동작방식이 변경되어야 함
- 스트림 메서드로 전달하는 코드는
  - **다른 코드와 동시 실행되더라도 안전해야함**
- 안전하게 실행되는 코드
  - 공유된 가변 데이터(shared mutable data)에 접근하지 x
  - **pure function**, **side-effect-free func**, **stateless**
- 공유된 변수나 객채에 있으면, **병렬성**에 문제가 발생
- 기존처럼 `synchronized`를 이용하여
  - **공유된 가변 데이터**를 보호할 수는 있음
- java 8 stream을 이용하면
  - 기존의 자바 스레드 API보다 **병렬성 쉽게 확보**
- 다중 프로세싱 코어에서 `synchronized`를 사용하면
  - 다중 처리 코어에서는 코드가 순차적으로 실행되어야 하므로, 병렬성 무산
  - **비용이 높음**
- 공유되지 않은 가변 데이터, 메서드, 함수 코드를 **다른 메서드**로 전달하는 기능은
  - **함수형 프로그래밍** 패러다임의 핵심적인 사항
- **명령형 프로그래밍** 패러다임
  - 일련의 가변 형태로 프로그램 정의
- 공유되지 않은 가변 데이터 요구사항  
  - 인수를 결과로 반환하는 기능과 관련됨 
  - 즉, 요구사항은 **수학적인 함수**처럼, 함수가 정해진 기능만 수행
  - 부작용을 일으키지 않음

### 1.2.5. 자바가 진화하는 이유
- generic
- Iterator -> for-each
- 고전적인 객체지향에서 벗어나, **함수형 프로그래밍**에 다가섬
  - **우리가 하려는 작업**이 최우선시 되며,
  - 그 작업을 **어떻게 수정하는지**는 별개의 문제
- 극단적으로 말하면
  - 전통적인 객체지향 != 함수형 프로그래밍
- 결국, 하드웨어나 프로그래머 기대의 변화에 부응하는 방향으로 변화

## 1.3. 자바 함수
- 수학적인 함수 처럼 사용되며
  - 부작용을 일으키지 않는 함수
- 프로그래밍 언어는 값을 바꾸는 것
  - 전통적인 언어에서는 이 값을 **일급 시민**(first-class)이라고 부름
- 픅로그램을 구성하는 동안, 모든 구조체를 자유롭게 전달 불가
  - 메서드, 클래스 등은 **이급 자바 시민**
  - 메서드와 클래스는 그 자체로 값이 될 수 없음
- 런타임에 메서드를 전달할 수 있다면
  - 메서드를 1급 시민으로 만들경우, 프로그래밍에 활용가능
  - java 8에 추가된 기능

### 1.3.1. 메서드와 람다를 일급 시민으로
- 메서드를 값으로 활용하도록 지원

#### 메서드 참조(method reference)
```java
File[] hiddenFils = new File(".").listFiles(File::isHidden);
```
- `::`로 메서드 참조 형태 전달 가능

#### 람다 : 익명 함수
```java
(int x) -> x+1
```

### 1.3.2. 코드 넘겨주기 : 예시
```java
public static boolean isGreenApple(Apple apple) {
    return GREEN.equals(apple.getColor());
}

public static boolean isHeavyApple(Apple apple) {
    return apple.getWeight() > 150;
}

public interface Predicate<T>{ // 명확하게 하기 위해 추가(보통 java.util.function에서 import)
    boolean test(T t);
}

static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory) {
        if (p.test(apple)) {
            return .add(apple);
        }
    }
    return result;
}

// 다음과 같이 호출 가능
filterApples(inventory, Apple::isGreenApple);
filterApples(inventory, Apple::isHeavyApple);
```

#### Predicate
- 인수로 값을 받아 `boolean` 값을 반환
- `Function<Apple, Boolean>`으로 구성할 수 있으나, `Predicate<Apple>`을 사용하는 것이더 일반적

### 1.3.3. 메서드 전달에서 람다로
```java
filterApples(inventory, (Apple a) -> GREEN.euqlas(a.getColor()));
filterApples(inventory, (Apple a) -> a.getWeight() > 150);

// filter를 사용한 코드
filter(inventory, (Apple a) -> a.getWeight() > 150);
```
- java 8에서 `filter`와 비슷한 동작을 하는 연산집합을 포함하는
  - stream api를 지원
    - Collection과 비슷하며, 함수형 프로그래머에게 더 익숙한 api
  - 컬렉션과 스트림 간에 변환할 수 있는 `map`, `reduce`등의 메서드 제공

## 1.4. 스트림
```java
import static java.util.stream.Collectors.groupingBy;
Map<Currency, List<Transaction>> transactionByCurrencies = 
    transactions.stream()
        .filter((Transaction t) -> t.getPrice() > 1000)
        .collect(groupiungBy(Transaction::getCurrency));
```
- 스트림 API를 사용하면, 컬렉션 API와는 매우 다른 방식으로 처리 가능
- 컬렉션에서는 기존 반복 과정 직접 반복 -> external iteration(e.g. for-each, ...)
- 스트림에서는 루프를 신경쓰지 않아도 됨 -> internal iteration
- 또한 스트림을 사용하면 멀티 코어를 사용하여 처리됨

### 1.4.1. 멀티스레딩은 어렵다
- 이전 자바 버전의 thread api로 multi-threading을 구현하기는 어려움
  - 각 스레드가 공유된 데이터 접근 및 갱신
  - thread를 잘 제어하지 못할경우, 예상치 못한 상황 발생
- stream api(java.util.stream)으로 아래 이슈를 모두 해결
  - 컬렉션을 처리하면서 발생하는 모호함
  - 반복적인 코드문제
  - 멀티코어 활용 어려움
- **컬렉션**은 어떻게 데이터를 저장하고 접근할지에 중심
- **스트림**은 데이터에 어떤 계산을 할 것인지 묘사하는것에 중점
  - 스트림 내 요소를 **병렬**로 처리할 수 있는 환경을 제공함
- 컬렉션을 필터링할 수 있는 제일 빠른 방법
  - 컬렉션 -> 스트림 변환
  - 병렬 처리 이후
  - 리스트로 복원
- 코드
  ```java
  // 순차 처리 방식 코드
  import static java.util.stream.Collectors.toList;
  List<Apple> heavyApples =
    inventory.stream().filter((Apple a) -> a.getWeight() > 150)
                        .collect(toList());
  
  // 병렬 처리 방식 코드
  import static java.util.stream.Collectors.toList;
  inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150)
                            .collect(toList());
  ```

#### 자바의 병렬성과 공유되지 않은 가변 상태
- 병렬성은 어려우며, `synchronized`는 쉽게 에러를 일으킨다?
- java8의 대안
  - 라이브러리에서 분할 처리
    - 큰 스트림을 병렬로 처리 가능하도록 **작은 스트림**으로 분할
    - 또한 `filter`와 같은 라이브러리 메서드로 전달된 메서드가 **상호작용**하지 않는다면
      - 가변 공유 객체를 통해 공짜로 **병렬성**을 높일 수 있음
  - 함수형 프로그래밍에서 **함수형**이란, 
    - **함수**를 일급값으로 사용한다라는 의미도 있으나,
    - 프로그램이 실행되는 동안 **컴포넌트 간에 상호작용이 일어나지 않음**의 의미도 포함

## 1.5. 디폴트 메서드와 자바 모듈
- 특정 패키지의 인터페이스를 바꿔야 하는 상황
  - 인터페이스를 구현하는 모든 내용을 변경..?
  - java 8,9에서는 다른 방법으로 해결

### java 9 모듈 시스템
- 모듈을 정의하는 문법 제공
  - 패키지 모음을 포함하는 모듈 정의
  - jar와 같은 컴포넌트 구조에 적용 가능
  - 문서화, 모듈 확인 작업에 용이

### java 8 모듈 시스템
- 인터페이스를 쉽게 변경 가능하도록, **디폴트 메서드** 지원
- 프로그래머가 디폴트 메서드를 구현하는 상황 자체는 흔치 않음
- 기존의 구현을 고치지 않고, 공개된 인터페이스를 변경하는 방법?
  - 구현 클래스에서 구현하지 않아도, 메서드를 인터페이스에 추가하는 기능 제공
  - 기존의 코드를 건드리지 않고, 원래 인터페이스 설계를 자유롭게 확장 가능
- `default`라는 키워드
  ```java
  default void sort(Comparator<? super E> c) {
      Collections.sort(this, c);
  }
  ```
- 하나의 클래스에 여러 인터페이스 구현 가능 -> 다중 상속이 가능할까?
  - 어느 정도는 맞는 부분
- 다이아몬드 상속 문제에 대해서는 **CHAP.09**참조

## 1.6. 함수형 프로그래밍에서 가져온 다른 유용한 아이디어
- 함수형 프로그래밍의 핵심
  - 메서드와 람다를 **일급값**으로 사용하는 것
  - 가변 공유 상태가 없는 **병렬 실행**을 이용하여
    - 효율적이고 안전하게 **함수**나 **메서드**를 호출할 수 있음
- java 8에서는 `NP`를 피할수 있도록 `Optional<T>`를 제공
  - 값이 없는 상태를 어떻게 처리할지에 대한 부분
- **구조적 패턴 매칭**
  - `if-then-else`가 아닌 케이스로 정의하는 수학과 함수형 프로그래밍 기능을 의미
  - java8에서는 완벽하게 지원하지 않음
- scala에서는 패턴 매칭을 사용
  ```scala
  def simplifyExpression(expr: Expr):Expr = expr match {
    case BinOp("+", e, Number(0)) => e
    case BinOp("-", e, Number(0)) => e
    case BinOp("*", e, Number(1)) => e
    case BinOp("/", e, Number(1)) => e
    case _ => expr
  }
  ```
  - scala의 `expr match` -> java의 `switch(expr)`와 동일한 기능 수행
- java의 `switch`에는 **문자열**과 **기본값**만 가능
  - 타 함수형 언어에서는 다양한 값 활용 가능

### java 8, 9, 10, 11 기능
- java 8 : 메서드 전달, 람다
- java 9 : 대규모 컴포넌트 정의 및 사용하는 기능, **모듈 사용** 및 **리액티브 프로그래밍**을 임포트해 시스템 구축
- java 10 : 지역 변수 추론
- java 11 : 더 풍부해진 람다 표현식 인수 문법


## 1.7. 마치며
- 언어 생태계에서, 모든 언어는 변화하여 살아남거나, 그대로 머물면서 사라짐
  - 자바가 영원히 지배적인 위치를 유지할 수 있을지는 모름
- java 8, 효과적이고 간결하게 구현가능한 새로운 개념 / 기능 제공
- 기존 java 기법으로는, 멀티코어 프로세서를 온전히 활용하기 어려움
- 함수는 **일급 값**
  - 메서드를 어떻게 함수형으로 넘겨주는지, **익명 함수**(람다)를 어떻게 구현하는지 기억할 것
- java 8의 스트림 개념중 일부는 **컬렉션**에서 가져옴
  - 스트림과 컬렉션을 적절하게 사용하면,
    - **스트림의 인수를 병렬 처리**
    - 더 가독성이 좋은 코드 구현 가능
- 기존 자바 기능으로는 대규모 컴포넌트 기반 프로그래밍, 진화하는 시스템의 **인터페이스**를 대응하기 어려움
  - java 9에서는 **모듈**을 이용해 **시스템의 구조**를 만들 수 있고
  - **default method**를 이용하여 기존 인터페이스를 구현하는 **클래스 변경 없이 인터페이스 변경 가능**
- 함수형 프로그래밍에서 `null`처리 방법과 **패턴 매칭 활용**등 흥미로운 기법 발견