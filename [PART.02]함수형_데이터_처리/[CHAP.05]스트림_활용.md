# [CHAP.05] 스트림 활용
- 스트림 API가 지원하는 다양한 연산
  - `스트림 API`는 **내부 반복**뿐 아니라, 코드를 **병렬**로 실행할지 여부 결정 가능
  - 순차적인 반복을 **단일 스레드**로 구현하는 **외부 반복**으로는 구현 불가
- java 8, java 9에서 추가된 다양한 연산
  - 스트림 api가 지원하는 연산을 이용하여 **필터링, 슬라이싱, 매핑, 검색, 페이징, 리듀싱**등의 다양한 데이터 처리 질의 가능
  - 숫자 스트림, 파일과 배열 등 다양한 소스로 스트림을 만드는 방법
  - 무한 스트림

## 5.1. 필터링

### 5.1.1. predicate로 필터링
- `stream` 인터페이스에서는 `filter`를 지원
- `filter`는 `predicate`(boolean return)을 인수로 받아
  - `predicate`와 일치하는 모든 요소를 포함하는 **스트림**을 반환
- 예시
  ```java
  List<Dish> vegetarianMenu = menu.stream()
                              .filter(Dish::isVegetarian)
                              .collect(toList());
  ```

#### 5.1.2. 고유 요소 필터링
- `distinct` 메서드 활용
  - 고유 요소로 이루어진 **스트림**을 반환
  - 고유 여부는 스트림에서 만든 객체의 `hashCode`, `equals`로 판단
- 예시 코드
  ```java
  List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
  numbers.stream()
          .filter(i -> i % 2 == 0)
          .distinct()
          .forEach(System.out::println);
  ```

## 5.2. 스트림 슬라이싱(java 9)
- 스트림의 요소를 선택하거나, 스킵하는 다양한 방법
  - `predicate`를 이용하는 방법
  - 스트림의 첫 몇 요소를 무시하는 방법
  - 특정 크기로 스트림을 줄이는 방법 등

### 5.2.1. predicate를 이용한 슬라이싱
- `java 9`에서는 **스트림 요소**를 효과적으로 선택할 수 있도록
  - `takeWhile, dropWhile` 두 가지 새로운 메서드를 지원

#### TAKEWHILE 활용
- 예시 : `요리중, 320 kcal 이하 선택`
  ```java
  List<Dish> filteredMenu = specialMenu.stream()
                                        .filter(dish -> dish.getCalories() < 320)
                                        .collect(toList());
  ```
- 리스트 자체가 이미 칼로리 순으로 정렬되어 있음
  - 순서가 존재할때, `320`이라는 기준에 일치한다면, **반복 작업 중단**이 가능함
  - 큰 리스트에서는 상당한 차이가 될 수 있음
- `takeWhile`
  - 무한 스트림을 포함한 모든 스트림에 `predicate`를 적용하여 스트림을 슬라이스 할 수 있음
  ```java
  List<Dish> sliceMenu1
        = specialMenu.stream()
                      .takeWhile(dish -> dish.getCalories() < 320)
                      .collect(toList());
  ```

#### DROPWHILE 활용
- 나머지 요소를 선택하라면?
  - 위 예시에서 `320 kcal`보다 큰 요소를 선택하는 경우
  - `dropWhile`을 사용하여 처리
- 예시
  ```java
  List<Dish> sliceMenu2
      = specialMenu.stream()
                    .dropWhile(dish -> dish.getCalories() < 320)
                    .collect(toList());
  ``` 
- `dropWhile`과 `takeWhile`은 정반대의 작업 수행
- `dropWhile`은 `predicate`가 거짓이 되면, 그 지점에서 작업을 중단하고 남은 모든 요소를 반환
- `dropWhile`은 무한한 남은 요소를 가진 **무한 스트림**에서도 동작

### 5.2.2. 스트림 축소
- `limit(n)`
  - 주어진 값 이하의 크기를 갖는 **새로운 스트림 반환**
- 스트림이 정렬되어 있으면, 최대 `n`개 요소를 반환할 수 있음
- 예시 : `300 kcal`이상의 세 요리를 선택하여 리스트 반환
  ```java
  List<Dish> dishes = specialMenu.stream()
                                  .filter(dish -> dish.getCalories() > 300)
                                  .limit(3)
                                  .collect(toList());
  ```
- 정렬되지 않은 스트림(`src가 set`일 경우)에도 `limit`은 사용 가능
  - 단, 정렬되지 않은 상태로 반환

### 5.2.3. 요소 건너 뛰기
- `skip(n)`
  - 처음 `n`개의 요소를 제외한 스트림을 반환
