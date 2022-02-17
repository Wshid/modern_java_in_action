# [CHAP.10] 람다를 이용한 도메인 전용 언어
- DSL(Domain Specific Languages)
- DSL로 어플리케이션의 비즈니스 로직을 표현함으로써,
  - 버그와 오해를 미리 방지할 수 있음
- DSL은 `범용이 아닌` 특정 도메인을 대상으로 만들어진 특수 프로그래밍 언어
- DSL은 많은 특성 용어를 사용
  - maven, ant, ...
  - 빌드 과정을 표현하는 DSL
- 예시
  - 메뉴에서 400칼로리 이하의 모든 요리를 찾으시오
    ```java
    while (block != null) {
      read(block, buffer)
      for (every record in buffer) {
        if (record.calorie < 400) {
          System.out.println(record.name);
        }
      }
      block = buffer.next();
    }
    ```
  - 위 코드에는 두 가지 문제를 있음
    - locking, I/O, Disk 할당 같은 여러가지 지식 필요
    - App 수준이 아니라 시스템 수준의 개념을 다루어야 한다는 점
  - 다른 프로그래머는 다음과 같은 sql레벨에서 수정하다는 의견이 있을 수 있음
    ```sql
    SELECT name FROM menu WHERE calorie < 400
    ```
    - java가 아닌 SQL이라는 DSL을 이용해 DB 조작
    - 기술적으로 DSL을 `external`이라 함
    - DB가 텍스트로 구현된 SQL 표현식을 파싱하고 평가하는 API를 제공하는 것이 일반적이기 때문
- **내부적 DSL**과는 차이 존재
- Stream API를 사용한 예시
  ```java
  menu.stream()
      .filter(d -> d.getCalories() < 400)
      .map(Dish::getName)
      .forEach(System.out::println)
  ```
- Stream의 API의 특성인 **메서드 체인**을
  - 보통 자바의 루프의 복잡한 제어와 비교해 유창함을 의미하는 `fluent style`이라고 함
  - 위 예제에서 DSL은 **외부적**이 아니라 **내부적**
- **내부적 DSL**에서는
  - 위에서 언급한 SQL의 `SELECT FROM`구문처럼 애플리케이션 수준의 기본값이
  - 자바 메서드가 사용할 수 있도록 DB를 대표하는 한 개 이상의 **클래스** 형식으로 노출
- 기본적으로 DSL을 만들려면
  - app 수준 프로그래머에 어떤 동작이 필요하며 이들이 어떻게 프로그래머에게 제공하는지 고민 필요
  - 동시에 `시스템 수준`의 개념으로 인해 불필요한 오염이 발생하지 않음
- **내부적 DSL**에서는 유창하게 코드를 구현할 수 있도록
  - 적절하게 클래스와 메서드 노출 과정 필요
- **외부적 DSL**은 DSL 문법 뿐 아니라 `DSL을 평가하는 파서`도 구현해야 함

## 10.1 도메인 전용 언어 
- `DSL`: 특정 비즈니스 도메인의 해결을 위한 언어
  - e.g. 회계 전용 소프트웨어, 입출금 내역, 계좌 통합 같은 개념의 문제를 해결하기 위한 `DSL` 생성 가능
- 특정 비즈니스 도메인을 **인터페이스**로 만든 **API**라고 할 수 있음
- DSL는 **범용 프로그래밍 언어가 아님**
  - 동작, 용어 역시, 특정 **도메인에 국한**되므로
  - 다른 문제는 걱정할 필요가 없으며
  - 자신의 문제에만 집중하면 됨
- DSL을 사용할 경우
  - 특정 도메인의 **복잡성**을 더 잘 다룰 수 있음
- 저수준 구현 세부 사항 메서드는 **클래스**의 **비공개**로 만들어서
  - 저수준 구현 세부 내용을 숨길 수 있음
  - 그럴경우, 사용자 친화적인 `DSL` 생성 가능
- **DSL**과 거리가 먼 특징
  - 평문 **영어**가 아님
  - 도메인 전문가가 **저수준 비즈니스 로직**을 구현하게 만드는 것이 아님
- **DSL**의 두가지 필요성
  - **의사 소통의 왕**
    - 코드의 의도가 명확히 전달되어야 함
    - 프로그래머가 아닌 사람도 이해할 수 있어야 함
    - 코드가 **비즈니스 요구사항**에 부합하는지 확인 가능
  - **한번 코드를 구현하지만 여러번 읽음**
    - **가독성**은 유지보수의 핵심
    - 동료가 쉽게 이해할 수 있도록 코드 구현

### 10.1.1. DSL의 장점과 단점
- **DSL**은 모든 문제의 해결법이 아님
- 특정 도메인에 이용하면, 약이 되거나, 독이 될 수 있음
- DSL은
  - 코드의 **비즈니스 의도**를 명확하게 하고
  - 가독성을 높인다는 점에서 장점
- DSL 구현은
  - **코드**이므로, 올바로 **검증**하고 **유지보수**해야하는 책임이 따름
- **DSL**의 **장점과 비용**을 모두 확인해야
  - DSL을 추가하는 것이
  - 투자대비 긍정적인 결과를 가져올지를 올바로 평가할 수 있음

#### DSL의 장점
- **간결함**
  - API는 비즈니스 로직을 **간편하게 캡슐화**
  - 반복 회피 및 코드 간결화 가능
- **가독성**
  - 도메인 영역의 언어를 사용하므로,
    - `비 도메인 전문가`도 코드를 쉽게 이해할 수 있음
  - 다양한 조직 구성원 간에 **코드**와 **도메인 영역**이 공유될 수 있음
