# [CHAP.08] 컬렉션 API 개선

## 8.1. 컬렉션 팩토리
- `java 9`에서는 **작은 컬렉션 객체**를 쉽게 접할 수 있는 방법 제공
- 예시 : 친구 이름을 포함하는 그룹을 만들기
  ```java
  List<String> friends = new ArrayList<>();
  friends.add("Raphael");
  friends.add("Olivia");
  friends.add("Thibaut");

  // Arrays.asList() 사용
  List<String> frieds = Arrays.asList("Raphael", "Olivia", "Thibaut");
  ```
- 위 코드의 경우, **고정 크기의 리스트**이기 때문에, 요소를 추가하거나 삭제는 불가능
  - `UnsupportedOperationException`이 발생

#### UnsupportedOperationException
- 내부적으로 고정된 크기의 변환할 수 있는 배열로 구성했기 때문
- `Set`의 경우 `HastSet`을 사용하여 작업 가능
  ```java
  Set<String> friends = new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut"));

  // Stream API
  Set<String> friends = Stream.of("Raphael", "Olivia", "Thibaut").collect(Collectors.toSet());
  ```
- 하지만 위 두 방법 모두, 내부적으로 **불필요한 객체 할당**을 필요로 함
- `Map`은 별도의 멋진 방법은 없으나,
  - `java 9`에서 **작은 리스트, 집합, 맵**을 만들 수 있는 **팩토리 메서드 제공

#### 컬렉션 리터럴
- `python`, `groovy`와 같은 언어에서 제공
- `java 9`에서는 **컬렉션 API**로 개선하여 제공

### 8.1.1. 리스트 팩토리
- `List.of` 팩토리 메서드로 간단하게 리스트 생성 가능
  ```java
  List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
  
  // 리스트 추가
  friends.add("Chih-Chun"); // UnsupportedOperationException
  ```
- 고정된 크기 뿐 아니라, `null`요소도 제한 됨

#### 오버로딩 vs 가변 인수
- `List.of`에는 다양한 오버로드 버전이 존재
  ```java
  static <E> List<E> of(E e1, E e2, E e3, E e4);
  static <E> List<E> of(E e1, E e2, E e3, E e4, E e5);
  ```
  - 단 가변인수 버전은 존재하지 않음
- 내부적으로 **가변 인수 버전**은 **추가 배열**을 할당하여 리스트로 감싸는 형태
- **배열**을 할당하고 초기화하여, 나중에 `GC` 비용을 지불해야함
- 고정된 숫자의 요소를 API로 제공하므로, 이러한 비용 제거가 가능
- `List.of`로 **열 개 이상**의 요소를 가진 리스트를 만들 수도 있겠지만,
  - 이 때는 가변 인수를 이용하는 메서드가 사용됨
- `Set.of`, `Map.of`에서도 동일한 패턴 등장

#### 스트림 API와 컬렉션 팩토리 메서드의 사용
- 데이터 처리 형식을 설정하거나, 데이터로 변환할 필요가 없다면, **팩토리 메서드**를 사용하기를 권장
  - 더 단순하고, 목적을 달성하기 쉽기 때문

### 8.1.2. 집합 팩토리
- `Set`의 사용 방법
  ```java
  Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
  ```
- 중복된 요소 집합을 만들려고 하면, `IllegalArgumentException`이 발생

#### 8.1.3. 맵 팩토리
- `Map`은 `List`나 `Set`에 비해 복잡함
  - `key`가 필요하기 때문
- 단, `java 9`부터는 `Map.of`를 사용하여 **키와 값을 번갈아 제공**하는 방법으로 사용 가능
  ```java
  Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
  ```
  - 열 개 이하의 키와 값 쌍을 가진 **작은맵**을 만들때는 이 메서드가 유용함
    - 이 이상의 맵을 구현하려면 `Map.Entry<K, V>`객체를 인수를 받으며
- **가변 인수**로 구현된 `Map.ofEntries` 팩토리 메서드를 이용하는 것이 좋음
  ```java
  import static java.util.Map.entry;
  Map<String, Integer> ageOffFriends = Map.ofEntries(entry("Raphael", 30),
                                                    entry("Olivia", 25),
                                                    entry("Thibaut", 26));
  ```
  - `Map.entry`는 `Map.Entry` 객체를 만드는 새로운 **팩터리 메서드**