- `n`개 이하의 요소를 포함하는 스트림에 `skip(n)`을 호출하면
  - **빈 스트림**이 반환됨
- `limit(n)`과 `skip(n)`은 상호 보완적인 연산 수행
- 예시
  ```java
  List<Dish> dishes = menu.stream()
                          .filter(d -> d.getCalories() > 300)
                          .skip(2)
                          .collect(toList());
  ```

## 5.3. 매핑
- 특정 객체에서 특정 데이터를 선택하는 작업
  - 데이터 처리 과정에서 자주 수행되는 연산
  - SQL 테이블에서 특정 열만 선택할 수 있음
  - `map`과 `flapMap`메서드가 그 역할을 함

### 5.3.1. 스트림의 각 요소에 함수 적용하기
- 스트림은 **함수**를 인수로 받는 `map` 메서드 지원
  - 각 요소에 적용되며, 함수를 적용한 결과가 새로운 요소로 매핑됨
  - translation에 가까운 `mapping`이라는 표현
- 예시
  ```java
  List<Integer> dishNames = menu.stream()
                               .map(Dish::getName)
                               .map(String::length)
                               .collect(toList());
  ```
  - 출력 스트림은 `Stream<Integer>` 형식을 가짐
  - `map`간의 체인 연산도 가능

### 5.3.2. 스트림 평면화
- 리스트의 데이터를 각 요소로 분해하는 잘못된 예시
  ```java
  String[] words = ["Hello", "World"];
  // 문자단위로 차원이 내려가지 않아, 원하는대로 동작하지 않음
  words.stream()
       .map(word -> word.split(""))
       .distinct()
       .collect(toList());
  ```

#### Map과 Arrays.stream의 활용
- 배열 스트림 대신 **문자열 스트림**이 필요한 상황
  - 문자열을 받아 스트림을 만드는 `Arrays.stream()`메서드 존재
    ```java
    String[] arrayOfWords = ["GoodBye", "World"];
    Stream<String> streamOfwords = Arrays.sstream(arrayOfWords);

    // 위 파이프라인에 `Arrays.stream()`메서드 적용
    words.stream()
         .map(word -> word.split("")) // 각 단어를 개별 문자열 배열로 반환
         .map(Arrays::stream) // 각 배열을 별도의 스트림으로 생성
         .distinct()
         .collect(toList());
    ```
    - 단, 위 예시에서는 `List<Strean<String>>`이 생성됨
- 문제를 해결하려면
  - 각 단어를 **개별 문자열**로 이루어진 배열로 만든 다음에
  - 각 배열을 별도의 **스트림**으로 만들어야 함

#### flapMap 사용
- 코드
  ```java
  List<String> uniqueCharacters = 
      words.stream()
           .map(word -> word.split("")) // 각 단어를 개별 문자를 포함하는 배열로 반환
           .flatMap(Arrays::stream) // 생성된 스트림을 하나의 스트림으로 평면화
           .distinct()
           .collect(toList());
  ```
- `flatMap`은 각 배열을
  - **스트림**이 아니라 **스트림의 콘텐츠**로 매핑
  - 하나의 **평면화된 스트림**을 반환

## 5.4. 검색과 매칭
- 특정 속성이 **데이터 집합에 있는지 여부**를 검색하는 데이터 처리도 자주 사용
- `allMatch, anyMatch, noneMatch, findFirst, findAny`등의 다양한 **유틸리티 메서드** 제공


### 5.4.1. 프레디케이트가 적어도 한 요소와 일치하는지 확인
- `predicate`가 주어진 스트림에서 **적어도 한 요소와 일치**하는지 확인할 때
  - `anyMatch`를 사용
- 예시
  ```java
  // menu에 채식요리가 있는지 확인 
  if(menu.stream().anyMatch(Dish::isVegetarian)) {
    System.out.println("The menu is (somewhat) vegetarian friendly!!");
  }
  ```

### 5.4.2. 프레디케이트가 모든 요소와 일치하는지 검사
- 코드
  ```java
  boolean isHealty = menu.stream().allMatch(dish -> dish.getCalories() < 1000);
  ```

#### NoneMatch
- `allMatch`와 반대연산 수행
- 주어진 프레디케이트와 일치하는 요소가 **없는지** 확인
- 코드
  ```java
  boolean isHealty = menu.stream().allMatch(dish -> dish.getCalories() < 1000);
  ```
- `anyMatch, allMatch, noneMatch` 세 메서드는
  - 스트림 **쇼트서킷** 기법
  - 즉 자바의 `&&`와 `||`와 같은 연산을 활용

