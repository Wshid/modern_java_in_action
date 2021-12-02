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
- 이를 제대로 설계한다면
  - 