## 8.2. 리스트와 집합 처리
- `java 8`에서 `List`, `Set` 인터페이스에 다음과 같은 메서드 추가
  - `removeIf`
    - `predicate`를 만족하는 요소 제거
    - `List`, `Set`을 구현하거나, 그 구현을 상속받은 모든 클래스에서 사용 가능
  - `replaceAll`
    - 리스트에서 이용할 수 있는 기능
    - `UnaryOperator` 함수를 이용해 요소를 변경
  - `sort`
    - `List` 인터페이스에서 제공
    - 리스트 정렬
- 새로운 결과를 만드는 **스트림 동작**이 아닌 **기존 컬렉션을 변경**

### 8.2.1. removeIf 메서드
- 예시 : 숫자로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드
  ```java
  // ConcurrentModificationException이 발생
  for (Transaction transaction : transactions) {
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
          transactions.remove(transaction);
      }
  }
  
  // 위 코드는 내부적으로 아래와 같이 처리, `for-each`는 `Iterator`객체를 사용
  for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
      Transaction transaction = iterator.next();
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
          transactions.remove(transaction); // 반복하면서 별도의 두 객체를 통해 컬렉션을 변경
      }
  }
  ```
- 두 개의 개별 객체가 **컬렉션**을 관리하는 상황
  - `Iterator 객체`, `next()`와 `hasNext()`를 이용해 소스를 질의
  - `Collection` 객체 자체, `remove()`를 호출해 요소 삭제
- 결과적으로 **반복자의 상태**가 **컬렉션의 상태**와 동기화 되지 않음
- `Iterator`를 명시적으로 사용하고, 그 객체의 `remove` 메서드를 호출함으로 이 문제 해결 가능
  ```java
  for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
      Transaction transaction = iterator.next();
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
          iterator.remove();
      }
  }

  // java 8의 removeIf 사용, 버그도 없으며, 간결
  transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
  ```

### 8.2.2. replaceAll 메서드
- `List` 인터페이스의 `replaceAll` 메서드를 이용하여
  - 리스트의 각 요소를 **새로운 요소로 변경**
- 코드
  ```java
  // Stream API, 새 문자열 컬렉션을 만듦
  referenceCodes.stream()
                .map(code -> 
                        Character.toUpperCase(code.charAt(0)) +
                        code.substring(1)
                ).collect(Collectors.toList())
                .forEach(System.out::println);

  // ListIterator 객체 사용, 요소를 바꾸는 set 메서드 활용
  for(ListIterator<String> iterator = referenceCodes.listIterator(); iterator.hasNex();) {
      String code = iterator.next();
      iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
  }

  // 이전과 마찬가지로
  // 위 코드는 컬렉션 객체를 `Iterator`객체와 혼용하면, 
  // 반복과 컬렉션 변경이 동시에 이루어지면서 문제 발생 가능성 존재
  // java 8 기능 활용
  referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
  ```

## 8.3. 맵 처리
- `java 8`에서는 `Map` 인터페이스에 몇 가지 **default method**가 추가됨

### 8.3.1. forEach 메서드
- 맵에서 **key, value**를 **반복**하며 확인하는 작업은, 번거로운 작업
- 실제로는 `Map.Entry<K,V>` 반복자를 활용하여, 맵의 **항목 집합**을 반복할 수 있음
  ```java
  for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
    System.out.println(friend + " is " + age + " years old");
  }

  // java 8부터 `BiConsumer`(key, value)를 인수로 받는 forEach를 지원
  ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
  ```

### 8.3.2. 정렬 메서드
- 두 개의 새로운 유틸리티를 활용, 맵의 항목을 `value`또는 `key`를 기준으로 정렬 가능
  ```java
  Entry.comparingByValue
  Entry.comparingByKey
  ```
- 예시 코드
  ```java
  Map<String, String> favoriteMovies
                        = Map.ofEntries(entry("Raphael", "Star Wars"),
                        entry("Cristina", "Matrix"),
                        entry("Olivia", "James Bond"));
  
  favoriteMovies
    .entrySet()
    .stream()
    .sorted(Entry.comparingByKey())
    .forEachOrdered(System.out::println); // 사람의 이름을 알파벳 순으로 스트림 요소 처리
  ```