#### 쇼트서킷 평가
- 전체 스트림을 처리하지 않았더라도 결과를 반환할 수 있음
- 전체 `boolean`식이 `and`로 연접되어 있을 때, 하나라도 `false`가 나오면 거짓
- `allMatch, noneMatch, findFirst, findAny`등의 연산은
  - 모든 스트림의 요소를 처리하지 않고도, 결과 반환 가능
- 마찬가지로, 스트림의 모든 요소를 처리할 필요 없이
  - 주어진 크기의 스트림을 생성하는 `limit`도 쇼트서킷 연산
- **무한한 요소를 가진 스트림**을
  - **유한한 크기**로 줄일 수 있는 유용한 연산

### 5.4.3. 요소 검색
- `findAny` 메서드, 현재 스트림에서 임의의 요소 반환
  ```java
  Optional<Dish> dish = menu.stream().filter(Dish::isVegeterian).findAny();
  ```
  - 내부적으로 단일 과정을 실행할 수 있도록 최적화
  - **쇼트서킷**을 이용하여, 결과를 찾는 즉시 종료

#### Optional이란?
- 값의 **존재**나 **부재**를 표현하는 컨테이너 클래스
- `findAny`의 경우 아무 요소도 반환할 수 있는데,
  - 이는 `null`이 반환될 경우 쉽게 에러 발생 가능성 존재
- 지원하는 메서드
  - `isPresent()`
    - `Optional`이 값을 포함하면 `true`, 아닐경우 `false`
  - `ifPresent(Consumer<T> block)`
    - 값이 있으면 **주어진 블록** 실행
    - `Consumer` 함수형 인터페이스를 통하여
      - `T`형식의 인수를 받아 `void`를 반환
  - `T get()`
    - 값이 존재하면 반환, 없을 경우 `NoSuchElementException` 발생
  - `T orElse(T other)`
    - 값이 있으면 값을 반환, 없으면 기본값 반환
- 사용 예시
  ```java
  menu.stream().filter(Dish::isVegeterian)
               .findAny() // Optional<Dish> 반환
               .ifPresent(dish -> System.out.println(dish.getName()));
  ```

### 5.4.4. 첫번째 요소 찾기
- 리스트 또는 정렬된 연속 데이터부터 생성된 스트림 처럼
  - 일부 스트림에는 **논리적인 아이템 순서**가 정해져 있을 수 있음
- 이런 스트림에서 첫 요소를 찾으려면?
- 예시
  ```java
  // 숫자 리스트에서 3으로 나누어 떨어지는 첫번째 제곱값을 반환하는 예시
  List<Integer> someNumbers = Arrays.asList(1,2,3,4,5);
  Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
                                                             .map(n -> n*n)
                                                             .filter(n -> n % 3 == 0)
                                                             .findFirst();
  ```

#### findFirst와 findAny의 사용처
- **병렬 실행**에서는 첫번째 요소를 찾기 어려움
- 반환 순서가 상관이 없다면, **병렬 스트림**에서는 `findAny`를 사용할 것

## 5. 리듀싱
- 최종 연산으로 `boolean`(allMatch), `void`(forEach), `Optional`(findAny)를 반환
- 또한 `collect`로 모든 스트림의 요소를 리스트로 모으는 방법도 존재
- `reduce`연산
  - `Integer`와 같은 결과가 나올때까지, 스트림의 모든 요소를 반복적으로 처리
  - 스트림 요소를 처리해서 **값으로 도출**
  - `fold`라는 의미로도 부름

### 5.5.1 요소의 합
- 반복된 패턴을 추상화하는 코드
  ```java
  int sum = numbers.stream().reduce(0, (a, b) -> a + b);
  ```
  - 초기값 `0`
  - 두 요소를 조합하여 새로운 값을 만드는 `BinaryOperator<T>` 사용
  ```java
  int product = numbers.stream().reduce(1, (a, b) -> a * b);
  ```
- 계속하여 누적값(`accumulated value`)를 만들어감
- 메서드 참조 예시
  ```java
  int sum = numbers.stream().reduce(0, Integer::sum);
  ```

#### 초기값 없음
- 초기값이 없는 `reduce`도 존재하나, `Optional`을 반환
  ```java
  Optional<Integer> sum = numbers.stream().reduce((a,b) -> (a+b));
  ```
  - 만약 스트림에 아무 요소도 없을 때, 반환할 값이 없기 때문

### 5.5.2 최댓값과 최소값
- `reduce`의 두가지 인자
  - 초깃값
  - 스트림의 두 요소를 합쳐 하나의 값으로 만들때 사용하는 람다
