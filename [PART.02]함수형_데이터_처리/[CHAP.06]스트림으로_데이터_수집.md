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

## 6.3. 그룹화
- 메뉴 그룹화 - `Collectors.groupingBy`의 예시
  ```java
  Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
  ```
- 위 함수를 통해 **스트림**이 **그룹화**되므로
  - 이를 **분류 함수**(classification function)이라고 함
- 메서드 참조 대신 람다 표현식으로 구현하기
  ```java
  public enum CaloricLevel {DIET, NORMAL, FAT}
  Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream.collect(
    goupingBy(dish -> {
      if (dish.getCalories() <= 400) return CaloricLevel.DIET;
      else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
      else return CaloricLevel.FAT;
    })
  )
  ```

### 6.3.1. 그룹화된 요소 조작
- 요소 그룹화 이후 **각 결과 그룹의 요소 조작 연산**
- 필터링 하게 될 경우 문제 발생
  - 해당 필터가 걸리면서, `key`자체가 제외되는 문제 발생
- `predicate`를 이동하여 문제 해결
  ```java
  Map<Dish.Type, List<Dish>> caloricDishesByType = 
    menu.stream().collect(groupingBy(Dish::getType, 
      filtering(dish -> dish.getCalories() > 500, toList())));
  ```
- `filtering` 메서드는 `Collectors` 클래스의 또 다른 **정적 팩터리 메서드**로 `predicate`를 인수로 받음
  - 이 `predicate`로 각 그룹의 **요소**와 **필터링 된 요소**를 **재그룹화**
- **매핑 함수**를 사용하여 요소를 변환하기
  - `Collectors` 클래스는, 매핑 함수와 각 **항목에 적용한 함수**를 모으는 데 사용하는
    - 또 다른 컬렉터를 인수로 받음 -> `mapping` 메서드
  - 예시
    ```java
    Map<Dish.Type, List<String>> dishNamesByType = 
      menu.stream()
            .collect(groupingBy(Dish::getType, mapping(Dish::getName, toList())));
    ```
- `flatMapping`으로 각 형식의 요리 태그를 추출할 수 있음
  ```java
  Map<Dish.Type, List<String>> dishNamesByType = 
      menu.stream()
            .collect(groupingBy(Dish::getType, flatMapping(dish -> dishTags.get(dish.getName()).stream(), toSet())));
  ```

### 6.3.2. 다수준 그룹화
- 두 인수를 받는 `Collectors.groupingBy`를 이용하여
  - 항목을 **다수준으로 그룹화**할 수 있음
- `Collectors.groupingBy`는 일반적인 **분류 함수**와 **컬렉터**를 인수로 받음
  - 바깥쪽 `groupingBy` 메서드에
    - 스트림의 항목을 분류할 **두번째 기준을 정의하는** 내부 `groupingBy`를 전달하여, 두 수준으로 스트림의 항목 그룹화 가능
- 예시
  ```java
  Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = 
    menu.stream().collect(
      groupingBy(Dish::getType,
        groupingBy(dish -> {
          if (dish.getCalories() <= 400)
            return CaloricLevel.DIET;
          else if (dish.getCalories() <= 700)
            return CaloricLevel.NORMAL; 
          else return CaloricLevel.FAT;
        })
      )
    );
  ```
- 보통 `groupingBy` 연산을 `bucket`의 개념으로 생각하면 됨
  - 각 `groupingBy`는 각 키의 **버킷**을 생성
  - 그리고 준비된 각 버킷을 `substream collector`로 채워가기를 반복하면서 `n수준 그룹화` 진행

### 6.3.3. 서브그룹으로 데이터 수집
- 메뉴 종류별 연산
  ```java
  Map<Dish.Type, Long> typesCount = menu.stream().collect(
    groupingBy(Dish::getType, counting())
  );
  ```
- 인수 하나의 `groupingBy(f)`는 `groupingBy(f, toList())`와 동일
- 팩토리 메서드 `maxBy`의 특징
  - `Optional`로 `Map`의 값이 변화
  - `groupingBy` 컬렉터는, 스트림의 **첫 번째 요소**를 찾은 이후에야
    - 그룹화 맵에 **새로운 키를** 게으르게 추가
    - 리듀싱 컬렉터가 반환하는 형식을 사용하는 상황이므로,
      - 굳이 `Optional` 래퍼를 사용할 필요가 없음