#### HashMap 성능
- `java 8`에서는 `HashMap`의 내부 구조를 바꿔 성능을 개선함
- 기존의 맵의 항목은
  - **키**로 생성한 **해시코드**로 접근할 수 있는 **버켓**에 저장
  - 많은 키가 같은 해시코드를 반환하는 상황이 되면
    - `O(n)`의 시간이 걸리는 `LinkedList`로 버킷을 반환해야 하기 때문에
    - 성능이 저하됨
- 최근에는 버킷이 너무 커질 경우, `O(log(n))`의 시간이 소요되는
  - **정렬된 트리**를 이용하여, 동적으로 치환
  - `충돌이 일어나는 요소 반환 성능을 개선`
- 단, `key`가 `String, Number` 클래스 같은 `Comparable` 형태여야만 **정렬된 트리**가 지원

### 8.3.3. getOrDefault 메서드
- 해당되는 키가 존재하지 않을 때, **기본값**을 반환
- 인수
  - 첫번째 인수 : `키`
  - 두번째 인수 : `기본값`
- 맵에 키가 존재하지 않으면 `기본값` 반환
- 예시
  ```java
  Map<String, String> favoriteMovies = 
        Map.ofEntries(entry("Raphael", "Star Wars"), entry("Olivia", "James Bond"));

  System.out.println(favoriteMovies.getOrDefault("Olivia", "Matrix"));
  ```
- 물론 `value`가 `null`일 경우엔, `getOrDefault`에서도 `null`이 반환됨

### 8.3.4. 계산 패턴
- 맵에 **키가 존재하는지 여부**에 따라, 특정 동작을 실행하고, 결과를 저장해야하는 상황이 필요한 경우
- 예시
  - 키를 이용해 **값 비싼 동작**을 실행하여 얻은 결과를 **캐시**
  - 키가 존재하면, 굳이 재계산하지 않음
- 관련 메서드
  - `computeIfAbsent` : `제공된 키에 해당하는 값이 없으면`(값이 없거나, `null`), 키를 이용해 새 값을 계산하여 맵에 추가
  - `computeIfPresent` : `제공된 키가 존재하면`, 새 값을 계산하고 맵에 추가
  - `compute` : 제공된 키로 새 값을 계산하고, 맵에 저장
- 정보를 **캐싱**할 때, `computeIfAbsent`를 이용해 계산 가능
- 예시 : 파일 집합의 각 행을 파싱하여 `SHA-256`을 계산
  - 기존에 이미 데이터 처리시, 다시 계산할 필요가 없음
  - 맵을 이용해 **캐시**를 구현했을 때,
    - 다음과 같이 `MessageDigest` 인스턴스로 `SHA-256`의 해시를 계산할 수 있음
  - 코드
    ```java
    Map<String, byte[]> dataToHash = new HashMap<>();
    MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");

    // 데이터를 반복하면서 결과 캐시 가능
    lines.forEach(line ->
      dataToHash.computeIfAbsent(
                      line, // 맵에서 찾을 키
                      this::calculateDigest // 키가 존재하지 않을경우 계싼 수행
    ));

    private byte[] calcutateDigest(String key) { // 헬퍼가 제공된 키의 해시 계산
      return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
    }
    ```
- 예시 : 여러 값을 저장하는 맵을 처리
  - `Map<K, List<V>>`에 요소를 추가하려면, 항목이 **초기화 되었는지 확인 필요**
  - `Raphael`에게 줄 영화 목록을 만든다고 가정,
    ```java
    String friend = "Raphael";
    List<String> movies = friendsToMovies.get(friend);
    if(movies == null) { // 리스트가 초기화 되었는지 확인
      movies = new ArrayList<>();
      friendsToMovies.put(friend, movies);
    }
    movies.add("Star Wars"); // 영화 추가
    System.out.println(friendsToMovies);

    // computeIfAbsent를 활용하기
    friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>()).add("Star Wars");
    ```
