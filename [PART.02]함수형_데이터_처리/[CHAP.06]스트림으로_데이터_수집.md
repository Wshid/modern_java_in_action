# [CHAP.06] 스트림으로 데이터 수집
- 다양한 요소 누적 방식은 `Collector` 인터페이스에 정의
- `collect`와 컬렉터로 구현할 수 있는 질의 예시
  - 통화별로 트랜잭션을 그룹화한 다음, 해당 통화로 일어난 모든 트랜잭션의 합 구하기
  - 트랜잭션을 비싼 트랜잭션과 저렴한 트랜잭션으로 구분하기
  - 트랜잭션을 도시 등 다 수준으로 그룹화
    - 그리고 각 트랜잭션이 비싼지 저렴한지 구분
- `Stream.toList`를 사용하는게 아닌, 
  - 범용적 컬렉터 파라미터를 `collect` 메서드에 전달하여 해결
  - 예시
    ```java
    Map<Currenty, List<Transaction>> transactionsByCurrencies = 
        transactions.stream().collect(groupingBy(Transaction::getCurrency));
    ```

## 6.1. 컬렉터란 무엇인가?
- `Collector` 인터페이스 구현
  - 스트림의 요소를 어떤식으로 도출할지 지정
- `groupingBy`를 활용하여
  - 각 키(통화) bucket 그리고 각 키 버킷에 대응하는 요소 리스트를
  - 값으로 포함하는 `Map`을 만드는 동작 수행
- multilevel으로 그룹화를 수행할 때
  - 명령형 코드에서는 `loop`와 `if`문을 사용한 처리 진행
  - **fp에서는 필요한 컬렉터를 쉽게 추가 가능**

### 6.1.1. 고급 리듀싱 기능을 수행하는 컬렉터
- 훌륭하게 설계된 함수형 API의 또다른 장점
  - 높은 수준의 **조합성**과 **재사용성**
- `collect`로 결과를 수집하는 과정을 **간단**하면서도 **유연한 방식**으로 정의할 수 있음
- 스트림에 `collect`를 호출하면
  - 스트림의 요소에 **컬렉터로 파라미터화된** 리듀싱 연산이 수행됨
- `collect`에서는 리듀싱 연산을 수행하여, 스트림의 각 요소를 방문하면서 컬렉터가 작업 처리 
- 보통 함수를 요소로 변환 할때는 **컬렉터**를 적용하며
  - 최종 결과를 저장하는 자료 구조에 값을 누적
- `Collectors` 유틸리티 클래스는
  - 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 **정적 팩터리 메서드** 제공
  - `toList`
    ```java
    List<Transaction> transactions = transactionSream.collect(Collectors.toList());
    ```

### 6.1.2. 미리 정의된 컬렉터
- `groupingBy` 같이 `Collectors` 클래스에서 제공하는 **팩토리 메서드**의 기능 설명
- `Collectors`에서 제공하는 메서드의 기능(3)
  - 스트림 요소를하나의 값으로 리듀스하고 요약
  - 요소 그룹화
  - 요소 분할
- 리듀싱과 요약 관련 기능을 수행하는 컬렉터
  - 트랜잭션 리스트에서 **트랜잭션 총합**을 찾는 등의 다양한 계산을 수행할 때 이를 컬렉터를 유용하게 활용 가능
- 스트림 요소를 그룹화하는 방법
  - 다수준으로 그룹화 하거나, 각각의 결과 서브 그룹에 **리듀싱 연산** 적용 하도록
  - **다양한 컬렉터**를 조합하는 방법
- **분할**(partitionting)도 가능
  - `한 개의 인수`를 받아 `Boolean`을 반환하는 함수
  - `predicate`를 그룹화 함수로 사용

## 6.2. 리듀싱과 요약
- 컬렉터(`Stream.collect 메서드의 인수`)로
  - 스트림의 항목을 **컬렉션**으로 재구성 가능