- **유지보수**
  - 잘 설계된 `DSL`로 구현한 코드는, 유지보수하고 바꿀 수 있음
  - 유지보수는 **비즈니스 관련 코드**에 중요
    - 가장 빈번히 변경
- **높은 수준의 추상화**
  - DSL은 `도메인`과 같은 **추상화 수준**에서 동작하므로
  - 도메인의 문제와 직접적으로 연관 되지 않은 **세부 사항을 숨김**
- **집중**
  - 비즈니스 도메인의 규칙을 표현할 목적으로 설계된 언어
  - 프로그래머가 특정 코드에만 집중 가능
- **관심사 분리**
  - 지정된 언어로 `비즈니스 로직`을 표현하므로
  - App의 인프라 구조와 관련된 문제와
    - 독립적으로 비즈니스 관련된 코드에서, 집중하기가 용이
  - 결과적으로 유지보수가 쉬운 코드 구현

#### DSL의 단점
- **DSL 설계의 어려움**
  - 간결하게 제한적인 언어에 **도메인 지식**을 담는 작업 => 어려움
- **개발 비용**
  - 초기 프로젝트에 **많은 비용/시간 소모**
  - 프로젝트에 부담을 주는 요소
- **추가 우회 계층**
  - DSL은 **추가적인 계층**으로 **도메인 모델**을 감싸는 형태
  - 계층을 최대한 작게 만들어 **성능 문제 회피**
- **새로 배워야 하는 언어**
  - DSL을 프로젝트에 추가함으로써, 팀이 배워야 하는 언어가 늘어남 => 부담
- **호스팅 언어 한계**
  - 일부 `java`와 같은 범용 프로그래밍 언어는
    - 장황하고 엄격한 문법을 가짐
  - 사용자 친화적 `DSL`을 만들기 어려움
  - `java 8`에서의 **람다 표현식**은 이 문제를 해결할 강력한 도구

### 10.1.2. JVM에서 이용할 수 있는 다른 DSL 해결책
- DSL의 **카테고리**를 구분하는 가장 흔한 방법은
  - 마틴 파울러(Martin Fowler)가 소개한 방법으로
  - **내부 DSL**과 **외부 DSL**을 나누는 것
- 내부 DSL(=임베디드 DSL)
  - **순수 자바 코드**와 같은 기존 **호스팅 언어**를 기반으로 구현
- 외부 DSL(standalone)
  - 호스팅 언어와는 **독립적**으로 **자체 문법**을 가짐
- JVM으로 인해 **내부 DSL**과 **외부 DSL**의 중간 카테로기에 해당하는 DSL이 만들어질 가능성이 존재
- `scala`나 `groovy`처럼
  - 자바가 아니지만 `JVM`에서 실행되며
  - 더 유연하고 표현력이 강력한 언어도 있음
- 이들을 **다중 DSL**이라는 **세 번째 카테고리**로 칭함

#### 내부 DSL
- 자바로 구현한 DSL
- **람다 표현식**이 등장하면서,
  - 읽기 쉽고, 간단하고, 표현력있는 DSL 생성 가능
- **동작 파라미터화**를 간단히 만드는데도 쓰임
- 신호 대비 잡음 비율
- 예시 코드
  ```java
  List<String> numbers = Arrays.asList("one", "two", "three");

  // 코드의 잡음 numbers.forEach
  numbers.forEach(new Consumer<String>() {
    @Override
    // 코드의 잡음 accept, System.out.println(s);
    public void accept(String s) {
      System.out.println(s);
    }
  });

  // 람다 표현식으로 개선
  numbers.forEach(s -> System.out.println(s));

  // 메서드 참조로 개선
  numbers.forEach(System.out::println);
  ```
  - 문법상 필요한 **잡음**이 존재
- 사용자가 **기술적인 부분**을 염두해 두고 있다면
  - 자바를 이용하여 `DSL`을 만들 수 있음
- 자바 문법이 문제가 아니라면,
  - **순수 자바**로 `DSL`을 구성하므로, 다음과 같은 장점을 얻을 수 있음

#### 내부 DSL의 장점
- 기존 자바 언어를 활용하기 때문에, **외부 DSL**에 비해 배워야하는 비용이 낮음
- 순수 자바로 DSL을 구현하면, 나머지 코드와 함께 **DSL 컴파일** 가능
  - 다른 언어의 컴파일러를 사용하거나,
  - 외부 DSL을 만드는 도구를 사용할 필요 없음 -> **추가 비용 x**
- `새로운 언어`를 배우거나, `외부 도구`를 배울 필요 없음
- DSL 사용자는 기존의 `Java IDE`를 이용해
  - **자동 완성**, **자동 리팩터링** 같은 기능을 그대로 사용 가능
- 한 개의 언어로 **한 개의 도메인** 또는 **여러 도메인**을 대응하지 못해
  - 추가로 `DSL`을 개발해야하는 상황에서
  - 자바를 이용하면, **추가 DSL**을 쉽게 합칠 수 있음

#### 다중 DSL
- `JVM`에서 실행되는 언어는 100개가 넘음
- `scala, ruby`외에 `JRuby, Jython`등도 존재
- `Kotlin`이나 `Ceylon`과 같이
  - scala와 호환성을 유지하면서, 단순하고 쉽게 배울 수 있는 언어도 존재
  - 위 언어들은 **간편한 문법**을 지향하도록 설계
- `DSL`은 기존 프로그래밍 언어의 영향을 받으므로
  - 간결한 DSL을 만드는데, **새로운 언어의 특성**이 중요
- `scala`은 `currying`, `임의 변환`등
  - `DSL` 개발에 필요한 여러 특성을 갖춤
