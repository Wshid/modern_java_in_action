# [CHAP.04] 스트림 소개
- **컬렉션**
  - 데이터를 그룹화하고 처리 가능
  - 많은 요소를 처리하려면?
    - 멀티코어 아티텍처를 활용해서 **병렬**로 컬렉션 요소 처리 필요
- 스트림으로 간략히 처리 가능

## 4.1. 스트림이란 무엇인가?
- 스트림 : java 8 api에 새로 추가된 기능
- 스트림을 이용하면 **선언형**으로 **컬렉션 데이터**를 처리할 수 있음
  - 선언형 : 데이터를 처리하는 임시 구현 코드 대신, 질의로 표현
- 스트림을 이용하면
  - **멀티스레드 코드**를 구현하지 않아도 데이터를 **투명하게** 병렬로 처리 가능
- java 8을 활용한 코드
  ```java
  import static java.util.Comparator.comparing;
  import static java.util.stream.Collectors.toList;
  List<String> lowCaloricDishesName =
                    menu.stream()
                        .filter(d -> d.getCalories() < 400)
                        .sorted(comparing(Dish::getCalories))
                        .map(Dish::getName)
                        .collect(toList());

  // 멀티코어 아키텍처에서 병렬로 실행
  List<String> lowCaloricDishesName =
                    menu.parallelStream()...
  ```
- 스트림의 이점
  - **선언형**으로 코드 작성 가능
    - `loop`나 `if`문을 사용하지 않고, 동작만 지정 가능
    - 람다 표현식을 활용할 수 있음
  - `filter`, `sorted`, `map`, `collect` 같은 **여러 빌딩 블록 연산**을 연결하여
    - **복잡한 데이터 처리 파이프라인**을 만들 수 있음
- `filter`, `sorted`, `map`, `collect` 같은 연산은
  - **고수준 빌딩 블록**(high-level building block)으로 이루어져 있으므로
    - 특정 스레딩 모델에 **제한되지 않음**
    - 자유롭게 어떤 상황이든 사용할 수 있음
  - 데이터 처리과정을 **병렬화**하면서
    - 스레드의 **락**을 걱정할 필요가 없음
- 스트림 api는 매우 **비싼 연산**
  ```java
  Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
  ```
- Guava, Apache, LambdaJ
  - 컬렉션을 제어하는데 도움이 되는 다양한 라이브러리
  - Guava의 경우 `Multimap`, `Multiset`과 같은 추가 컨테이너 클래스 제공
  - Apache의 경우 `Apache Commons Collections` 라이브러리 제공
  - LambdaJ는 **함수형 프로그래밍**에서 영감을 받은
    - **선언형**으로 **컬렉션**을 제어하는 다양한 유틸리티 제공

### Stream API 특징 요약
- **선언형** : 간결, 가독성
- **조립할 수 있음** : 유연성이 좋아짐
- **병렬화** : 성능이 좋아짐

## 4.2. 스트림 시작하기
- **컬렉션 스트림**
  - java 8 컬렉션에서 **스트림**을 반환하는 `stream` 메서드 제공
  - `java.util.stream.Stream`참고
    - 숫자 범위나 `I/O`자원에서, 스트림 요소를 만드는 등
      - stream 메서드 이외에도, 다양한 방법으로 **스트림**을 얻을 수 있음
- **스트림**이란
  - 데이터 처리 연산을 지원하도록 **소스**에서 추출된 **연속된 요소**(Sequence of elements)
    - **연속된 요소**
      - 스트림은 **특정 요소 형식**으로 이루어진 **연속된 값 집합**의 인터페이스
      - 컬렉션의 경우
        - 시간과 공간의 복잡성과 관련된 **요소 저장** 및 **접근 연산**이 주를 이룬다
      - 스트림의 경우
        - `filter`, `sorted`, `map`과 같은 **표현 계산식**이 주를 이룬다
      - **컬렉션**의 주제는 **데이터**고, **스트림**의 주제는 **계산**이다
    - **소스**
      - 스트림은 `컬렉션`, `배열`, `I/O자원` 등의 **데이터 제공 소스**로부터 데이터를 소비
      - 정렬된 컬렉션으로 **스트림**을 생성하면, **정렬이 그대로 유지**
      - 리스트로 스트림을 만들면, 스트림의 요소는 **리스트의 요소와 같은 순서**
    - **데이터 처리 연산**
      - 함수형 프로그래밍 언어에서 일반저긍로 지원하는 연산과 **데이터베이스**와 비슷한 연산을 지원
      - `filter, map, reduce, find, match, sort`등으로 데이터 조작 가능
      - 데이터를 **순차적**으로 또는 **병렬**로 실행할 수 있음

#### 주요 특징(2)
- **파이프라이닝**(pipelining)
  - 대부분의 스트림 연산은, 스트림 연산끼리 **연결**해서 **커다란 파이프라인**을 구성할 수 있도록 스트림 자산 반환
  - 덕분에 `laziness`, `shirt-circuiting`과 같은 최적화도 얻을 수 있음
  - 연산 파이프라인은 **데이터 소스**에 적용하는 db의 질의와 유사
- **내부 반복**
  - 반복자를 명시해서 반복하는 **컬렉션**과 달리, **내부 반복**을 지원

#### 예제
```java
import static java.util.stream.Collectors.toList;
List<String> threeHighCaloricDishNames =
    menu.stream() // list -> stream
        .filter(dish -> dish.getColories() > 300)
        .map(Dish::getName)
        .limit(3) // 스트림 크기 축소
        .collect(toList());
```
- 데이터 소스로 `요리 리스트`를 가져옴
  - 연속된 요소를 스트림에 제공
