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