- 예시: 주어진 함수 `f`를 주어진 횟수만큼 반복 실행
  ```scala
  def times(i: Int, f: => Unit): Unit = {
    f // f 함수 실행
    if ( i > 1) times(i-1, f) // 횟수가 양수면, 횟수를 감소시켜 재귀적으로 times 실행
  }

  // 3번 호출 예시
  times(3, println("Hello World"))
  ```
- `scala`에서는 `i`가 아주 큰 숫자여도, `stack overflow`가 발생하지 않음
  - **꼬리 호출 최적화**를 통해
  - `times` 함수 호출을 `stack`에 추가하지 않기 때문
- 추가 코드
  ```scala
  // `times` 함수를 커링하거나 두 그룹으로 인수를 놓을 수 있음
  def times(i: Int)(f: => Unit): Unit = { f
    if (i > 1) times (i - 1)(f)
  }

  // 여러번 실행할 명령을 `{}`안에 넣어 결과를 얻을 수 있음
  times(3) {
    println("Hello World")
  }
  ```
- `scala`는 함수가 **반복할 인수**를 받는 한 함수를 가지면서
  - `Int`을 익명 클래스로 **암묵적 변환**하도록 정의 가능
  - 코드 예시
    ```scala
    implicit def intToTimes(i: Int) = new { // Int를 `무명 클래스`로 변환하는 `암묵적 변환`을 정의
      def times(f: => Unit): Unit = { // 다른 함수 `f`를 인수로 받는 times 함수 한개만 정의
        def times(i: Int, f: => Unit): Unit = { // 가장 가까운 범주에서 정의한 두 개의 인수를 받는 함수 이용
          f
          if(i > 1) times(i-1, f)
        }
        times(i, f) // 내부 times 함수 호출
      }
    }

    // "Hello World" 세번 출력
    3 times {
      println("Hello World")
    }
    ```
  - 문법적 잡음이 전혀 없으며, 개발자가 아닌 사람도 코드 이해 가능
  - `3`은 컴파일러에 의해 **클래스 인스턴스**로 변환되며 `i`필드에 저장
  - `.`표기법을 이용하지 않고 `times`함수를 호출했는데,
    - 이 때 반복할 함수를 인수로 받음
- 위와 같은 예시는, **자바로 얻기 어려움**

#### 다중 DSL의 단점
- 새로운 프로그래밍 언어를 배우거나, 팀의 누군가가 이미 사용중이어야 함
  - `DSL`을 만드려면, 언어의 **고급 기능**을 활용할 수 있는 충분한 지식이 필요
- `두 개 이상의 언어`가 혼재하므로, 여러 `compiler`로 소스를 빌드하도록, **빌드 과정 개선 필요**
- `JVM`상의 언어도, `java`와의 호환성이 완벽하지 않을 수 있음
  - 호환성 때문에 **성능 손실**도 발생
  - e.g. `scala`와 `java`의 `Collection`은 **상호 호환 되지 않으므로**
    - 상호 컬렉션을 전달하려면, 기존 컬렉션을 대상 언어의 `API`에 맞게 변환 필요

#### 외부 DSL
- 자신만의 문법으로 **새 언어 설계**
- 새 언어를 **파싱**하고, 파서의 결과를 분석하고, 외부 `DSL`을 실행할 코드를 만들어야 함
- 매우 큰 작업
- 이 방법을 택해야 할 경우, `ANTLR`과 같은
  - **자바 기반 파서 생성기**를 이용하면 도움이 됨
- 논리 정연한 프로그래밍 언어를 새로 개발한다는 것은 **간단한 작업이 아님**
- **외부 DSL**을 쉽게 제어 범위를 벗어날 수 있으며
  - 처음 설계한 목적을 벗어나는 경우도 많음
- **무한한 유연성**이 장점
- 필요한 특성에 맞게 언어 구성 가능
- 제대로 언어를 설계하면
  - 비즈니스 문제를 묘사하고 해결하는 제일 좋은 언어 생성 가능
- 자바로 개발한 `인프라 구조의 코드`와 `비즈니스 코드`를 명확하게 분리할 수 있음
  - 하지만 이 분리로, `DSL`과 `호스트 언어`사이의 인공 계층이 생김

## 10.2. 최신 자바 API의 작은 DSL
- `Native Java API`
  - `java 8` 이전의 `Native Java API`는 이미 한 개의 **추상 메서드**를 가진 **인터페이스**를 가지고 있었음
  - `무명 내부 클래스`를 구현하려면, `불필요한 코드`가 추가되어야 함
  - `lambda`와 **메서드 참조**가 등장하면서 게임의 규칙이 바뀌었음
- `java 8`의 `Comparator interface`에 새로운 메서드가 추가됨
- `Compator interface`를 통해 `lambda`가
  - 어떻게 `Native Java API`의 **재사용성**과 **메서드 결합도**를 높였는지 확인
- 예시: `Persons`을 가리키는 객체 목록을 가지고 있음
  - 사람의 나이를 기준으로 **객체**를 정렬한다고 가정
  - `lambda`가 없으면 **내부 클래스**로 `Comparator` 인터페이스를 구현해야 함
  ```java
  Collections.sort(person, new Comparator<Person>() {
    public int compare(Person p1, Person p2) {
      return p1.getAge() - p2.getAge();
    }
  });

  // 내부 클래스를 람다 표현식으로 바꿀 수 있음
  Collections.sort(people, (p1, p2) -> p1.getAge() - p2.getAge());

  // Comparator.comparing method
  Collections.sort(persons, comparing(p -> p.getAge()));

  // lambda를 method 참조로 대신하여 코드 개선
  Collections.sort(persons, comparing(Person::getAge));

  // reverse
  Collections.sort(persons, comparing(Person::getAge).reverse());

  // 이름으로 비교를 수행하는 Comparator를 구현해 같은 나이의 사람들을 알파벳 순으로 정렬
  Coolections.sort(persons, comparing(Person::getAge).thenComparing(Person::getName));

  // List 인터페이스에 추가된 새 sort 메서드를 이용해 코드를 깔끔하게 정리할 수 있음
  persons.sort(comparing(Person::getAge).thenComparing(Person::getName));
  ```
