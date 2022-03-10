# [CHAP.11] null 대신 Optional 클래스

### 이 장의 주요 내용
- `null` 참조의 문제점 / `null`을 멀리해야하는 이유
- `null` 대신 `Optional`: `null`로부터 안전한 도메인 모델 재구현하기
- `Optional` 활용: `null` 확인 코드 제거하기
- `Optional`에 저장된 값을 확인하는 방법
- 값이 없을 수도 있는 상황을 고려하는 프로그래밍

## 11.1. 값이 없는 상황을 어떻게 처리할까?

#### CODE.11.1. Person/Car/Insurance 데이터 모델
```java
public class Person {
  private Car car;
  public Car getCar() {
    return car;
  }
}

public class Car {
  private Insurance insurance;
  public Insurance getInsurance() {
    return insurance;
  }
}

public class Insurance {
  private String name;
  public String getName()
}
```
- 아래 코드 실행시 문제 발생 가능
  ```java
  public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
  }
  ```
  - `person == null`, `Person::getInsurance() == null`일 수 있음

### 11.1.1. 보수적인 자세로 `NullPointerException` 줄이기

#### CODE.11.2. null 안전 시도 1: 깊은 의심
```java
public String getInsuranceName(Person person) {
  if (person != null) { // null 코드 확인으로, 나머지 호출 체인의 들여쓰기 수준 증가
    Car car = person.getCar();
    if (car != null) {
      Insurance insurance = car.getInsurance();
      if (insurance != null) {
        return insurance.getName();
      }
    }
  }
  return "Unknown";
}
```
- `null`에 대한 의심으로, **변수 접근**시 마다 `if`가 추가되면서 **들여쓰기 수준 증가**
- 이와 같은 **반복 패턴**(recurring pattern) 코드를 **깊은 의심**(deep doubt)라고 함
  - 코드의 구조가 엉망이되고, **가독성이 떨어짐**

#### CODE.11.3. null 안전 시도 2: 너무 많은 출구
```java
public Stringg getInsuranceName(Person person) {
  if (person == null) {
    return "Unknown";
  }
  Car car = person.getCar();
  if (car == null) {
    return "Unknown";
  }
  Insurance insurance = car.getInsurance();
  if (insurance == null) {
    return "Unknown";
  }
  return insurance.getName();
}
```
- 메서드에 4개의 출구 존재
  - 출구때문에 **유지보수가 어려워짐**
- `null`이 반환되는 기본값의 중복으로,
  - 같은 문자열 반복으로 오타등의 실수 발생 가능 

## 11.1.2. null 때문에 발생하는 문제
- **에러의 근원**
  - NPE
- **코드가 어지러움**
  - 중첩된 `null`확인 코드
- **아무 의미가 없음**
  - 더군다나 **정적 형식 언어**에서 `값이 없음을 표현하는 방법`으로 부적합
- **자바 철학 위배**
  - 자바는 개발자로부터, 모든 포인터를 숨김
  - 단, 예외가 있는데, 그것이 `null`
- **형식 시스템에 구멍을 만듦**
  - `null`은 무형식이며, 정보를 포함하고 있지 않음
    - 모든 참조 형식에 `null`을 할당할 수 있음
  - `null`이 이런식으로 할당되면서,
    - 시스템의 다른부분으로 `null`이 퍼졌을때, `null`이 어떤 의미를 지녔는지 알 수 없음

## 11.1.3. 다른 언어는 `null` 대신 무얼 사용하나?
- `groovy`에서는 **안전 네비게이션 연산자**(Safe Navigation Operator `?.`)를 도입
  ```groovy
  def carInsuranceName = person?.car?.insurance?.name
  ```
  - `person, car, insurance`별로 `null`을 가질 수 있음
  - `null`참조 예외 걱정 없이 객체 접근 가능
  - 호출 체인에 `null`인 참조가 있으면, `null`이 반환됨
- `Haskell, Scala`등의 함수형 언어에서는 아예 다른 관점에서 `null`을 접근
- `Haskell`
  - `optional value`를 저장할 수 있는 `Maybe`형식 제공
  - `Maybe`: 주어진 형식의 값을 갖거나, 아무 값도 갖지 않을 수 있음
- `Scala`
  - `Option[T]` 구조 제공
    - `T`형식의 값을 갖거나, 아무 값도 갖지 않을 수 있음
  - `Option` 형식에서 제공하는 연산을 사용해서
    - 값이 있는지 여부를 **명시적으로 확인해야 함**
  - 형식 시스템에서 이를 강제하므로, `null`과 관련 문제 회피가 가능
- `Java 8`
  - `java.util.Optional<T>`형식 사용