- 예시
  ```java
  Optional<Integer> max = numbers.stream().reduce(Integer::max);
  Optional<Integer> min = numbers.stream().reduce(Integer::min);
  // Integer::min 대신 (x,y) -> x<y?x:y 도 ㅅ ㅏ용 가능
  ```

#### reduce 메서드의 장점과 병렬화
- 단계적 반복으로의 합계와 `reduce`를 이용한 합계의 차이
- `reduce`를 사용하게 되면
  - **내부 반복**이 추상화 되면서
  - 내부 구현에서 **병렬**로 `reduce`를 실행할 수 있음
- 반복적인 합계에서는 `sum`을 사용해야하기 때문에, **쉽게 병렬화 하기 어려움**
  - 강제적으로 동기화시키더라도,
  - 결국 병렬화로 얻어야할 이득이
    - 스레드 간의 소모적인 경쟁 때문에 상쇄
- 해당 작업을 **병렬화**하려면
  - 입력을 분할하여, 불할된 입력을 더한 후, 더한 값을 합쳐야 함
- `fork/join framework`를 활용하면 가능함
- 병렬화한 코드
  ```java
  int sum = numbers.parallelStream().reduce(0, Integer::sum);
  ```

#### 스트림 연산 : 상태 없음과 상태 있음
- 스트림에서 `stream`메서드를
  - `parallelStream`으로 바꾸는 것만으로 **병렬성**확보 가능
- 각각의 연산은 **내부적인 상태**를 고려해야 함
- `map, filter` 등은
  - 입력 스트림에서 각 요소를 받아 `0`또는 **결과**를 **출력 스트림**으로 보낸다
  - 따라서 보통 **상태가 없는** 연산, `stateless operation`
- `reduce, sum, max`연산은 결과를 누적할 **내부 상태**가 필요
  - 내부 상태의 크기는, 요소 수와 관계 없이 한정적(`bounded`)
- 반면 `sorted`와 `distinct` 같은 연산은
  - `filter, map`처럼 스트림을 입력으로 받아, 다른 스트림을 출력하는 것처럼 보이지만, 아님
  - 요소를 **정렬**하거나 **중복 제거**하려면, **과거 이력**을 알고 있어야 함
  - 예시로 어떤 요소를 출력 스트림으로 추가시
    - 모든 요소가 **버퍼에 추가되어 있어야 함**
  - 연산을 수행하는데 필요한 **저장소 크기**는 정해져 있지 않음
  - 따라서, 데이터 스트림의 크기가 크거나, 무한이라면
    - **문제 발생 가능**
  - 이러한 연산을 **내부 상태를 갖는 연산**(`stateful operation`)이라 함

## 5.6. 실전 연습
- 트랜잭션(거래)를 실행하는 거래자 예제

### 5.6.1. 거래자와 트랜잭션
- `Trader` 리스트와 `Transaction` 리스트를 이용
  ```java
  Trader raoul = new Trader("Raoul", "Cambridge");
  ...
  List<Transaction> transactions = Arrays.asList(
    new Transaction(brian, 2011, 300),
    ...
  );

  public class Transaction {
    private final Trader trader;
    private final int year;
    private final int value;

    ...
  }
  ```

### 5.6.2. 실전 연습 정답

#### CODE.5.1. 2011년에 일어난 모든 트랜잭션을 찾아 오름차순 정렬
```java
List<Transaction> tr2011 = transactions.stream()
                                      .filter(transaction -> transaction.getYear() == 2011)
                                      .sorted(comparing(Transaction::getValue))
                                      .collect(toList());
```

#### CODE.5.2. 거래자가 근무하는 모든 도시를 중복 없이 나열
```java
List<String> cities = transactions.stream()
                                  .map(transaction -> transaction.getTrader().getCity())
                                  .distinct()
                                  .collect(toList());
```
- `distinct`외에 스트림을 **집합**으로 변환하는 `toSet()`을 사용할 수 있음
```java
Set<String> cities = transactions.stream()
                                  .map(transaction -> transaction.getTrader().getCity())
                                  .collect(toSet());
```

#### CODE.5.3. 케임브리지에서 근무하는 모든 거래자를 찾아 이름순 정렬
```java
List<Trader> traders = transactions.stream()
                                    .map(Transaction::getTrader)
                                    .filter(trader -> trader.getCity().equals("Cambridge"))
                                    .distinct()
                                    .sorted(comparing(Trader::getName))
                                    .collect(toList());
```