- 위 작은 API는 **컬렉션 정렬 도메인**의 **최소 DSL**
  - **람다**와 **메서드 참조**를 이용한 `DSL`이 코드의 **가독성**, **재가용성**, **결합성**을 높일 수 있는지 보여줌

### 10.2.1. 스트림 API는 컬렉션을 조작하는 DSL
- `Stream` 인터페이스는 `Native Java API`에 작은 **내부 DSL**을 적용한 좋은 예시
- `Stream`은 **컬렉션**의 항목을 `필터, 정렬, 변환, 그룹화, 조작`하는 작지만 강력한 DSL
- 예시: `ERROR`라는 단어로 시작하는 파일의 **첫 40행**을 수집하는 작업을 수행한다고 가정

#### CODE.10.1. 반복 형식으로 예제 로그 파일에서 에러 행을 읽는 코드
```java
List<String> errors = new ArrayList<>();
int errorCount = 0;
BufferedReader bufferedReader = new BufferedReader(new FileReader(fileName));
String line = bufferedReader.readLine();
while (errorCount < 40 && line != null) {
  if (line.startsWith("ERROR")) {
    errors.add(line);
    errorCount++;
  }
  line = bufferedReader.readline();
}
```
- 위 코드는 장황하여 의도를 한 눈에 파악하기 어려움
- 같은 읨루를 지닌 코드가 여러 행에 분산되어 있음
  - `FileReader`가 만들어짐
  - 파일이 종료되었는지 확인하는 `while` 루프의 두 번째 조건
  - 파일의 다음 행을 읽은 `while` 루프의 마지막 행
- 첫 40행을 수집하는 코드도 세 부분
  - `errorCount` 변수를 초기화하는 코드
  - `while` 루프의 첫 번쨰 조건
  - `Error`을 로그에서 발견하면 카운터를 증가시키는 행

#### CODE.10.2. 함수형으로 로그 파일의 에러 행 읽음
```java
List<String> errors = Files.lines(Paths.get(fileName)) // 파일을 열어서 문자열 스트림의 만듦
                           .filter(line -> line.startsWith("ERROR")) // ERROR로 시작하는 행을 필터링
                           .limit(40) // 결과를 첫 40행으로 제한
                           .collect(toList()); // 결과 문자열을 리스트로 수집
```
- `String`은 파일에서 파싱할 행을 의미하여
  - `Files.lines`는 **정적 유틸리티 메서드**로 `Stream<String>`을 반환
  - 파일을 한 행씩 읽은 부분의 코드는 이게 전부
- 마찬가지로 `limit(40)`이라는 코드로 **에러 행**을 `첫 40개`만 수집
- `Stream API`의 `fluent` 형식은 잘 설계된 `DSL`의 또 다른 특징
- 모든 중간 연산은 게으르며,
  - 다른 연산으로 파이프라인될 수 있는 스트림으로 반환
- 최종 연산은 적극적이며 전체 파이프라인이 계산을 일으킴

### 10.2.2. 데이터를 수집하는 DSL인 Collectors
- `Stream` 인터페이스를 데이터 리스트를 조작하는 `DSL`로 간주할 수 있었음
- `Collector` 인터페이스는 **데이터 수집**을 수행하는 `DSL`로 간주할 수 있음
- 특히 `Comparator` 인터페이스는
  - **다중 필드 정렬**을 지원하도록 합쳐질 수 있으며
- `Collectors`는 **다중 수준 그룹화**를 달성할 수 있도록 합쳐질 수 있음
- 예제: 자동차를 브랜드별, 색상별로 그룹화
  ```java
  Map<String, Map<Color, List<Car>>> carsByBrandAndColor =
        cars.stream().collect(groupingBy(Car::getBrand, groupingBy(Car::getColor)));
  ```
- `Comparators`를 연결하는 것과 비교할때의 차이점?
  - 두 `Comparator`를 `fluent`방식으로 연결해서 다중 필드 `Comparator`를 정의
    ```java
    Comparator<Person> comparator =
        comparing(Person::getAge).thenComparing(Person::getName);
    ```
- `Collectors API`를 활용해 `Collectors`를 중첩, 다중 수준 `Collector`를 만들 수 있음
  ```java
  Collector<? super Car, ? Map<Brand, Map<Color, List<Car>>>> carGroupingCollector =
      groupingBy(Car::getBrand, groupingBy(Car::getColor));
  ```
- 셋 이상의 컴폰넌트를 조합할 때는, 보통 `fluent` 형식이 **중첩 형식**에 비해 가독성이 좋음
- 형식이 중요할까?
  - 가장 안쪽의 `Collector`가 첫 번째로 평가되어야 하지만,
  - 논리적으로는 **최종 그룹화**에 해당하므로,
    - 서로 다른 형식은 이를 어떻게 처리하는지를 **쌍반적으로** 보여줌
- 위 예제에서 `fluent` 형식으로 `Collector`를 연결하지 않고,
  - `Collector`생성을 여러 **정적 메서드**로 중첩함으로써
  - 안쪽 그룹화가 처음 평가되고, 코드에서는 반대로 나중에 등장
- `groupingBy` 팩터리 메서드에 작업을 위임하는 `GroupingBuilder`를 만들면, 문제를 쉽게 해결할 수 있음
  - `GroupingBuilder`는 유연한 방식으로 여러 **그룹화 작업**을 만듦