- 일련의 데이터 처리 연산을 적용
- `collect` 연산으로 파이프라인을 처리하여 **결과**를 반환
  - 리스트 반환
  - `collect`를 호출하기 전까지, `menu`는 무엇도 선택되지 않으며, 출력 결과도 없음
  - `collect`호출 전까지 **메서드 호출**이 **저장**되는 효과
- `toList`
  - 스트림을 **리스트**로 변환 지시

## 4.3. 스트림과 컬렉션
- 자바의 기존 **컬렉션**과 **스트림** 모두
  - **연속된 요소** 형식의 값을 저장하는 자료구조 인터페이스 제공
- 연속된(`sequenced`)
  - 순서와 상관없이 아무 값에나 접속하는 것이 아니라
  - **순차적**으로 접근함을 의미

### 컬렉션과 스트림의 차이?
- 데이터를 **언제** 계산하느냐
- 컬렉션
  - 현재 자료구조가 포함하는 **모든 값**을 메모리에 저장하는 자료구조
  - 컬렉션에 추가하기 전에 계산되어야 함
  - 컬렉션에 요소를 **추가**하거나 **제거**가 가능함
    - 이런 연산 수행시마다, 컬렉션의 모든 요소를 **메모리**에 저장해야하며
    - 컬렉션에 추가하려는 요소는 **미리 계산**되어야 함
  - 적극적으로 생성(`supplier-driven`)
- 스트림
  - **요청할 때만 요소 계산**
  - 요소를 추가하거나 제거할 수 없음
  - 사용자의 입장에서 요청하는 값만 추출하는 특징을 알 수 없음
  - **생산자**와 **소비자** 관계를 형성
  - **게으르게 만들어지는 컬렉션**
    - demand-driven manufacturing / just-in-time manufacturing

### 4.3.1. 딱 한 번만 탐색할 수 있음
- **반복자**와 마찬가지로, **스트림**도 한번만 탐색 가능
  - 탐색된 스트림의 요소는 **소비**됨
- 한번 탐색한 요소를 다시 탐색하려면,
  - src로부터 새로운 스트림을 생성해야 함
- 코드
  ```java
  List<String> title = Arrays.asList("java8", "in", "action");
  Stream<String> s = title.stream();
  s.forEach(System.out::println); // 요소 정상 소비
  s.forEach(System.out::println); // java.lang.IllegalStateException 발생
  ```

### 4.3.2. 외부 반복과 내부 반복
- 컬렉션 인터페이스를 사용하려면
  - 사용자가 **직접 요소**를 반복 해야함 -> `for-each`
  - 이를 **외부 반복(external-iteration)**
- 스트림 라이브러리는
  - 반복을 알아서 처리하며, 결과 스트림 값을 어딘가에 저장해주는
  - **내부 반복**(internal iteration)을 사용
  - 코드
    ```java
    List<String> names = menu.stream()
                              .map(Dish::getName)
                              .collect(toList()); // 파이프라인을 실행한다(반복자 필요 x)
    ```
- 내부 반복이 좋은 이유
  - 작업을 **투명**하게 **병렬**로 처리하거나
  - 더 **최적화**된 다양한 순서로 처리할 수 있음
- java 8에서의 내부 반복은
  - **데이터 표현**과 **하드웨어**를 활용한 **병렬성 구현**을 자동으로 선택
- `for-each`를 이용하는 외부 반복에서는 **병렬성을 스스로 관리**해야 함
  - `synchronized` 사용 등

## 4.4. 스트림 연산
- `java.util.stream.Stream` 인터페이스는 많은 연산을 정의
- 연결 할 수 있는 스트림 연산을 **중간 연산**(intermediate operation)
- 스트림을 닫은 연산을 **최종 연산**(terminal opertaion)

### 4.4.1. 중간 연산
- `filter`와 `sorted`와 같은 **다른 스트림**을 반환
- 여러 연산을 연결하여 질의를 만들 수 있음
- **단말 연산**을 스트림 파이프라인에서 실행하기 전까지는
  - **아무 연산도 수행하지 않음**
  - `lazy`
- 디버깅을 추가한 코드
  ```java
  List<String> names =
    menu.stream()
        .filter(dish -> {
          System.out.println("filtering:" + dish.getName());
          return dish.getCalories() > 300;
        })
        .map(dish -> {
          System.out.println("mapping:" + dish.getName());
          return dish.getName();
        })
        .limit(3)
        .collect(toList());
  System.out.println(names);
  ```
  - 실제 내용 출력시, `filtering: .. mapping: .. filtering ..`과 같이 순차적으로 실행
- `short circuit`
  - `limit`을 통하여 제한된 수만 반복
- `loop fusion`
  - `filter`와 `map`이 병합되어 수행

### 4.4.2. 최종 연산
- **스트림 파이프라인**에서 **결과**를 도출한다
- 최종 연산에 의해 `List`, `Integer`, `void` 등 **스트림 이외**의 결과가 반환

### 4.4.3. 스트림 이용하기
- 스트림 이용 과정
  - 질의를 수행할 (컬렉션 같은) **데이터 소스**
  - 스트림 파이프라인을 구성할 **중간 연산** 연결
  - 스트림 파이프라인을 실행하고 **결과**를 만들 **최종 연산**
- 스트림 파이프라인의 개념은 `builder pattern`과 유사
  - 호출을 연결해서 설정을 만든이후 `build` 메서드 호출
- 중간 연산 : `filter, map, limit, sorted, distinct`
- 최종 연산 : `forEach, count, collect`