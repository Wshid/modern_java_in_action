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

### 4.2. 스트림 시작하기
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