- 컬렉터로 스트림의 모든 항목을 **하나의 결과**로 합칠 수 있음
- `counting` 팩토리 메서드
  ```java
  import static java.util.stream.Collectors.*;
  long howManyDishes = menu.stream().collect(Collectors.counting());

  // 아래 내용을 대체함
  long howManyDishes = menu.strea().count();
  ```
  - 다른 컬렉터와 함께 사용할 때 유용함

### 6.2.1. 스트림 값에서 최댓값과 최솟값 검색
- 예제 : 칼로리가 가장 높은 요리 찾기
- `Collectors.maxBy`, `Collectors.minBy` 두 개의 메서드를 이용
- 칼로리로 요리를 비교하는 `Comparator`를 구현한 다음 `Collectors.maxBy`로 전달
  ```java
  Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);

  Optional<Dish> mostCaloriesDish = menu.stream().collect(maxBy(dishCaloriesComparator));
  ```
  - `Optional<Dish>`의 역할
    - `menu`가 비어있는 경우

### 6.2.2. 요약 연산
- `summarization`
- 스트림에 있는 객체의 숫자 필드의 `합계`나 `평균`등을 반환
- `Collectors.summingInt` : 요약 팩토리 메서드
  - `summingInt`는 객체를 `Int`로 매핑하는 컬렉터 반환
  - `summingInt`가 `collect` 메서드로 전달되면, 요약 작업 수행
- 예시 : 메뉴 리스트의 총 칼로리를 계산하는 코드
  ```java
  int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
  
  // 평균 계산 코드
  int avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));
  ```
- `Collectors.summingLong`과 `Collectors.summingDouble` 메서드는 같은 방식으로 동작
- 평균값 계산 시,
  - `Collectors.averagingInt, averagingLong, averagingDouble` 등으로 다양한 형식의 숫자 집합 평균 계산 가능
- 두 개 이상의 연산을 한번에 수행하는 경우
  - 팩토리 메서드 `summarizingInt`가 반환하는 컬렉터 사용
- 예시 : 하나의 요약 연산으로, 메뉴에 있는 요소 수, 요리 칼로리 합계, 평균, 최댓값, 최솟값 등을 계산하는 코드
  ```java
  // 아래 코드 실행시, IntSummaryStatistics 클래스로 모든 정보가 수집됨
  IntSummaryStatistics menuStatistics = 
    menu.stream().collect(summarizingInt(Dish::getCalories));
  
  // 객체 출력시 다음과 같은 정보 확인 가능
  IntSummaryStatistics(count=9, sum=4300, min=120, average=477, max=800)
  ```
- `log, double`에 대응하는 클래스도 역시 존재

### 6.2.3. 문자열 연결
- `joining`을 이용하여
  - 각 객체에 `toString` 메서드를 호출하여, 모든 문자열을 하나로 연결
- 예시 : 메뉴의 모든 요리명을 연결하는 코드
  ```java
  String shortMenu = menu.stream().map(Dish::getName).collect(joining());

  // Dish::toString가 존재한다면
  String shortMenu = menu.stream().collect(joining());

  // 구분자 포함
  String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
  ```
- `joining`내부적으로 `StringBuilder`를 활용하여 문자열을 하나로 만듦

### 6.2.4. 범용 리듀싱 요약 연산
- 위 메서드들 모두 `reducing` 팩토리 메서드로 적용 가능
  - `Collectors.reducing`
- 예시 : 모든 칼로리 합계
  ```java
  int totalCalories = menu.stream().collect(reducing(
    0, Dish::getCalories, (i, j) -> i + j
  ));
  ```
- `reducing`의 인수 세가지
  - 첫 번째 인수 : 연산의 초기값 및 인수가 없을 때의 반환 값
  - 두 번째 인수 : 요리를 칼로리 정수로 변환할 때 사용하는 변환 함수
  - 세 번째 인수 : 같은 종류의 내용을 하나의 값으로 더하는 `BinaryOperator`
    - 예제에서는 두개의 `int`가 사용됨
- 한개의 인수를 가진 `reducing` 버전 활용
  ```java
  Optional<Dish> mostCaloriesDish =
    menu.stream().collect(
      reducing(
        (d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2
      )
    );
  ```