#### CODE.5.4. 모든 거래자의 이름을 알파벳순으로 정렬
```java
String traderStr = transactions.stream()
                                .map(transaction -> transaction.getTrader().getName())
                                .distinct()
                                .sorted() // 중복된 이름 제거
                                .reduce("", (n1, n2) -> n1 + n2);
```
- 각 반복과정에서, 모든 문자열을 **반복적으로 연결**하여 새로운 문자열 객체를 만듦
  - 효율성이 떨어짐
- `joining`을 활용하여 효율적으로 처리 가능
  - 내부적으로 `StringBuilder`를 이용
```java
String traderStr = transactions.stream()
                                .map(transaction -> transaction.getTrader().getName())
                                .distinct()
                                .sorted()
                                .collect(joining())
```

#### CODE.5.5. 밀라노에 거래자가 있는지
```java
boolean milanBased = transactions.stream()
                                  .anyMatch(transaction -> transaction.getTrader().getCity().equals("Milan"));
```

##### CODE.5.6. 케임브리지에 거주하는 거래자의 모든 트랜잭션 값을 출력하시오
```java
transactions.stream()
            .filter(t -> "Cambridge".equals(t.getTrader().getCity()))
            .map(Transaction::getValue)
            .forEach(System.out::println)
```

#### CODE.5.7. 전체 트랜잭션 중 최댓값은 얼마인가
```java
Optional<Integer> highestValue =
  transactions.stream()
              .map(Transaction::getValue)
              .reduce(Integer::max);
```

#### CODE.5.8. 전체 트랜잭션 중 최솟값은 얼마인가
```java
Optional<Transaction> smallestTransaction = 
                          transactions.stream().reduce((t1, t2) -> t1.getValue() < t2.getValue() ? t1 : t2);
```
- 스트림은 **최댓값**이나 **최솟값**을 계산하는 데 사용할 **키**를 지정하는 `Comparator`를 인수로 받는
  - `min`, `max` 메서드를 제공
    ```java
    Optional<Transaction> smallestTransaction =
                                        transactions.stream().min(comparing(Transaction::getValue));
    ```

## 5.7. 숫자형 스트림
- 믹싱 비용이 포함된 코드
  ```java
  int calories = menu.stream().map(Dish::getCalories).reduce(0, Integer::sum);
  ```
- `map()` 메서드가 `Stream<T>`를 생성하기 때문에, `sum()`을 직접 호출할 수 없음
  - 스트림의 요소 형식은 `Integer`지만, 인터페이스에는 `sum`메서드가 없음
- 스트림 API에는 **숫자 스트림**을 효율적으로 처리할 수 있도록 **기본형 특화 스트림**(primitive stream sspecialization)을 제공

### 5.7.1. 기본형 특화 스트림
- `java 8`에는 세 가지 기본형 특화 스트림을 제공
  - `IntStream`, `DoubleStream`, `LongStream`
- 스트림 API는 믹싱 비용을 피할 수 있음
- 각 인터페이스에는, `sum`, `max`, 숫자 관련 **리듀싱 연산 수행 메서드**를 제공
- 필요시 다시 **객체 스트림**으로 복원하는 기능도 제공
- 특화 스트림은 오직
  - **박싱 과정**에서 일어나는 **효율성**과 연관 됨
  - 스트림에 추가적인 기능을 제공하진 않음

#### 숫자 스트림으로 매핑
- `스트림 -> 특화 스트림`으로 변경 시, `mapToInt`, `mapToDouble`, `mapToLong` 세 가지 메서드를 많이 사용
- `map`과 정확히 같은 기능을 수행하나 `Stream<T>`대신 특화된 스트림 반환
  ```java
  int calories = menu.stream() // Stream<Dish>
                      .mapToInt(Dish::getCalories) // IntStream
                      .sum();
  ```
- `mapToInt` 메서드는
  - 각 요리에서 **모든 칼로리**를 추출한 다음
  - `IntStream`을 반환(`Stream<Integer>` 가 아님)
- `IntStream` 인터페이스에서 제공하는 `sum` 메서드를 이용해서, 칼로리 합계를 계산할 수 있음
- 스트림이 비어있을 경우 `sum`은 기본값 `0`을 반환
- `IntStream`은 `min`, `max`, `average` 등 다양한 유틸리티 메서드 제공

#### 객체 스트림으로 복원하기
- `숫자 스트림 -> 특화되지 않은 스트림`
- `IntStream::map`의 경우
  - `int`인수를 받아 `int`를 반환하는 `IntUnaryOperator`를 인수로 받ㄱ음
  - 정수가 아닌 `Dish`와 같은 다른 값을 변환하려면
    - **스트림 인터페이스**에 정의된 **일반적인 연산**을 사용해야 함
    - `boxed` 사용