#### CODE.10.3. 유연한 그룹화 컬렉터 빌더
```java
public class GroupingBuilder<T, D, K> {
  private final Collector<? super T, ?, Map<K, D>> collector;

  private GroupingBuilder(Collector<? super T, ?, Map<K, D>> collector) {
    this.collector = collector;
  }

  public Collector<? super T, ?, Map<K, D>> get() {
    return collector;
  }

  public <J> GroupingBuilder<T, Map<K, D>, J> after(Function<? super T, ? extends J> classifier) {
    return new GroupingBuilder<>(groupingBy(classifier, collector));
  }

  public static <T, D, K> GroupingBuilder<T, List<T>, K> groupOn(Function<? super T, ? extends K> classifier) {
    return new GroupingBuilder<>(groupingBy(classifier));
  }
}
```
- `fluent` 형식 빌더의 문제?
  ```java
  Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>>
    carGroupingCollector =
      groupOn(Car::getColor).after(Car::getBrand).get()
  ```
  - 중첩화 그룹화 수준에 **반대로 그룹화 함수 구현**해야 하므로
    - 유틸리티 사용코드가 직관적이지 않음
  - 자바 형식 시스템으로는 이런 순서 문제를 해결할 수 없음

## 10.3. Java로 DSL을 만드는 패턴과 기법
- `DSL`은 특정 **도메인 모델**에 적용할 가독성 높은 API를 제공
- 따라서 가장 간단한 도메인 모델을 정의하면서 시작
- 예시: 주어진 시장에 주식 가격을 모델링 하는 순수 Java beans
```java
// 주식 가격
@Getter
@Setter
public class Stock {
  private String symbol;
  private String market;
}

// 거래
@Getter
@Setter
public class Trade {
  public enum Type { BUY, SELL }
  private Type type;

  private Stock stock;
  private int quantity;
  private double price;
}

// 주문
@Getter
@Setter
public class Order {
  private String customer;
  private List<Trade> trades = new ArrayList<>();
}
```
- 도메인 모델은 직관적
- `주문을 의미하는 객체`를 만드는 것은 조금 번거로움
- `BigBank`라는 고객이 요청한 두 거래를 포함하는 주문 만들기

#### CODE.10.4. 도메인 객체의 API를 직접 이용해 주식 거래 주문 생성
```java
Order order = new Order();
order.setCustomer("BigBank");

Trade trade1 = new Trade();
trade1.setType(Trade.Type.BUY);

Stock stock1 = new Stock();
stock.setSymbol("IBM");
stock.setMarket("NYUSE");

trade1.setStock(stock1);
trade1.setPrice(125.00);
trade1.setQuantity(80);
order.addTrade(trade1);
```
- 코드가 상당히 장황한 편
- 비개발자인 **도메인 전문가**가 위 코드를 이해하고 검증하기를 기대할 수 없기 때문
- 조금 더 직접적이고, 직관적으로 도메인 모델을 반영할 수 있는 `DSL`이 필요

### 10.3.1. 메서드 체인
- `DSL`의 가장 흔한 방식중 하나
- 이 방법을 이용하면, 한 개의 메서드 호출 체인으로 거래 주문 정의 가능

#### CODE.10.5. 메서드 체인으로 주식 거래 주문 만들기
```java
Order order = forCustomer("BigBank")
      .buy(80)
      .stock("IBM")
      .on("NYSC")
      .at(125.00)
      .sell(50)
      .stock("GOOGLE")
      .on("NASDAQ")
      .at(375.00)
      .end();
```
- 위 결과를 달성하려면
  - `fluent api`로 **도메인 객체를 만드는 몇개의 빌더 구현**이 필요
- 다음처럼 **최상위 수준 빌더**를 만들고, 주문을 감싼 다음
  - 한 개 이상의 거래를 주문에 추가할 수 있어야 함

#### CODE.10.6. 메서드 체인 DSL을 제공하는 주문 빌더
```java
public class MethodChainingOrderBuilder {
  public final Order order = new Order(); // 빌더로 감싼 주문

  private MethodChainingOrderBuilder(String customer) {
    order.setCustomer(customer);
  }

  public static MethodChainingOrderBuilder forCustomer(String customer) {
    return new MethodChainingOrderBuilder(customer); // 고객의 주문을 만드는 정적 팩터리 메서드
  }

  public TradeBuilder buy(int quantity) {
    return new TradeBuilder(this, Trade.Type.BUY, quantity); // 주식을 사는 TradeBuilder 만들기
  }

  public TradeBuilder sell(int quantity) {
    return new TradeBuilder(this, Trade.Type.SELL, quantity); // 주식을 파는 TradeBuilder 만들기
  }

  public MethodChainingOrderBuilder addTrade(Trade trade) {
    order.addTrade(trade); // 주식에 주문 추가
    return this; // 유연하게 추가 주문을 만들어 추가할 수 있도록 주문 빌더 자체 반환
  }

  public Order end() {
    return order; // 주문 만들기를 종료하고 반환
  }
}

public class TradeBuilder {
  private final MethodChainingOrderBuilder builder;
  public final Trade trade = new Trade();

  private TradeBuilder(MethodChainingOrderBuilder builder, Trade.Type type, int quantity) {
    this.builder = builder;
    trade.setType(type);
    trade.setQuantity(quantity);
  }

  public StockBuilder stock(String symbol) {
    return new StockBuilder(builder, trade, symbol);
  }
}
```
- 주문 빌더의 `buy()`, `sell()`메서드는
  - 다른 주문을 만들어 추가할 수 있도록 자신을 만들어 반환