#### 컬렉터 결과를 다른 형식에 적용하기
- 맵의 모든 값을 `Optional`로 감쌀 필요가 없을 때, 제거 가능
- `Collectors.collectingAndThen` 컬렉터를 활용하면 됨
  ```java
  Map<Dish.Type, Dish> mostCaloricByType =
    menu.stream()
        .collect(groupingBy(Dish::getType, // 분류 함수 
          collectingAndThen(
            maxBy(comparingInt(Dish::getCalories)), // 감싸진 컬렉터
            Optional::get))); // 변환 함수
  ```
- `collectingAndThen`은
  - 적용할 **컬렉터**와 **변환 함수**를 인수로 받아 처리
- **리듀싱 컬렉터**의 경우, 절대 `Optional.empty()`를 리턴하지 않으므로, **안전한 코드**

#### groupingBy와 함께 사용하는 다른 컬렉터 예제
- `goupringBy`에 두번째 인수로 전달한 컬렉터 사용
  - 같은 그룹으로 분류된 **모든 요소**에 **리듀싱 작업** 수행시
- 예제 - 모든 요리의 칼로리 합계
  ```java
  Map<Dish.Type, Integer> totalCaloriesByType =
    menu.stream().collect(groupingBy(Dish::getType, summingInt(Dish::getCalories)));
  ```
- `mapping`으로 만들어진 **컬렉터**도 `groupingBy`와 자주 사용
  - `mapping` 메서드는
    - **스트림의 인수**를 변환하는 함수와
    - **변환 함수의 결과 객체를 누적하는 컬렉터**를 인수로 받음
- 예를 들어, 각 요리 형식에 존재하는 모든 `CaloricLevel`을 알고 싶을 때
  ```java
  Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType =
    menu.stream().collect(
      groupingBy(Dish::getType, mapping(dish -> {
        if(dish.getCalories() <= 400) return CaloricLevel.DIET;
        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
        else return CaloricLevel.FAT; }, toSet()
      ))
    );
  ```
- 결과 제어 : `toCollection`을 사용한 결과 제어(`Set`의 형식 부여)
  ```java
  Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType =
    menu.stream().collect(
      groupingBy(Dish::getType, mapping(dish -> {
        if(dish.getCalories() <= 400) return CaloricLevel.DIET;
        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
        else return CaloricLevel.FAT; },
        toCollection(HashSet::new)
      }
      ))
    );
  ```

## 6.4. 분할
- **분할 함수**(partitioning function)라 불리는 `predicate`를 **분류 함수**로 사용하는 특수한 그룹화 기능
- `Boolean`을 반환하기 때문에, 키 형식은 `Boolean`
- 그룹화 맵은 최대 두개의 그룹으로 분류(참 or 거짓)
- 예제 : 채식 요리 구분하기
  ```java
  Map<Boolean, List<Dish>> partitionedMenu = 
    menu.stream().collect(partitioningBy(Dish::isVegetarian)); // 분할 함수
  // false, true를 키로하는 분류된 맵이 반환됨

  // 이전 예제에서 생성한 `predicate`로 필터링하여, 별도의 리스트에 결과 수집을 하는 방법
  List<Dish> vegetarianDishes = menu.stream().filter(Dish::isVegetarian.collect(toList()));
  ```

### 6.4.1. 분할의 장점
- 분할 함수가 반환하는 `참, 거짓` 두 가지 요소의 스트림 리스트를
  - 모두 유지한다는 것이 분할의 장점
- **컬렉터**를 **두 번째 인수**로 전달할 수 있는 `partitioningBy`
  ```java
  Map<Boolean, Map<Dish.Type, List<Dish>>> vegetariuanDishesByType = menu.stream().collect(
    partitioningBy(Dish::isVegetarian, // 분할 함수
    groupingBy(Dish::getType))); // 두 번째 컬렉터
  ```
  - depth가 깊어지면서, `false, true`로 구분된 맵 내부에서 `Dish::type`별로 나누어짐
