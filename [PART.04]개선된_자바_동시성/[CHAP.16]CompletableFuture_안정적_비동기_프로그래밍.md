# [CHAP.16] CompletableFuture: 안정적 비동기 프로그래밍

## 16.1. Future의 단순 활용
- 비동기 계산 모델링시 `Future` 활용 가능
  - 계산이 끝났을 경우, **결과**에 접근할 수 있는 참조 제공
- `Future`는 저수준의 스레드에 비해 **직관적**으로 이해하기 쉽다는 장점 존재
- `Future`를 이용하려면
  - 시간이 오래 걸리는 작업을 `Callable` 객체 내부로 감싼 다음
  - `ExecutorService`에 제출해야 함

#### CODE.16.1. Future로 오래걸리는 작업을 비동기적으로 실행하기
- Java 8 이전의 코드
```java
ExecutorService executor = Executors.newCachedThreadPool(); // ThreadPool에 task 제출시 ES 필요
Future<Double> future = executor.submit(new Callable<Double>() { // Callable을 ES로 제출
    public Double call() {
        return doSomeLongComputation(); // 시간이 오래 걸리는 작업을 다른 스레드에서 비동기적으로 수행
    }
});
doSomethingElse(); // 비동기 작업을 수행하는 동안, 다른 작업 수행
try {
    // 비동기 작업의 결과를 가져옴. 결과가 준비되어 있지 않으면 호출 스레드 블록
    // 최대 1초까지만 기다림
    Double result = future.get(1, TimeUnit.SECONDS); 
} catch (ExecutionException ee) {
    // 계산 중 예외 발생
} catch (InterruptedException ie) {
    // 현재 스레드에서 대기 중 인터럽트 발생
} catch (TimeoutException te) {
    // Future가 완료되기 전 타임아웃 발생
}
```
- ES에서 제공하는 스레드가 **시간이 오래걸리는 작업**을 처리하는 동안
  - 우리 스레드로, 다른 작업을 **동시** 실행 
- 다른 작업을 처리하다가, 오래 걸리는 작업의 결과가 필요한 시점에
  - `Future.get`으로 데이터를 가져올 수 있음
    - 결과가 있다면 즉시 반환, 작업이 준비되지 않았다면 기다림
- `get` 메서드를 오버로드 하여 **스레드가 대기할 최대 타임아웃 시간 지정 필요**

### 16.1.1. Future 제한
- 여러 `Future` 결과가 있을 때, 이들의 의존성을 표현하기는 어려움
- 선언형 기능이 있다면 유용
  - 두 개의 **비동기 계산 결과**를 하나로 합침
  - 서로 독립적일 수 있으며
    - 또는 두번째 결과가 첫번째 결과에 **의존**하는 상황일 수 있음
  - `Future` 집합이 실행하는 모든 task의 완료를 기다림
  - `Future` 집합에서 가장 빨리 완료되는 task를 기다렸다가 결과를 얻음
  - 프로그램적으로 `Future`를 완료
    - **비동기 동작**에 **수동**으로 결과 제공
  - Future 완료 동작에 반응
    - 결과를 기다리면서, **블록되지 않음**
    - 결과가 준비되었다는 알림을 받은 이후
      - `Future`의 결과로 원하는 추가 동작 수행

### 16.1.2. CompletableFuture로 비동기 어플리케이션 만들기
- 고객에게 비동기 API를 제공하는 방법
- 동기 API를 사용해야할 때, 코드를 **비블록**으로 만드는 방법
  - 두 개의 **비동기 동작**을 파이프라인으로 만드는 방법
  - 두 개의 동작 결과를 **하나의 비동기 계산**으로 합치는 방법
- **비동기 동작의 완료**에 대응하는 방법

#### 동기 API와 비동기 API
- 동기 API를 호출하는 상황을 **블록 호출**(blocking call)
- **비동기 API**에서는 메서드가 **즉시 반환**
  - 끝내지 못한 나머지 작업을 **호출자 스레드**와 동기적으로 실행되도록, 다른 스레드에 할당
- 이를 **비블록 호출**(non-blocking call)
- 다른 스레드에 할당된 나머지 계산 결과는
  - **콜백 메서드**를 호출해서 전달하거나
  - 호출자가 **계산 결과가 끝날때까지 기다림** 메서드를 추가로 호출하면서 전달
- 주로 `I/O System Programming`에서 이와같은 방식으로 동작 수행
  - **계산 동작**을 수행하는 동안 **비동기적**으로 **디스크 접근**을 수행
  - 더 이상 수행할 동작이 없으면 **디스크 블록**이 **메모리**로 로딩될때까지 기다림