- 한개의 인수를 가질 경우,
  - 세 개의 인수를 갖는 `reducing` 메서드에서
    - 첫 번째 요소를 시작 요소
    - 두 번째 요소를 자신을 그대로 반환하는 항등 함수(`identity function`)
  - 시작값이 존재하지 않기 때문에, **빈 스트림 전달 시, 시작값이 설정되지 않는 상황**
  - 그렇기 때문에 `Optional<Dish>` 객체 반환

#### collect와 reduce
- `Stream::collect, reduce`의 차이?
- `toList` 컬렉터를 사용하는 `collect` 대신, `reduce`를 사용하기
  ```java
  Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();
  List<Integer> numbers = stream.reduce(
    new ArrayList<Integer>(),
    (List<Integer> l, Integer e) -> {
      l.add(e);
      return l; 
    },
    (List<Integer> l1, List<Integer> l2) -> {
      l1.addAll(l2);
      return l1;
    }
  );
  ```
- `collect`는
  - 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드
- `reduce`
  - 두 값을 **하나로 도출하는** **불변형 연산**
- `reduce` 메서드는 **누적자로 사용된 리스트**를 변환 시키므로, `reduce`를 잘못 사용한 예시
- 여러 스레드가 **동시에 같은 데이터 구조체**를 고치면 리스트 전체가 망가짐
  - **리듀싱 연산을 병렬로 수행할 수 없음**
- **가변 컨테이너** 관련 작업이면서, **병렬성**을 확보하려면
  - `collect` 메서드로 **리듀싱 연산**을 구현하는 것이 바람직

### 컬렉션 프레임워크 유연성 : 같은 연산도 다양한 방식으로 수행 가능
- `reducing` 컬렉터를 사용하면,
  - 이전 예제에서 람다 표현식 대신
  - `Integer` 클래스의 `sum` 메서드 참조를 이용하면, 코드 단순화 가능
- 예시 : 람다를 사용하지 않는 sum 
  ```java
  int totalCalories = menu.stream().collect(
    reducing(
      0,
      Dish::getCalories,
      Integer::sum
    )
  );
  ```
- 예시2 : `counting` 컬렉터 구현
  ```java
  public static <T> Collector<T, ?, Long> counting() {
    return reducing(0L, e -> 1L, Long::sum);
  }
  ```
- 예제 3 : 컬렉터를 사용하지 않는 방법
  ```java
  int totalCalories = menu.stream().map(Dish::getCalories).reduce(Integer::sum).get
  ```
- `Integer::sum`도
  - **빈 스트림**과 `null` 문제를 피할 수 있도록 `Optional<Int>`를 반환
  - 이후 `get`을 통해 자유롭게 호출
  - `orElse`나 `orElseGet` 등을 이용하여, `Optional` 값을 가져오는 것이 좋음
- 예제 4: `IntStream` 매핑
  ```java
  int totalCalories = menu.stream().mapToInt(Dish::getCalories).sum()
  ```

#### 자신의 상황에 맞는 최적의 해법 선택
- **스트림 인터페이스**에서 직접 제공하는 메서드를 이용하는 것에 비해
  - **컬렉터**를 사용하는 코드가 더 복잡함
  - 하지만 **재사용성**과 **커스터마이즈 가능성**을 제공하는 **높은 수준의 추상화**와 **일반화**를 얻음
- 일반적으로 **문제**에 특화된 해결책을 고르는 것이 바람직
  - **가독성**과 **성능**
- 위에서 `예제 4`에 해당하는 코드가, 상대적으로 간결
  - `IntStream`덕분에 **자동 언박싱** 연산을 수행하거나
  - `Integer`를 `int`로 변환하는 과정을 피할 수 있음

#### 제네릭 와일드카드 `?` 사용법
- `?`는 컬렉터의 누적자 형식이 **알려지지 않음**을 의미
  - 누적자의 형식이 **자유로움**
- `Collector` 클래스에서 원래 정의된 **메서드 시그니처**를 그대로 사용함