- 채식 요리와 채식 요리가 아닌 요리 각각 그룹에서, 가장 칼로리가 높은 요리 찾기
  ```java
  Map<Boolean, Dish> mostCaloricPartitionedByVegeterian = 
    menu.stream().collect(
      partitioningBy(Dish::isVegetarian,
      collectiongAndThen(maxBy(comparingInt(Dish::getCalories)), Optional::get)));
  ```
- `partitioningBy`가 반환한 맵 구현은
  - 참과 거짓 두 가지 키만 포함하므로 더 간결하고 효과적

### 6.4.2. 숫자를 소수와 비소수로 분할하기
- 정수 `n`을 받아 `2 ~ n`까지의 **자연수**를 **소수**(prime)와 **비소수**(non-prime)으로 나누는 프로그램
  ```java
  public boolean isPrime(int candidate) {
    return IntStream.range(2, candidate) // 2부터 candidate 미만의 자연수 생성
              .noneMatch(i -> candidate % i == 0); // 스트림을 모든 정수로 candidate를 나눌수 없다면 참
  }

  // // 소수의 대상을 제고급 이하의 수로 제한
  public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double)candidate);
    return IntStream.rangeClosed(2, candidateRoot)
    .noneMatch(i -> candidate % i == 0);
  }
  ```
- `n`개의 숫자를 포함하는 스트림 제작 이후
  - `isPrime` 메서드를 `predicate`로 이용하고 `partitioningBy` 컬렉터로 **리듀스**해서
  - 숫자를 **소수**와 **비소수**로 분류할 수 있음
  - 코드
    ```java
    public Map<Boolean, List<Integer>> partitionPrimes(int n) {
      return IntStream.rangeClosed(2,n).boxed()
      .collect(
        partitioningBy(candidate -> isPrime(candidate)))};
    ```

## 6.5. Collector 인터페이스
- **Collector 인터페이스**는 리듀싱 연산(컬렉터)을 어떻게 구현할지 제공하는 **메서드 집합**으로 구성
- `toList, groupingBy` 등
- **Collector 인터페이스**를 구현하는 **리듀싱 연산**을 만들수도 있음

#### CODE.6.4. Collector 인터페이스
```java
public interface Collector<T, A, R> {
  Supplier<A> supplier();
  BiConsumer<A, T> accumulator();
  Function<A, R> finisher();
  BinaryOperator<A> combinder();
  Set<Chracterlistics> characteristics();
}
```
- `T` : **수집**될 스트림 항목의 제네릭 형식
- `A` : **누적자**, 수집 과정에서 중간 결과를 누적하는 결과의 형식
- `R` : 수집 연산 **결과 객체**의 형식(대개 걸렉션)
- `Stream<T>`의 모든 요소를 `List<T>`로 수집하는 `ToListCollector<T>` 객체
  ```java
  public class ToListCollector<T> implements Collector<T, List<T>, List<T>>
  ```

### 6.5.1. Collector 인터페이스의 메서드 살펴보기
- 네 개의 메서드는 `collect` 메서드에서 실행하는 **함수**를 반환
- 마지막 메서드 `characteristics`는 
  - `collect` 메서드가 어떤 최적화(e.g. 병렬화)를 이용하여 
  - **리듀싱 연산**을 수행할 것인지 결정하도록 돕는 **힌트 특성 집합**을 제공

#### accumulator 메서드 : 결과 컨테니어에 요소 추가하기
- **리듀싱 연산**을 수행하는 함수
- 스트림에서
  - `n`번째 요소를 **탐색**할 때
  - 두 인수, 즉 누적자(`n-1을 수집한 상태`)와 `n`번째 요소를 함수에 적용
- 함수의 반환값은 `void`
  - 요소를 탐색하면서 **적요하는 함수에 의해 누적자 내부 상태가 변경**
  - 누적자가 어떤 값인지 단정할 수는 없음
- `ToListCollector`에서 `accumulator`가 반환하는 함수는
  - 이미 탐색한 항목을 포함하는 **리스트**에 **현재 항목을 추가하는 연산 수행**