- 빌더를 계속 이어나가려면, `Stock` 클래스의 인스턴스를 만드는
  - `TradeBuilder`의 공개 메서드를 이용해야함
  ```java
  public class StockBuilder {
    private final MethodChainingOrderBuilder builder;
    private final Trade trade;
    private final Stock stock = new Stock();

    private StockBuilder(MethodChainingOrderBuilder builder, Trade trade, String symbol) {
      this.builder = builder;
      this.trade = trade;
      stock.setsymbol(symbol);
    }

    public TradeBuilderWithStock on(String market) {
      stock.setMarket(market);
      trade.setStock(stock);
      return new TradeBuilderWithStock(builder, trade);
    }
  }

  public TradeBuilderWithStock {
    private final MethodChainingOrderbuilder builder;
    private final Trade trade;

    public TraddeBuilderWithStock(MethodChainingOrderBuilder builder, Trade trade) {
      this.builder = builder;
      this.trade = trade;
    }

    public MethodChainingOrderBuilder at(double price) {
      trade.setPrice(price);
      return builder.addTrade(trade);
    }
  }
  ```
- `StockBuilder`는 주식의 시장을 지정하고, 거래에 주식을 추가하고, 최종 빌더를 반환하는 `on()`메서드 한개를 지정
- 한 개의 공개 메서드 `TradeBuilderWithStock`은
  - 주식의 단위 가격을 설정한 다면, 원래 주문 빌더를 반환
- `MethodChainingOrderBuilder`가 끝날 때까지
  - 다른 거래를 `fluent`방식으로 추가 가능
- 여러 **빌드 클래스**
  - 특히 두 개의 거래 빌더를 따로 만듦으로써
  - 사용자가 미리 지정된 절차에 따라 `fluent api`의 메서드를 호출하도록 강제
- 덕분에 사용자가 다음 거래를 설정하기 전에, 기존 거래를 올바르게 설정하게 됨
- 이 접근 방법은
  - 주문에 사용한 **파라미터**가 **빌더 내부로 국한**된다는 이점 제공
  - **정적 메서드 사용**을 최소화 하고, 메서드 이름이 인수의 이름을 대신하도록 만듦으로써
    - 이런 형식의 `DSL`의 가독성을 개선하는 효과를 더함
  - `fluent DSL`에는 문법적 잡음이 최소화
- 단, **빌더 구현**이 **메서드 체인**의 단점
- `상위 수준의 빌더`를 `하위 수준의 빌더`와 연결할 **접착 코드**가 필요
  - 도메인의 객체의 중첩 구조와 일치하게 **들여쓰기 방법을 강제할 방법이 없음**

### 10.3.2. 중첩된 함수 이용
- 다른 함수안에 함수를 이용해 도메인 모델 생성

#### CODE.10.7. 중첩된 함수로 주식 거래 만들기
```java
Order order = order("BigBank",
                      buy(80,
                          stock("IBM", on("NYSE")), at(125.00)),
                      sell(50,
                          stock("GOOGLE", on("NASDAQ")), at(375.00))
);
```

#### CODE 10.8. 중첩된 함수 DSL을 제공하는 주문 빌더
```java
public class NestedFunctionOrderBuilder {
  public static Order order(String customer, Trade... trades) {
    Order order = new Order(); // 해당 고객의 주문 생성
    order.setCustomer(customer);
    Stream.of(trades).forEach(order::addTrade); // 주문에 모든 거래 추가
    return order;
  }

  public static Trade buy(int quantity, Stock stock, double price) {
    return buildTrade(quantity, stock, price, Trade.Type.BUY); // 주식 매수 거래 생성
  }

  public static Trade sell(int quantity, Stock stock, double price) {
    return buildTrade(quantity, stock, price, Trade.Type.SELL); // 주식 매도 거래 생성
  }

  private static Trade buildTrade(int quantity, Stock stock, double price, Trade.Type buy) {
    Trade trade = new Trade();
    trade.setQuantity(quantity);
    trade.SetType(buy);
    trade.setStock(stock);
    trade.setPrice(price);
    return trade;
  }

  public static double at(double price) { // 거래된 주식의 단가를 정의하는 더미 메서드
    return price;
  }

  public static Stock stock(String symbol, String market) {
    Stock stock = new Stock(); // 거래된 주식 만들기
    stock.setSymbol(symbol);
    stock.setMarket(market);
    return stock;
  }

  public static String on(String market) { // 주식이 거래된 시장을 정의하는 더미 메서드 정의
    return market;
  }
}
```
- 메서드 체인 방식에 비해 **함수 중첩 방식**이
  - **도메인 객체 계층 구조**에 그대로 반영된다는 것이 장점
- 단, `DSL`에 **많은 괄호**를 사용해야하는 것이 단점
- 또한 인수 목록을 **정적 메서드**에 넘겨주어야 함
- 도메인 객체에 **선택 사항 필드**가 있다면 **인수 생략**이 가능하므로
  - 여러 메서드를 `override`하여 구현해야 함
- 인수의 의미가 **이름**이 아닌 **위치**에 의해 정의
- `NestedFunctionOrderBuilder`의 `at()`, `on()` 메서드에서 했던 것처럼
  - 인수의 역할을 확실하게 만드는 여러 더미 메서드를 이용하여, 문제 완화가 가능

### 10.3.3. 람다 표현식을 이용한 함수 시퀀싱
- 람다 표현식으로 정의한 함수 시퀀스를 사용하기

#### CODE.10.9. 함수 시퀀싱으로 주식 거래 주문 만들기
```java
Order order = order { o -> {
  o.forCustomer("BigBank");
  o.buy(t -> {
    t.quantity(80);
    t.price(125.00);
    t.stock(s -> {
      s.symbol("IBM");
      s.market("NYSE");
    });
  });
  o.sell(t -> {
    t.quantity(50);
    t.price(375.00);
    t.stock(s -> {
      s.symbol("GOOGLE");
      s.market("NASDAQ");
    });
  });
}};
```
- 위와 같은 DSL을 만들려면
  - 람다 표현식을 받아 실행해
  - **도메인 모델**을 만들어 내는 여러 **빌더**를 구현해야 함