- 예시
  ```java
  IntStream intStream = menu.stream().mapToInt(Dish::getCalories); // 스트림을 숫자 스트림으로 변환
  Stream<Integer> stream = intStream.boxed(); // 숫자 스트림 -> 스트림 변환
  ```

#### 기본값 : OptionalInt
- 합계 예제에서는 `0`이라는 기본값 존재
- `IntStream`에서 **최댓값**을 찾을 때는 `0`이라는 기본값 때문에
  - 잘못된 값이 도출될 수 있음
- 스트림에 **요소가 없는** 상황과, 실제 최댓값이 `0`인 상황을 구분하려면
  - `Optional` 연관 참조 내역을 활용
- `Optional`을 `Integer, String`등의 참조형식으로 **파라미터화**
- `OptionalInt, OptionalDouble, OptionalLong`의 세 가지 **기본형 특화 스트림 버전**을 제공
- 예시 : `OptionalInt`를 사용하여 `IntStream`의 최댓값 요소 찾기
  ```java
  OptionalInt maxCalories = menu.stream().mapToInt(Dish::getCalories).max();
  int max = maxCalories.orElse(1); // 값이 없을 때, 기본 최댓값을 명시적으로 설정
  ```

### 5.7.2. 숫자 스트림 활용 : 피타고라스 수
- 피타 고라스 수 스트림
  - `a^2 + b^2 = c^2`를 만족하는 세수의 스트림 찾기
- 세 수를 표현할 클래스가 아닌, **세 요소를 갖는 int배열**을 사용하기

#### 좋은 필터링 조합
```java
Math.sqrt(a*a + b*b) % 1 == 0;
```
- 만약 `x`가 부동 소숫점 수라면, `x%1.0`이라는 코드로 소숫점 이하 부분을 얻을 수 있음
- `filter`를 활용하는 방법
  ```java
  filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
  ```
  - `a`라는 값이 주어지고, `b`는 스트림이라고 가정할 때,
    - `filter`로 `a`와 함께 피타고라스 수를 구성하는 모든 `b`를 필터링 할 수 있음

#### 집합 생성
- 각 요소를 피타고라스 수로 변환
  ```java
  stream.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
  .map(b -> new int[]{a,b, (int) Math.sqrt(a*a + b*b)});
  ```

#### b값 생성
- `Stream.rangeClosed`로 주어진 범위의 수를 만들 수 있음
- `1 ~ 100`까지의 `b`값 생성
  ```java
  IntStream.rangeClosed(1, 100)
            .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
            .boxed()
            .map(b -> new int[]{a, b, (int) Math.sqrt(a*a + b*b)});
  ```
- `filter` 이후 `rangeClosed`가 반환한 `IntStream`을
  - `boxed`를 이용해서 `Stream<Integer>`로 복원
- `map`은 스트림의 각 요소를 `int 배열`로 반환하기 때문
- 스트림의 각 요소로 `int`가 반환될 것을 기대하나, 우리가 원하는 연산이 아님
- 개체값 스트림을 반환하는 `IntStream`의 `mapToObj` 메서드를 이용하여 코드 재구현
  ```java
  IntStream.rangeClosed(1, 100)
  .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
  .mapToObj(b -> new int[]{a, b, (int) Math.sqrt(a*a + b*b)})
  ```

#### a값 생성
- a값을 추가하여 최종 완성 코드를 작성
  ```java
  Stream<int[]> pythagoreanTriples = 
                      IntStream.rangeClosed(1, 100).boxed()
                                .flatMap(a -> 
                                  IntStream.rangeClosed(a, 100)
                                            .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
                                            .mapToObj(b -> new int[]{a, b, (int)Math.sqrt(a*a + b*b)})
                                        );
  ```

#### 코드 실행
- `limit`을 포함하여, 얼마나 많은 세 수를 포함하는 스트림을 만들지 결정
```java
pythagoreanTriples.limit(5)
                  .forEach(t -> System.out.println(t[0] + ", " + t[1] + ", " + t[2]));
```

#### 개선할 점
- 제곱근을 두번 계산함
  - `(a*a, b*b, a*a+b*b)` 형식을 만족하는 세 수를 만든 다음에
  - 우리가 원하는 조건에 맞게 필터링 하는 것이 더 최적화된 방법
- 개선 코드
  ```java
  Stream<double[]> pythoagoreanTriples2 =
                      IntStream.rangeClosed(1, 100).boxed()
                                .flatMap(a -> 
                                  IntStream.rangeClosed(a, 100)
                                  .mapObj(b -> new double[]{a, b, Math.sqrt(a*a + b*b)})
                                  .filter(t -> t[2] % 1 == 0)
                                );
  ```