- 코드
  ```java
  public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
  }

  // 메서드 참조를 사용하는 방법
  public BiConsumer<List<T>, T> accumulator() {
    return List::add;
  }
  ```

#### finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용하기
- `finisher`메서드는
  - **스트림 탐색**을 끝내고, **누적자 객체**를 **최종 결과**로 반환하면서
  - 누적 과정을 끝낼 때 호출할 함수를 반환해야 함
- 때로는 `ToListCollector`에서 볼 수 있는 것처럼, 누적자 객체가 **이미 최종 결과**인 상황도 있음
- 이런 때는 **변환 과정**이 필요하지 않으므로, `finisher` 메서드는 항등 함수를 반환
- 코드
  ```java
  public Function<List<T>, List<T>> finisher() {
    return Function.identity();
  }
  ```

#### 순차 리듀싱 리듀싱 기능 수행
- 순서
  ```java
  시작.
  A accumulator = collector.supplier().get();
  collector.accumulator().accept(accumulator, next)
  
  스트림에 요소가 남아 있는가?
    yes
      T next = 스트림의 다음 항목
      collector.accumulator().accept(accumulator, next)
      반복
    no
      R result = collector.finisher().apply(accumulator);
      return result;
  끝.
  ```
- 실제로는 `collect`가 동작하기 이전에
  - 다른 **중간 연산**과
  - **파이프라인**을 구성할 수 있게 해주는 **게으른 특성**
  - **병렬 실행** 등도 고려해야 하므로, **스트림 리듀싱 구현 기능**은 생각보다 복잡함

#### combiner 메서드 : 두 결과 컨테이너 병합
- 리듀싱 연산에서 **사용할 함수**를 반환
- `combiner`는
  - 스트림의 서로 다른 **서브 파트**를 **병렬**로 처리할 때
  - **누적자**가 이 결과를 어떻게 처리할지를 정의
- `toList`의 `combiner`
  - 스트림의 두번째 **서브 파트**에서 수집한 **항목 리스트**를
  - **첫번째 서브 파트** 결과 리스트에 추가하면 됨
  - 코드
    ```java
    public BinaryOperator<List<T>> combimner() {
      return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
      }
    }
    ```
- 이 메서드를 활용하면, **스트림의 리듀싱**을 **병렬**로 수행할 수 있음
- 스트림의 리듀싱을 **병렬**로 수행할 때
  - java 7의 `fork/join` 프레임워크와
  - `Spliterator`를 사용

#### 스트림의 병렬 리듀싱 수행 과정
- 스트림을 분할해야하는지 정의하는 조건이 **거짓**으로 바뀌기 전까지
  - 원래의 스트림을 **재귀적으로 분할**
  - 분산된 작업의 크기가 `너무 작아지면`
    - 병렬 수행의 속도는 **순차 수행의 속도보다 느림**
    - 병렬 수행의 효과가 **상쇄**
- 모든 `substream`의 각 요소에 **리듀싱 연산**을 순차적으로 적용
  - `substream`을 병렬로 처리
- `combiner` 메서드가 반환하는 함수로 모든 부분결과를 **쌍**으로 합친다
  - 분할된 모든 서브 스트림의 결과를 합치면서 연산 완료

#### Characteristics 메서드
- 컬렉션의 연산을 정의하는 `Characteristics` 형식의 **불변 집합**을 반환
- 스트림을 **병렬**로 리듀스할 것인지, 병렬로 리듀스 한다면
  - 어떤 **최적화**를 선택해야할지 힌트 제공
- 다음 세 항목을 포함하는 **열거형**
  - `UNORDERED`
    - 리듀싱 결과는 **스트림 요소**의 **방문 순서**나 **누적 순서**에 영향을 받지 않음
  - `CONCURRENT`
    - 다중 스레드에서 `accumulator`함수를 동시에 호출할 수 있음
    - 이 컬렉터는 스트림의 **병렬 리듀싱**을 수행할 수 있음
    - 컬렉터의 플래그에 `UNORDERED`를 함께 설정하지 않았다면
      - 데이터 소스가 **졍렬되어 있지 않은**(집합처럼 요소의 **순서가 무의미**) 상황에서만 병렬 리듀싱 가능
  - `IDENTITY_FINISH`
    - `finisher` 메서드가 반환하는 함수는 단순히 `identity`를 적용할 뿐이므로, 이를 생략할 수 있음
    - 리듀싱 과정의 최종 결과로 **누적자 객체를 바로 사용 가능**
    - 누적자 `A`를 결과 `R`로 안전하게 형변환 가능