- 이 빌더는 **메서드 체인 패턴**을 이용해 만들려는 **객체의 중간 상태**를 유지
- **메서드 체인 패턴**에는 주문을 만드는 **최상위 빌더**를 가졌지만,
  - 이번에는 `Customer`객체를 빌더가 인수로 받음으로서
  - DSL 사용자가 **람다 표현식**으로 인수를 구현할 수 있게 함

#### CODE.10.10. 함수 시퀀싱 DSL을 제공하는 주문 빌더
```java
public class LambdaOrderBuilder {

  private Order order = new Order(); // 빌더로 주문을 감쌈

  public static Order order(Customer<LambdaOrderBuilder> consumer) {
    LambdaOrderBuilder builder = new LambdaOrderBuilder();
    // 주문 빌더로 전달된 람다 표현식 실행
    consumer.accept(builder);
    // OrderBuilder의 Customer를 실행해 만들어진 주문 반환
    return builder.order;
  }

  public void forCustomer(String customer) {
    order.setCustomer(customer); // 주문을 요청한 고객 설정
  }

  public void buy(consumer<TradeBuilder> consumer) {
    // 주식 매수 주문을 만들도록 TradeBuilder 소비
    trade(consumer, Trade.Type.BUY);
  }

  public void sell(Consumer<TradeBuilder> consumer) {
    // 주식 매도 주문을 만들도록 TradeBuilder 소비
    trade(consumer, Trade.Type.SELL);
  }

  private void trade(Consumer<TradeBuilder> consumer, Trade.Type.type) {
    TradeBuilder builder = new TradeBuilder();
    builder.trade.setType(type);
    // TradeBuilder로 전달할 람다 표현식 실행
    consumer.accept(builder);
    // TradeBuilder의 Consumer를 실행해 만든 거래를 주문에 추가
    order.addTrade(builder.trade);
  }
}
```
- `buy(), sell` 메서드는 두 개의 `Consumer<TradeBuilder>` 람다 표현식을 받음
  - 이 람다 표현식을 실행하면, 다음처럼 거래가 만들어짐
  ```java
  public class TradeBuilder {
    private Trade trade = new Trade();

    public void quantity(int quantity) {
      trade.setQuantity(quantity);
    }

    public void price(double price) {
      trade.setPrice(price);
    }

    public void stock(Consumer<StockBuilder> consumer) {
      StockBuilder builder = new StockBuilder();
      consumer.accept(builder);
      trade.setStock(builder.stock);
    }
  }

  public class SockBuilder {
    private Stock stock = new Stock();

    public void symbol(String symbol) {
      stock.setSymbol(symbol);
    }

    public void market(String market) {
      stock.setMarket(market);
    }
  }
  ```

#### 해당 패턴의 장/단점
- 장점
  - **메서드 체인 패턴**처럼 **플루언트 방식**으로 거래 주문 정의 가능
  - **중첩 함수 형식**처럼 **람다 표현식**의 중첩 수준과 비슷하게
    - 도메인 객체의 계층 구조 유지
- 단점
  - 많은 설정 코드가 필요함
  - DSL 자체가 `java 8` lambda expression 문벙에 의한 잡음에 영향을 받음

#### DSL 사용 결정하기
- 자신의 기호에 따라 다름
- 자신이 만들려는 **도메인 언어**에 어떤 도메인 모델이 맞는지 찾으려면
  - 실험을 해야함

### 10.3.4. 조합하기
- 세가지 `DSL` 패턴 각자가 **장단점**을 가지고 있음
- 하나의 `DSL` 패턴만 사용하라는 법은 없음

#### CODE.10.11. 여러 DSL 패턴을 이용해 주식 거래 주문 만들기
```java
Order order =
  forCustomer("BigBank", // 최상위 수준 주문 속성 지정하는 중첩 함수
    buy(t -> t.quantity(80) // 한개의 주문을 만드는 람다 표현식
              .stock("IBM") // 거래 객체를 만드는 람다 표현식 바디의 메서드 체인
              .on("NYSE")
              .at(125.00)),
    sell(t -> t.quantity(50)
               .stock("GOOGLE")
               .on("NASDAQ")
               .at(125.00))
  );  
```
- **중첩된 함수 패턴**과 **람다** 기법 혼용

#### CODE.10.12. 여러 형식을 혼합한 DSL을 제공하는 주문 빌더
- `TradeBuilder`의 `Consumer`가 만든 각 거래는
  - 다음 코드처럼 **람다 표현식**으로 구현
```java
public class MixedBuilder {
  public static Order forCustomer(String customer, TradeBuilder... builders) {
    Order order = new Order();
    order.setCustomer(customer);
    Stream.of(builders).forEach(b -> order.addTrade(b.trade));
    return order;
  }

  public static TradeBuilder buy(Customer<TradeBuilder> customer) {
    return buildTrade(customer, Trade.Type.BUY);
  }

  public static TradeBuilder sell(Customer<TradeBuilder> customer) {
    return buildTrade(customer, Trade.Type.SELL);
  }

  private static TradeBuilder buildTrade(Consumer<TradeBuilder> consumer, Trade.Trade buy) {
    TradeBuilder builder = new TradeBuilder();
    builder.trade.setType(buy);
    consumer.accept(builder);
    return builder;
  }
}
```
- 헬퍼 클래스 `TradeBuilder`와 `StockBuilder`는 내부적으로 **메서드 체인 패턴**을 구현해 `Fluent API`를 제공
  - 람다 표현식 바디를 구현해 가장 간단하게 거래 구현 가능
  - 코드
    ```java
    public class TradeBuilder {
      private Trade trade = new Trade();

      public TradeBuilder quantity(int quantity) {
        trade.setQuantity(quantity);
        return this;
      }

      public TradeBuilder at(double price) {
        trade.setPrice(price);
        return this;
      }

      public StockBuilder stock(String symbol) {
        return new StockBuilder(this, trade, symbol);
      }
    }

    public class StockBuilder {
      private final TradeBuilder builder;
      private final Trade trade;
      private final Stock stock = new Stock();

      private StockBuilder(TradeBuilder builder, Trade trade, String symbol) {
        this.builder = builder;
        this.trade = trade;
        stock.setSymbol(symbol);
      }

      public TradeBuilder on(String market) {
        stock.setMarket(market);
        trade setStock(stock);
        return builder;
      }
    }
    ```