- `computeIfPresent` 메서드는
  - 현재 키와 관련된 값이 **맵**에 존재하며, `null`이 아닐때만 새 값을 계산
  - 값을 만드는 함수가 `null`을 반환하면, 현재 매핑을 **맵에서 제거**
  - 하지만 매핑 제거시엔, `remove` 메서드를 `override`하는 것이 더 적합


### 8.3.5. 삭제 패턴
- 제공된 키에 해당하는 맵 항목 제거 -> `remove` 메서드
- `java 8`에서는 키가 특정한 값과 연관되었을 때만 **항목**을 제거하는 **오버르도 버전 메서드**를 제공
- 기존 코드와 비교
  ```java
  String key = "Raphael";
  String value = "Jack Reacher 2";
  if (favoriteMovies.containsKey(key) ** Object.equals(favoriteMovies.get(key), value)) {
    favoriteMovies.remove(key);
    return true;
  } else {
    return false;
  }

  // 간결하게 구현
  favoriteMovies.remove(key, value);
  ```

### 8.3.6. 교체 패턴
- 맵의 **항목을 바꾸기**
  - `replaceAll` : `BiFunction`을 적용한 결과, 항목의 값을 교체
  - `replace` : 키가 존재하면, 맵의 값을 변경. 특정 값으로 매핑되었을 때만 값을 교체하는 **overload**버전도 존재
- 코드
  ```java
  Map<String, String> favoriteMovies = new HashMap<>(); // replaceAll을 적용하기 위해, 바꿀 수 있는 맵을 지정
  favoriteMovies.put("Raphael", "Star Wars");
  favoriteMovies.put("Olivia", "james bond");
  favoriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
  System.out.println(favoriteMovies);
  ```

### 8.3.7 합침
- 예시 : 두 그룹의 연락처를 포함하는 **두 개의 맵을 합치기**
- `putAll`을 사용할 수 있음
  ```java
  Map<String, String> family = Map.ofEntries(
    entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
  Mapo<String, String> friends = Map.ofEntries(
    entry("Raphael", "Star Wars"));
  Map<String, String> everyone = new HashMap<>(family);
  everyone.putAll(friends); // friends의 항목을 모두 everyone으로 복사
  System.out.println(everyone);
  ```
- `중복된 키가 없다면`, 위 코드는 정상 동작
- 좀 더 유연하게 합치려면 `merge` 메서드를 사용해야 함
  - 이 메서드는, 중복된 키를 어떻게 합칠지에 대한 `BiFunction`을 인수로 받음
  ```java
  Map<String, String> family = Map.ofEntries(
    entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
  Map<String, String> friends = Map.ofEntries(
    entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"));
  
  Map<String, String> everyone = new HashMap<>(family);
  friends.forEach((k, v) -> 
    everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2)); // 중복된 키가 있으면 두 값을 연결
  System.out.println(everyone);
  ```
- javadoc의 `merge` 메서드
  - 지정된 키와 연관된 값이 없거나, 값이 `null`이면
    - 키를 `null`이 아닌 다른 값과 연결 함
  - 아니면 `merge`는 연결된 값을 주어진 매핑 함수의 **결과**로 대치하거나
    - 결과가 `null`이면 항목을 **제거** 함
- `merge`를 이용해 초기화 검사를 수행하기
  - 예시 : `영화를 몇 회 시청했는지 기록하는 맵`
    ```java
    Map<String, Long> moviesToCount = new HashMap<>();
    String movieName = "JamesBond";
    long count = moviesToCount.get(movieName);
    if(count == null) {
      moviesToCount.put(movieName, 1);
    } else {
      moviesToCount.put(movieName, count + 1);
    }

    // merge를 통해 구현
    moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
    ```
    - `merge`의 두번째 인수는 `1L`
    - `javadoc`에 따르면
      - **키와 연관된 기존 값**에 합쳐질 `null`이 아닌 값 또는
      - 값이 없거나 키에 `null`값이 관련되어 있다면
        - 이 값을 키에 연결
    - 키의 반환값이 `null`이므로 처음에는 `1`이 사용됨
      - 그 다음부터는 값이 `1`로 초기화 되어 있으므로, `Bifunction`을 적용해 값이 증가됨