## 5.8. 스트림 만들기

### 5.8.1. 값으로 스트림 만들기
- 임의의 수를 인수로 받는 정적 메서드 `Stream.of`를 이용해 스트림을 만들 수 있음
- `Stream.of`로 문자열 스트림을 만드는 예시
  ```java
  Stream<String> stream = Stream.of("Modern ", "Java ", "In ", "Action");
  stream.map(String::toUpperCase).forEach(System.out::println);
  ```
- `empty`메서드를 이용하여 스트림을 비우기
  ```java
  Stream<String> emptyStream = Stream.empty();
  ```

### 5.8.2. null이 될 수 있는 객체로 스트림 만들기
- `java 9`에서는 `null`이 될 수 있는 개체를 스트림으로 만들 수 있는 **새로운 메서드** 추가
- 때로는 `null`이 될 수 있는 객체를 **스트림**으로 만들어야 할 수 있음
  - 객체가 `null`이라면 빈 스트림
- `System.getProperty` : 제공 키에 대응되는 속성이 없으면 `null`을 반환
  ```java
  String homeValue = Sytem.getProperty("home");
  Stream<String> homeValueStream = homeValue == null ? Stream.empty() : Stream.of(value);

  // Stream.ofNullable을 활용
  Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));

  // null이 될 수 있는 객체를 포함하는 스트림 값을 `flapMap`과 함께 사용하는 상황에서 더 유용함
  Stream<String> values = 
                Stream.of("config", "home", "user")
                      .flatMap(key -> Stream.ofNullable(Sytem.getProperty(key)));
  ```

### 5.8.3. 배열로 스트림 만들기
- `Arrays.stream`을 이용
  ```java
  int[] numbers = {2, 3, 5, 7, 11, 13};
  int sum = Arrays.stream(numbers).sum(); // 41
  ```

### 5.8.4. 파일로 스트림 만들기
- File I/O 연산에 수행하는 자바의 `NIO API`(Non-blocking I/O)도 `Stream API`를 활용하도록 업데이트
- `java.nio.file.Files`의 많은 **정적 메서드**가 스트림을 반환한다
- `Files.lines`
  - 주어진 행 스트림을 문자열로 반환
- 파일에서 고유한 단어수를 찾는 프로그램
  ```java
  long uniqueWords = 0;
  // stream은 자원을 자동으로 해제할 수 있는 `AutoCloseable`, try-finally가 필요 없음
  try(Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
    uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                        .distinct()
                        .count();
  }
  ```
- `Files.lines` 파일의 각 행요소 반환, 스트림
- `Stream` 인터페이스는 `AutoCloseable` 인터페이스 구현
- `try` 블록 내의 자원은 자동으로 관리

### 5.8.5. 함수로 무한 스트림 만들기
- 함수에서 스트림을 만들 수 있는 두 정적 메서드
  - `Stream.iterate`
  - `Stream.generate`
- 두 연산을 이용하여, **infinite stream**을 생성 가능
  - 크기가 고정되지 않은 스트림
- `iterate`와 `generate`는 요청할 때마다, 주어진 함수를 이용하여 값을 생성
- 무제한으로 값 생성은 가능하나, 보통 무한한 값을 출력하지 않도록 `limit(n)`함수를 함께 연결하여 사용

#### iterate 메서드
- 코드
  ```java
  Stream.iterate(0, n -> n+2)
        .limit(10)
        .forEach(System.out::println);
  ```
  - 초깃값(예제, `0`)과 람다(예제, `UnaryOperator<T>`)를 인수로 받아, 끊임없이 값 생산
- 무한 스트림을 생성함
- **Unbounded Stream**으로 표현
- 최종 연산인 `forEach`를 호출하여, 스트림을 소비하고, 개별 요소 출력

#### java 9의 iterate
- java 9에서는 `predicate`를 지원
- `0`에서 시작하여 `100`보다 크면 숫자 생성을 중단하는 코드
  ```java
  IntStream.iterate(0, n -> n < 100, n -> n + 4)
            .forEach(System.out::println);
  ```
- `iterate` 메소드는 두 번째 인수로 `predicate`를 받아, 언제까지 작업을 수행할 것인지를 기준으로 사용
  - `filter` 동작으로도 같은 결과를 얻을 수 있다고 생각할 수 있으나..
    ```java
    IntStream.iterate(0, n -> n + 4)
              .filter(n -> n < 100)
              .forEach(System.out::println);
    ```
    - 위와 같은 방법으로는 불가, 종료되지 않음
    - `filter`의 경우, 이 작업을 중단해야하는지 알 수 없기 때문
