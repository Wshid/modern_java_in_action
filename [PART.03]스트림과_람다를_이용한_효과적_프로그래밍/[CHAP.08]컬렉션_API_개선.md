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