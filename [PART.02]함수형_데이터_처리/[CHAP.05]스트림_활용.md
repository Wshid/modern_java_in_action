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