- 지금까지 개발한 `ToListCollector`에서
  - 스트림의 요소를 누적하는데 사용한 리스트가 **최종 결과 형식**이므로, 추가 변환이 필요 없음
- `ToListCollector`은 `IDENTITY_FINISH`
- 리스트의 순서는 상관이 없으므로, `UNORDERED`
- `CONCURRENT`에도 부합함
  - 단, 요소의 순서가 무의미한 데이터 소스일 경우에만 가능

### 6.5.2. 응용하기
- 커스텀 `ToListCollector` 구현하기

#### CODE.6.5. ToListCollector
```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {
  @Override
  public Supplier<List<T>> supplier() {
    return ArrayList::new; // 수집 연산의 시작점
  }

  @Override
  public BiConsumer<List<T>, T> accumulator() {
    return List::add; // 탐색한 항목을 누적하고, 누적자를 고침
  }

  @Override
  public Function<List<T>, List<T>> finisher() {
    return Function.identity(); // 항등 함수
  }

  @Override
  public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> { 
      list1.addAll(list2); // 두 번째 콘텐츠와 합쳐서, 첫 번째 누적자를 고침
      return list1; // 변경된 첫번째 누적자 반환
    };
  }

  @Override
  public Set<Caracteristics> characteristics() {
    return Collections.unmodifiableSet(EnumSet.of(IDENTIFY_FINISH, CONCURRENT)); // 컬렉션의 플래그 설정
  }
}
```
- 위 구현이 `Collectors.toList` 메서드가 반환하는 결과와 완전히 같은 것은 아님,
  - 사소한 최적화를 제외하면, 대체로 비슷함
- `JAVA API`에서 제공하는 컬렉터는 singleton `Collectors.emptyList()`로 **빈 리스트**를 반환
- 기존의 코드와 비교
  ```java
  // ToListcollector 활용
  List<Dish> dishes = menuStream.collect(new ToListCollector<Dish>());
  // 기존 코드
  List<Dish> dishes = menuStream.collect(toList());
  ```

#### 컬렉터 구현을 만들지 않고도 커스텀 수집 수행하기
- `IDENTITY_FINISH` 수집 연산에서는 `Collector` 인터페이스를 완전히 새로 구현하지 않고도, 같은 결과를 얻을 수 있음
- `Stream`은 세 함수(발생, 누적, 합침)을 인수로 받는 `collect` 메서드를 **오버로드**하며
  - 각가의 메서드는 `Collector` 인터페이스의 메서드가 반환하는 함수과 같은 기능 수행
- 예시 : 스트림의 모든 항목을 리스트에 수집
  ```java
  List<Dish> dishes = menuStream.collect(
    ArrayList::new, // 발행
    List:add, // 누적
    List::addAll // 합침
    );
  ```
- 더 간결하고 축약성은 있으나, **가독성이 떨어짐**
- 적절한 클래스로 **커스텀 컬렉터**를 구현하는 편이
  - 중복을 피하고 재사용성을 높임
- 위 `collect` 메서드로는 `Characteristics`를 전달 할 수 없음
  - 오직 `IDENTITY_FINISH`와 `CONCURRENT`지만, `UNORDERED`는 아닌 컬렉터로만 동작

## 6.6. 커스텀 컬렉터를 구현해서 성능 개선하기
#### CODE.6.6. n이하의 자연수를 소수와 비소수로 분류하기
```java
public Map<Boolean, List<Integer>> partitionPrimes(int n) {
  return IntStream.rangeClosed(2, n).boxed()
                  .collect(partitioningBy(candidate -> isPrime(candidate)));
}

// 제곱근 이하로 candidate(대상)의 숫자 범위를 제한하여 isPrime 메서드 개선
public boolean isPrime(int candidate) {
  int candidateRoot = (int) Math.sqrt((double) candidate);
  return IntStream.rangeClosed(2, candidateRoot)
  .noneMatch(i -> candidate % i == 0);
}
```