## 8.4. 개선된 ConcurrentHashMap
- `ConcurrentHashMap`
  - 동시성 친화적, 최신 기술을 반영한 `HashMap` 버전 
  - 내부 자료구조의 특정 부분만 잠구어
    - **동시 추가/갱신**작업을 허용함
  - 동기화된 `HashTable` 버전에 비해
    - 읽기 쓰기 권한 성능이 월등함
  - 표준 `HashMap`연산은 **비동기**로 동작

### 8.4.1. 리듀스와 검색
- 세가지 새로운 연산 지원
  - `ForEach` : 각 `key, value`싸엥 주어진 액션을 실행
  - `reduce` : 모든 `key, value` 쌍을 제공된 **리듀스 함수**를 이용해 결과로 합침
  - `search` : `null`이 아닌 값을 반환할 때까지 각 `key, value` 쌍에 함수를 적용
- 다음처럼 네가지 연산 형태 지원
  - `key, value`로 연산 : `forEach, reduce, search`
  - `key` 연산 : `forEachKey, reduceKeys, searchKeys`
  - `value` 연산 : `forEachValue, reduceValues, searchValues`
  - `Map.Entry` 객체로 연산 : `forEachEntry, reduceEntries, searchEntries`
- 위 연산들 모두 `ConcurrentHashMap`의 상태를 **잠그지 않고 수행** 
- 위 연산에 제공한 함수는 계산이 진행되는 동안
  - 바뀔 수 있는 객체, 값, 순서 등에 의존하지 말아야 함
- 이들 연산에 **병렬성 기준값**(threshold)를 지정해야 함
  - **맵의 크기**가 주어진 기준값보다 작다면, **순차적**으로 연산 진행
- 기준값
  - `1`; 공통 `threadpool`을 이용해, **병렬성**을 극대화
  - `Long.MAX_VALUE`; **한 개의 스레드**로 연산
- 소프트웨어 아키텍처가, 고급 수준의 자원 활용 최적화를 사용하고 있지 않다면,
  - **기존값 규칙을 따르는 것이 좋음**
- 예시 : `reduceValues`를 사용한 맵의 최댓값 찾기
  ```java
  ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
  long parallelismThreshold = 1;
  Optional<Integer> maxValue =
                    Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
  ```
- `int, long, double`등의 기본값에는, 전용 `each reduce` 연산이 제공됨
  - `reduceValuesToInt, reduceKeyToLong`을 사용하게 되면 **박싱 작업 필요 없음**
  - 효율적인 처리 가능


### 8.4.2. 계수
- 맵의 **매핑 개수**를 반환하는 `mappingCount`연산 지원
- 기존의 `size` 메서드 대신, 새 코드에서는 `int`를 반환하는 `mappingCount`를 사용하는 것이 좋음
  - 매핑의 개수가 `int`범위를 넘어서는 **이후 상황 대처 가능**

### 8.4.3. 집합뷰
- `ConcurrentHashMap`을 집합 뷰로 반환하는 `KeySet`이라는 새 메서드 제공
- 맵을 바꾸면 집합도 바뀌고,
  - 반대로 집합을 바꾸면 **맵**이 영향을 받음
- `newKeySet`이라는 메서드를 통해
  - `ConcurrentHashMap`으로 유지되는 **집합**을 만들 수 있음

## 8.5. 마치며
- `java 9`는 적의 원소를 포함하며
  - 바꿀 수 없는 리스트, 집합, 맵을 쉽게 만들 수 있도록
  - `List.of, Set.of, Map.of, Map.ofEntiries` 등의 **컬렉션 팩토리 지원**
- 이들 컬렉션 팩토리가 반환한 객체는 **만들어진 다음 변경 불가**
- `List` 인터페이스는 `removeIf, replaceAll, sort` 세 가지 **디폴드 메서드** 제공
- `Set` 인터페이스는 `removeIf` **디폴트 메서드** 제공
- `Map` 인터페이스는
  - 자주 사용하는 패턴과 버그 방지를 위해 다양한 **디폴트 메서드** 제공
- `ConcurrentHashMap`은 `Map`을 상속받은 새 디폴트 메서드를 지원함과 동시에
  - **스레드 안전성**도 제공