#### 혼합한 DSL 패턴의 특징
- 여러 패턴의 장점을 이용할 수 있으나,
  - 여러 DSL이 섞여있어, 상대적으로 DSL을 배우는데 오랜 시간이 걸릴 수 있음

### 10.3.5. DSL에 메서드 참조 사용하기
- 기존 예시에
  - 주문의 총 합에 0개 이상의 세금을 추가해 최종값을 계산하는 기능 추가

#### CODE.10.13. 주문의 총 합에 적용할 세금
```java
public class Tax {
  public static double regional(double value) {
    return value * 1.1;
  }

  public static double general(double value) {
    return value * 1.3;
  }

  public static double surcharge(double value) {
    return value * 1.05;
  }
}
```
- 세금을 적용할 것인지 결정하는
  - **불리언 플래그**를 인수로 받는 정적 메서드를 이용해 간단하게 해결할 수 있음

#### CODE.10.14. 불리언 플래그 집합을 이용해 주문에 세금 적용
```java
public static double calculate(Order order, boolean useReional, boolean useGeneral, boolean useSurcharge) {
  double value = order.getValue();
  if (useRegional) value = Tax.regional(value);
  if (useGeneral) value = Tax.general(value);
  if (useSurcharge) value = Tax.surcharge(value);
  return value;
}

// 지역 세금고 추가 요금 적용, 일반 세금은 뺀 주문의 최종값을 계산할 수 있음
double value = calculate(order, true, false, true);
```
- 단 위 구현에는 **가독성 문제** 존재
  - 불리언 변수의 순서 기억 어려움
  - 어떤 세금이 적용되었는지 어려움
- 오히려 아래의 유창하게 불리언 플래그를 설정하는 **최소 DSL 제공**, `TaxCalculator`를 이용하는게 더 좋음

#### CODE.10.15. 적용할 세금을 유창하게 정의하는 세금 계산기
```java
public class TaxCalculator {
  private boolean useRegional;
  private boolean useGeneral;
  private boolean useSurcharge;

  public TaxCalculator withTaxRegional() {
    useRegional = true;
    return this;
  }

  public TaxCalculator withTaxGeneral() {
    useGeneral = true;
    return this;
  }

  public TaxCalculator withTaxSurcharge() {
    useSurcharge = true;
    return this;
  }

  public double calculate(Order order) {
    return calculate(order, useRegional, useGeneral, useSurcharge);
  }
}

// TaxCalculator는 지역 세금과 추가 요금은 주문에 추가하고 싶다는 점은 명확하게 보여줌
double value = new TaxCalculator().withTaxRegional()
                                  .withTaxSurcharge()
                                  .calculate(order);
```
- 이 방법은 **코드가 장황하다는게 문제**
- 도메인의 각 세금에 해당하는 **불리언 필드**가 필요하므로 **확작성**도 제한적
- 자바의 **함수형 기능**을 이용하면 더 간결하고 유연한 방식으로 같은 **가독성**을 달성할 수 있음

#### CODE.10.16. 유창하게 세금 함수를 적용하는 세금 계산기
```java
public class TaxCalculator {
  public DobubleUnaryOperator taxFunction = d -> d; // 주문값에 적용된 모든 세금을 계산하는 함수

  public TaxCaculator with(DoubleUnaryOperator f) {
    taxFunction = taxFunction.addThen(f); // 새로운 세금 계산 함수를 얻어서 인수로 전달된 함수와 현재 함수를 합침
    return this; // 유창하게 세금 함수가 연결될 수 있도록 결과를 반환
  }

  public double caculate(Order order) {
    return taxFunction.applyAsDouble(order.getValue()); // 주문의 총 합에 세금 계산 함수를 적용해 최종 주문값을 계산
  }
}
```
- 이 기법은 주문의 총 합에 적용할 함수 **한 개의 필드**만 필요로하며
  - `TaxCalculator` 클래스를 통해 모든 세금 설정 적용
- 이 함수의 시작값은 확인 함수
- 처음 시점에는 세금이 적용되지 않았으므로
  - 최종값은 총합과 같음
- `with()`메서드로 새 세금이 추가되면
  - 현재 세금 계산 함수에 이 세금이 조합되는 방식으로 한 함수에 모든 추가된 세금이 적용
- `calculate()`메서드에 전달하면 다양한 세금 설정의 결과로 만들어진 **세금 계산 함수**가 주문의 합계에 적용
  ```java
  double value = new TaxCalculator().with(Tax::regional)
                                    .with(Tax::surcharge)
                                    .calculate(order);
  ```
- 메서드 참조는 읽기 쉽고 코드를 간결하게 만듦
- 새로운 세금 함수를 `Tax` 클래스에 추가해도
  - **함수형** `TaxCalculator`로 바꾸지 않고 바로 사용할 수 있는 유연성