### 6.6.1. 소수로만 나누기
- 소수로 나누어떨어지는지 확인하여 대상의 범위 좁히기
- 컬렉터로는 **컬렉터 수집 과정**에서 **부분결과**에 접근하기 어려우므로
  - 커스텀 컬렉터 클래스로 이를 해결하기
- 중간 결과 리스트가 있다면, `isPrime`을 가지고 전달
  ```java
  public static boolean isPrime(List<Integer> primes, int candidate) {
    return primes.stream().noneMatch(i -> candidate % i == 0);
  }
  ```
- 대상의 제곱보다 큰 소수를 찾으면 검사를 중단하여 성능 문제 개선하기
  - 정렬된 리스트와 `predicate`를 인수로 받아,
  - 리스트의 첫 요소에서 시작해서, `predicate`를 만족하는 가장 긴 요소로 이루어진 리스트 반환
  ```java
  public static boolean isPrime(List<Integer> primes, int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return primes.stream()
                  .takeWhile(i -> i <= candidateRoot)
                  .noneMatch(i -> candidate % i == 0);
  }
  ```
- `takeWhile`의 구현
  - 정렬된 리스트의 `predicate`를 인수로 받아, `predicate`를 만족하는 가장 긴 **첫 요소 리스트**를 반환하도록 하기
  ```java
  public static <A> List<A> takeWhile(List<A> list, Predicate<A> p) {
    int i = 0;
    for (A item : list) {
      if (!p.test(item)) { // 리스트의 현재 항목이 predicate를 만족하는지 확인
        return list.subList(0, i); // 만족하지 않을 경우, 현재 검사한 항목의 이전 하위 항목 리스트를 반환
      }
      i++;
    }
    return list;
  }
  ```

#### 1단계 : Collector 클래스 시그니처 정의
```java
// T : 스트림 요소의 형식
// A : 중간 결과를 누적하는 객체의 형식
// R : collect 연산의 최종 결과 형식
public interface Collector<T, A, R>
```
- 정수로 이루어진 스트림에서
  - `Map<Boolean, List<Integer>>`인 컬렉터를 구현해야 함
  - 값에는 `소수`와 `소수가 아닌 수`를 가짐
- 형태
  ```java
  public class PrimeNumbersCollector implements Collector<Integer, 
                                                          Map<Boolean, List<Integer>>, // 누적자 형식
                                                          Map<Boolean, List<Integer>>> // 수집 연산의 결과 형식
  ```

#### 2단계 : 리듀싱 연산 구현
- `Collector` 인터페이스에 선언된 다섯 메서드 구현하기
- **Supplier**
  ```java
  public Supplier<Map<Boolean, List<Integer>>> supplier() {
    return () -> new HashMap<Boolean, List<Integer>>() {{
      put(true, new ArrayList<Integer>());
      put(false, new ArrayList<Integer>());
    }}
  }
  ```
  - 누적자로 사용할 `Map`을 생성하면서
    - `true`, `false`키와 **빈 리스트**로 초기화
- **BiConsumer**, accumulator 만들기
  - 스트림의 요소를 어떻게 수집할 것인지에 대한 내용
  - 언제든지 원할 때 **수집 과정의 중간 결과**를 리턴할 수 있어야 함
  ```java
  public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator() {
    return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
      acc.get(isPrime(acc.get(true), candidate)) // isPrime의 결과에 따라 소수 리스트와 비소수 리스트를 만듦
      .add(candidate); // candidate에 알맞은 리스트를 추가
    }
  }
  ```

#### 3단계 : 병렬 실행할 수 있는 컬렉터 만들기(가능하다면)
- **병렬 수집 과정**에서 **두 부분 누적자**를 합칠 수 있는 메서드 만들기
- 코드
  ```java
  public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
    return (Map<Boolean, List<Integer>> map1, Map<Boolean, List<Integer>> map2) -> {
      map1.get(true).addAll(map2.get(true));
      map1.get(false).addAll(map2.get(false));
      return map1;
    };
  }
  ```