- `stream shortCircuit`을 지원하는 `takeWhile`을 이용해야 함
  ```java
  IntStream.iterate(0, n -> n + 4)
            .takeWhile(n -> n < 100)
            .forEach(System.out::println);
  ```

#### generate 메서드
- 요구할 때 값을 계산하는 무한 스트림 생성 가능
- `iterate`와 달리 생산된 각 값을 **연속적으로 계산하지 않음**
- `Supplier<T>`를 인수로 받아, 새로운 값을 생성
- 예시
  ```java
  Stream.generate(Math::random)
        .limit(5)
        .forEach(System.out::println);
  ```
- 사용처
  - `supplier`는 상태가 없는 메서드
    - 나중에 계산할 어떤 값도 저장하지 않음
  - 하지만 `supplier`에 꼭 상태가 없어야하는 것은 아님
  - `supplier`가 상태를 저장한 이후, 다음에 스트림의 다음 값을 만들 때, 해당 상태를 고칠 수 있음
- 병렬 코드에 `supplier`가 있으면 안전하지 않음
  - 예시로는 존재하나, 실제로는 피해야 함
- 예시 코드 : 피보나치 수열
  ```java
  // 박싱 타입을 사용하여 `IntSupplier`를 인수로 받음
  IntStream ones = IntStream.generate(() -> 1);

  // IntSupplier에 정의된 getAsInt를 구현하는 객체를 명시적으로 전달 가능
  IntStream twos = IntStream.generate(new IntSupplier() {
    public int getAsInt() {
      return 2;
    }
  });

  // generate는 supplier를 이용하여 2를 반환하는 `getAsInt` 메서드를 반복적으로 호출
  // 여기서의 익명 클래스는, 람다와 비슷한 연산을 수행하나, 
  // 익명 클래스에는 `getAsInt` 메서드의 연산 커스텀 할 수 있는 상태 필드를 정의할 수 있음
  // 부작용이 발생할 수 있음(상태를 가지지 않는 람다는 부작용이 없음)

  // 기존 수열 상태 저장, getAsInt로 다음 요소를 계산하도록 IntSupplier를 만들어야 함
  // 다음에 호출될 때는 IntSupplier의 상태 갱신 필요
  // 피보나치 요소를 반환하도록 IntSupplier 구현
  IntSupplier fib = new IntSupplier() {
    private int previous = 0;
    private int current = 1;
    public int getAsInt() {
      int oldPrevious = this.previous;
      int nextValue = this.previous + this.current;
      this.previous = this.current;
      this.current = nextValue;
      return oldPrevious;
    }
  };
  IntStream.generate(fib).limit(10).forEach(System.out::println);
  ```
- 위 코드에서는 `IntSupplier` 인스턴스를 생성
  - 만들어진 객체는 `기존 피보나치 요소`와, `두 인스턴스 변수에 어떤 요소`인지 추적하므로 `mutable`상태
  - `getAsInt`를 호출하면, 객체 상태가 바뀌며 새로운 값을 생산
  - `iterate` 사용시, 각 과정에서 새로운 값을 생성
    - 기존 상태를 **변경하지 않음**, `immutable`
- 스트림을 **병렬**로 처리하면서, 올바른 결과를 얻으려면 **불변 상태 기법**을 고수해야 함
- 위 예시에서 `limit`을 이용하여 명시적으로 **스트림 크기 제한** 필요
  - 그렇지 않을경우, 최종 연산(e.g. `forEach`) 수행시, 아무런 결과도 리턴되지 않음
  - 무한 스트림의 요소는 **무한적**으로 계산이 반복되므로
    - **정렬 및 리듀스가 불가**

## 5.9. 마치며
- **스트림 API**를 이용하면 **복잡한 데이터 처리 질의 표현 가능**
- `filter, distinct, takeWhile(java9), dropWhile(java9), skip, limit` 메서드로, 스트림을 필터링 하거나 자를 수 있음
- 소스가 **정렬**되었다는 사실을 알고 있을 때, `takeWhile, dropWhile` 메서드를 효과적으로 사용할 것
- `map, flatMap` 메서드로 **스트림의 요소 추출/변환** 가능
- `findFirst, findAny` 메서드로, 스트림의 요소 **검색** 가능
  - `allMatch, noneMatch, anyMatch` 메서드를 이용하여
  - 주어진 `predicate`와 일치하는 요소를 스트림에서 **검색** 가능