- 알고리즘 자체가 순차적이라, 실제 병렬로 사용은 불가능
- `combiner`는 실제 호출될 일은 없음
  - 빈 구현으로 남기거나, `UnsupportedOperationException`을 던지도록 구현하면 좋음

#### 4단계 : finisher 메서드와 컬렉터의 characteristics 메서드
- **finisher**
  ```java
  // acc는 collector 결과 형식과 같으므로, 변환이 필요 없음
  // 항등함수 identity를 반환하도록 finisher 메서드 구현
  public Function<Map<Boolean, List<Integer>>, Map<Boolean, List<Integer>>> finisher() {
    return Function.identity();
  }
  ```
- **characteristics**
  - `CONCURRENT`도 아니고, `UNORDERED`도 아니지만, `IDENTITY_FINISH`이므로 다음과 같이 구현
  ```java
  public Set<Characteristics> characteristics() {
    return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH));
  }
  ```
- 위 내용을 활용하여 `partitioningBy`를 이용하면, 커스텀 컬렉터로 교체 가능
  ```java
  public Map<Boolean, List<Integer>> partitionPrimesWithCustomCollector(int n) {
    return IntStream.rangeClosed(2, n).boxed().collect(new PrimeNumbersCollector());
  }
  ```

### 6.6.2. 컬렉터 성능 비교
- `partitioningBy`로 만든 코드와 **커스텀 컬렉터**로 만든 코드의 기능은 같음
- 성능 확인 코드
  ```java
  public class CollectorHarness {
    public static void main(String[] args) {
      long fastest = Long.MAX_VALUE;
      for(int i = 0; i < 10 ; i ++) { // 테스트를 10번 반복
        long start = Sysmtem.nanoTime();
        partitionPrimes(1_000_000); // 백만개의 숫자를 소수와 비소수로 분할
        long duration = (System.nanoTime() - start) / 1_000_000; // duration을 ms단위로 측정
        if (duration < fastest) fastest = duration; // 가장 빨리 실행되었는지 확인
      }
      System.out.println("Fastest execution done in " + fastest + " msecs");
    }
  }
  ```
- **JMH**와 같은 과학적인 벤치마킹을 사용할 수 있으나,
  - 간단한 예제이므로, 작은 벤치마킹 클래스로도 정확한 결과를 얻을 수 있음
- `partitionPrimes`를 `partitionPrimesWithCustomCollector`로 변경하여 상호 구동하면 됨
- **오버로드된 버전**의 `collect` 메서드로, `PrimeNumbersCollector`의 핵심 로직을 구현하는 세 함수를 전달하여 구현하기
  ```java
  public Map<Boolean, List<Integer>> partitionPrimesWithCustomCollector(int n) {
    IntStream.rangeClosed(2, n).boxed()
                                .collect(
                                  () -> new HashMap<Boolean, List<Integer>>() {{ // 발행
                                    put(true, new ArrayList<Integer>());
                                    put(false, new ArrayList<Integer>());
                                  }},
                                  (acc, candidate) -> { // 누적
                                    acc.get(isPrime(acc.get(true), candidate))
                                        .add(candidate);
                                  },
                                  (map1, map2) -> { // 합침
                                    map1.get(true).addAll(map2.get(true));
                                    map1.get(false).addAll(map2.get(false));
                                  }
                                );
  }
  ```
  - `Collector` 인터페이스를 구현하는 새로운 클래스를 만들지 않아도 되지만,
    - 코드가 간결하나, 가독성과 재사용성이 떨어짐

## 6.7. 마치며
- `collect`는 스트림의 요소를 **요약 결과**로 누적하는 다양한 방법을 인수로 갖는 최종 연산
- 스트림의 요소를 하나의 값으로 **리듀스** 하는 것 뿐 아니라, **최솟값, 최댓값, 평균값**을 계산하는 컬렉터가 미리 정의됨
- 미리 정의된 컬렉터인
  - `groupingBy`로 스트림 요소를 그룹화 하거나
  - `partitioningBy`로 스트림 요소 분할 가능
- 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합
- `Collector` 인터페이스에 정의된 메서드를 구현하여, **커스텀 컬렉터** 개발 가능