# [CHAP.07] 병렬 데이터 처리와 성능
- 외부 반복을 **내부 반복으로** 바꾸면,
  - native java lib이 stream 요소의 처리를 제어할 수 있음
- 컴퓨터의 멀티코어를 이용하여 파이프라인 연산을 처리할 수 있음
- java 7의 경우 `fork-join framework`를 활용

## 7.1. 병렬 스트림
- 컬렉션에 `parallelStream`을 호출하면 **parallel stream**이 생성됨
- 각 스레드로 처리할 수 있도록
  - 스트림 요소를 `chunk` 단위로 분할한 스트림
- 숫자 `n`을 받아 `1~n`의 합계를 반환하는 메서드
  ```java
  // 순차 실행
  public long sequentialSum(long n) {
    return Stream.iterate(1L, i -> i +1) // 무한 자연수 스트림
                  .limit(n)
                  .reduce(0L, Long::sum); // 모든 숫자를 더하는 스트림 리듀싱 연산
  }

  // for 사용
  public long iterativeSum(long n) {
    long result = 0;
    for (long i = 1L; i <= n; i++) {
      result += i;
    }
    return result;
  }
  ```

### 7.1.1. 순차 스트림을 병렬 스트림으로 변환하기
- 순차 스트림에 `parallel` 메서드를 호출하면 병렬 처리 가능
  ```java
  public long parallelSum(long n) {
    return Stream.iterate(1L, i -> i +1)
                  .limit(n)
                  .parallel() // 스트림을 병렬 스트림으로 전환
                  .reduce(0L, Long::sum);
  }
  ```
- 스트림이 여러 `chunk`단위로 분할되며,
  - 리듀싱 연산을, 여러 청크에 걸쳐 **병렬 수행**
  - 이후 마지막으로 다시 합치는 연산 수행
- `sequential`을 사용하면, 순차로 수행 가능
  ```java
  stream.parallel()
        .filter(...)
        .sequential(...)
        .map(...)
        .parallel() // 마지막 호출이 parallel이므로, 이 파이프라인은 병렬 수행
        .reduce();
  ```
  - `parallel()`과 `sequential()` 메서드중, 가장 **마지막에 호출된 메서드**가
    - 전체 파이프라인에 영향을 미침

#### 병렬 스트림에서 사용하는 스레드 풀 설정
- 병렬 스트림은 내부적으로 `ForkJoinPool`을 사용
- 기본적으로 **프로세서 수**, `Runtime.getRuntime().availableProcessors()`가 반환하는 값에 맞게 스레드를 가짐
- 아래와 같이 코드로 설정 가능
  ```java
  // 전역적으로 설정, 모든 병렬 스트림 연산에 영향
  // 하나의 병렬 스트림에 개별 설정은 불가능 함
  System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "12");
  ```

### 7.1.2. 스트림 성능 측정
- 병렬화를 이용하면 `순차`나 `반복`형식에 비해 성능이 좋아질 것이라고 추측
- `JMH`(Java Microbenchmark Harness) 라이브러리를 사용하여 작은 벤치마크 구현
- JVM으로 실행되는 프로그램을 벤치마크하는 것은 어려운 작업
  - `Hotspot`이 바이트 코드를 최적화 하는데 필요한 **준비(warm-up)** 시간
  - **가비지 컬렉터로 인한 Overhead** 등의 여러 요소 고려 필요
- `maven`을 사용할 경우, `pom.xml`에 의존성을 추가하여 **JMH** 사용 가능
  ```java
  org.openjdk.jmh / jmh-core // 핵심 JMH
  org.openjdk.jmh / jmk-generator-annprocess // jar 파일을 만드는데 도움을 주는 annotation processor 포함
  ```

#### 예제 7.1. n개의 숫자를 더하는 함수의 성능 측정
- 코드
  ```java
  @BenchmarkMode(Mode.AverageTime) // 벤치마크 대상 메서드를 실행하는데 걸린 평균 시간 측정
  @OutputTimeUnit(TimeUnit.MILLISECONDS) // 벤치마크 결과를 ms단위로 출력
  @Fork(2, jvmArgs={"-Xms4G", "-Xmx4G"}) // 4G 힙공간을 제공한 환경에서 2번 수행
  public class ParallelStreamBenchmark {
    private static final long N=10_000_000L;

    @Benchmark // 벤치마크 대상 메서드
    public long sequentialSum() {
      return Stream.iterate(1L, i -> i+1).limit(N)
                                          .reduce(0L, Long::sum);
    }

    @TearDown(Level.Invocation) // 매번 벤치마크를 실행한 다음에는 `GC` 동작시도
    public void tearDown() {
      System.gc();
    }
  }
  ```
- 컴파일 이후 다음과 같이 수행
  ```bash
  java -jar ./target/benchmarks.jar ParallelStreamBenchmark
  ```
- `GC`의 영향을 받지 않도록, `힙의 크기를 충분히 크게 설정`했을 뿐 아니라
  - 매번 벤치마크 이후 `gc`를 수행하도록 강제
  - 이럼에도 결과는 정확하지 않을 수 있음
    - `기계가 지원하는 코어의 갯수 등..`
- `JMH` 명령은
  - 핫스팟이 코드를 최적화 할 수 있도록 `20`번을 수행하면서 벤치마크 준비 이후
  - `20`번을 더 시행해 **최종 결과**를 계산
- 특정 어노테이션이나 `-w`, `-i` 플래그를 명령행에 추가하여, 기본 동작 횟수를 조절할 수 있음
- 결과
  - 전통적인 `for loop`가 저수준으로 동작, 기본값 박싱/언박싱 비용 x
    - 빠를 것
  - 병렬 처리가 순차 처리보다 느린 결과를 보임
- 병렬 버전의 문제점
  - 반복 결과로 **박싱된 객체**가 만들어지므로, 숫자를 더하려면 **언박싱**이 필요
  - 반복 작업은 **병렬로 수행할 수 있는 독립 단위**로 나누기가 어려움
    - 이전 결과의 연산에 따라, 다음 결과 연산의 입력이 달라지는 구조이기 때문
- 리듀싱 과정을 시작하는 시점에 **전체 숫자 리스트**가 준비되지 않으므로
  - `스트림을 병렬 처리할 수 있도록 청크 분할 x`

#### 더 특화된 메서드 사용
- `LongStream.rangeClosed`라는 메서드 사용해보기
  - 기본형 `long`을 사용 // **박싱/언박싱 오버헤드 x**
  - 쉽게 청크로 분할 할 수 있는 **숫자 범위를 생산**
    - `1-20`범위의 숫자를 각각 `1-5`, `6-10`, `10-15`, `16-20`으로 분할 가능
- 실제로 훨씬 빠른 결과를 보임
- 코드
  ```java
  @Benchmark
  public long rangedSum() {
    return LongStream.rangeClosed(1, N).reduce(0L, Long::sum);
  }

  // 병렬 스트림을 추가한 코드
  // 실제 순차실행보다 빠른 성능을 보임
  @Benchmark
  public long rangedSum() {
    return LongStream.rangeClosed(1, N).parallel().reduce(0L, Long::sum);
  }
  ```

### 7.1.3. 병렬 스트림의 올바른 사용법
- 병렬 스트림의 문제는
  - **공유된 상태를 바꾸는 알고리즘의 사용** 때문에 일어남
- n까지의 자연수를 더하면서 **공유된 누적자**를 바꾸는 프로그램
  ```java
  public long sideEffectSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add);
    return accumulator.total;
  }

  public class Accumulator {
    public long total = 0;
    public void add(long value) {total += value;}
  }
  ```
  - 누적자를 초기화 하고, 리스트의 각 요소를 하나씩 탐색하면서 누적자에 숫자 추가
- 본질적으로 **순차 실행**에 맞게 구현되어 있기 때문에
  - **병렬로 실행하면 문제**
  - `total` 접근 시, `race condition` 문제 발생 가능
  - 동기화로 해결하려면, 병렬화 하려는 특징이 사라짐
- `total += value`는 `atomic operation`이 아님

### 7.1.4. 병렬 스트림 효과적으로 사용하기
- `천 개 이상의 요소가 있을 때만 병렬 스트림을 사용하라`와 같은 **양의 기준**은 **적절하지 않음

#### 병렬 스트림을 사용할 때 고려할 사항
- **확신이 서지 않으면 직접 측정하라**
  - 벤치마크로 성능 측정하기
- **박싱을 주의하라**
  - `IntStream, LongStream, DoubleStream`과 같은 기본형 특화 스트림 사용
- **순차 스트림 보다 병렬 스트림에서 성능이 떨어지는 연산 존재**
  - `limit`, `findFirst`처럼, **순서에 의존하는 연산**
  - `findAny`의 경우 순서와 연관이 없음
  - 정렬된 스트림에 `unordered`를 호출하면 **비정렬된 스트림**을 얻을 수 있음
  - 요소의 순서가 상관 없을 경우, 비정렬된 스트림에 `limit`를 하는것이 효과적
- **스트림에서 수행하는 전체 파이프 라인 연산 비용 고려**
  - 처리할 요소가 `N`, 하나 요소 처리시 비용 `Q` => `N*Q`만큼의 비용
  - `Q`가 높아진다는 것은, **병렬 스트림**으로 성능 개선 가능
- **소량의 데이터에서는 병렬 스트림이 도움이 되지 않음**
- **스트림을 구성하는 자료구조가 적절한지 확인**
  - `ArrayList`가 `LinkedList`보다 효율적인 분산 가능
    - `LinkedList`는 분할시 **모든 요소**탐색이 필요
    - `ArrayList`는 탐색 없이도 리스트 분할 가능
  - `range` 팩토리 메서드로 만든 **기본형 스트림**도 쉽게 분해 가능
  - `custom Spliterator`로 분해과정 제어 가능
- **스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라짐**
  - `SIZED` 스트림은 정확히 같은 크기로 분할 가능 => 효과적으로 병렬 처리 가능
  - `filter`연산이 있을 경우, **스트림 연산의 길이 예측 x**
    - 스트림을 병렬 처리할 수 있을지 모름
- **최종 연산의 병합 과정(Collector::combiner) 비용을 확인하기**
  - 병합 과정의 비용이 비쌀 경우, 스트림으로 얻은 이익이 상쇄됨
- **병렬 스트림이 수행되는 내부 인프라 구조 확인하기**
  - java 7의 `fork-join framework`로 **병렬 스트림**이 처리

#### 스트림 소스와 분해성
**소스**|**분해성**
:-----:|:-----:
ArrayList|3
LinkedList|1
IntStream.range|3
Stream.iterate|1
HashSet|2
TreeSet|2


## 7.2. 포크/조인 프레임워크
- 포크/조인 프레임워크는
  - **병렬화**할 수 있는 작업을 **재귀적**으로 작은 작업으로 분할
  - 이후 **서브테스크** 각각의 결과를 **합쳐서** 전체 결과로 만듦
- 서브 태스크를 스레드 풀(ForkJoinPool)의 **작업자 스레드**에
  - 분산 할당하는 `ExcecutorService` 인터페이스를 구현

### 7.2.1. RecursiveTask 활용
- 스레드 풀을 이용하려면 `RecursiveTask<R>`의 **서브 클래스**를 생성해야 함
- `R`은 병렬화 된 태스크가 생성하는 **결과 형식** 또는
  - **결과가 없을 때**(결과가 없더라도, 다른 비지역 구조를 바꿀수 있음)
  - 이 때는 `RecursiveAction` 형식
- `RecursiveTask`를 정의하려면
  - 추상 메서드 `compute`를 구현해야 함 
    ```java
    protected abstract R compute();
    ```
- `compute` 메서드는
  - 태스크를 **서브태스크**로 분할하는 로직과
  - 더 이상 분할 할 수 없을 때 **개별 서브태스크의 결과를 생산**할 알고리즘 정의
- 다음과 같은 의사코드를 따름
  ```java
  if(태스크가 충분히 작거나, 분할 불가능 하다면) {
    순차적으로 태스크 계산
  } else {
    태스크를 두 서브태스크로 분할
    태스크가 다시 서브태스크로 분할되도록 메서드 재귀적 호출
    모든 서브태스크의 연산이 완료될 때까지 기다림
    각 서브태스크의 결과를 합침
  }
  ```
- 위 알고리즘은 `devide-and-conquer`의 병렬화 버전
- `ForkJoinSumCalculator` 코드에서 보여주듯
  - `RecursiveTask`를 구현해야 함

#### CODE.7.2. 포크/조인 프레임워크를 이용해서 병렬 합계 수행
```java
public class ForkJoinSumCalculator extends java.util.concurrent.RecusiveTask<Long> {
  private final long[] numbers;
  private final int start;
  private final int end;
  private static final long THRESHOLD = 10_000; // 이 값 이하의 서브 태스크는 더 이상 분할 x

  public ForkJoinSumCalculator(long[] numbers) { // 메인 태스크를 생성할 때 사용할 공개 생성자
    this(numbers, 0, numbers.length);
  }

  private ForkJoinSumCalculator(long[] numnbers, int start, int end) { // 메인 태스크의 서브태스크를 재귀적으로 만들 때 사용하는 비공개 생성자
    this.numbers = numbers;
    this.start = start;
    this.end = end;
  }

  @Override
  protected Long compute() { // RecursiveTask의 추상 메서드 override
    int length = end - start;
    if (length <= THRESHOLD) {
      return computeSequentially(); // 기준값과 같거나 작으면, 순차적으로 결과 계산
    }

    // 배열의 첫번째 절반을 더하도록 서브태스크 생성
    ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2);
    // ForkJoinPool의 다른 스레드로 새로 생성한 태스크를 비동기로 실행
    leftTask.fork();
    
    // 배열의 나머지 절반을 더하도록 서브태스크 생성
    ForkJoinSumCalculator rightTask = new forkJoinSumCalculator(numbers, start + length/2, end);
    // 두번째 서브 태스크를 동기 실행, 이 때 추가로 분할 가능성 존재
    Long rightResult = rightTask.compute();
    Long leftResult = leftTask.join(); // 첫번째 서브태스크의 결과를 읽거나, 결과가 없을 경우 기다림
    return leftResult + rightResult;
  }

  // 더 분할할 수 없을 때 서브태스크의 결과를 계산하는 단순한 알고리즘
  private long computeSequentially() {
    long sum = 0;
    for (int i = start; i < end; i ++) {
      sum += numbers[i];
    }
    return sum;
  }
}
```
- 다음 코드 처럼 `ForkJoinSumCalculator`의 생성자로, 원하는 수의 배열을 넘길 수 있음
  ```java
  public static long forkJoinSum(long n) {
    long[] numbers = LongStream.rangeClosed(1,n).toArray();
    ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
    return new ForkJoinPool().invoke(task);
  }
  ```
  - `LongStream`으로 `n`까지의 자연수를 포함하는 배열 생성
  - 마지막으로 생성한 태스크를 새로운 `ForkJoinPool::invoke`로 전달
  - 마지막 `invoke` 메서드의 반환 값은 `ForkJoinSumCalculator`에서 정의한 태스크가 결과

#### ForkJoinPool의 사용
- 일반적으로 app에서는 **둘 이상의 ForkJoinPool을 사용하지 ㅇ낳음**
- 필요한 곳에서 언제든 가져다 쓸 수 있도록
  - `ForkJoinPool`을 한번만 **인스턴스화**하여
  - **정적 필드에 싱글턴**으로 저장
- `ForkJoinPool`을 만들면서
  - 인수가 없는 **디폴트 생성자**를 사용했는데,
  - 이는 `JVM`에서 이용할 수 있는 모든 `processor`가 자유롭게 **pool**에 접근 가능함을 의미
- `Runtime.availableProcessors`의 반환값으로 **풀에 사용될 스레드 수를 결정**
  - 실제 프로세서 외에 **하이퍼 스레딩**과 관련된 **가상 프로세서 개수**도 포함

#### ForkJoinSumCalculator 실행
- `ForkJoinSumCalculator`를 `ForkJoinPool`로 전달하면
  - 풀의 스레드가 `ForkJoinSumCalculator`의 `compute` 메서드를 실행하면서 작업 수행
- `compute` 메서드는 **병렬 실행이 가능할 만큼** 태스크가 작아졌는지 확인하며,
  - 태스크의 크기가 크다고 판단되면, 두 개의 새로운 `ForkJoinSumCalculator`로 할당
  - 그런 이후, `ForkJoinPool`이 새로 생성된 `ForkJoinSumCalculator`를 실행
- 위 과정이 재귀적으로 반복 하면서, 주어진 조건에 만족할 때까지 태스크 분할
- 각 서브태스크는 순차적으로 처리되며,
  - 포킹 프로세스로 만들어진 **이진 트리**의 태스크를 **루트에서 역순으로 방문**
- 서브 태스크의 부분결과를 합쳐 태스크의 최종 결과 계산
- 성능 측정 방법
  ```java
  // 하니스를 사용한 포크/조인 프레임워크의 합계 메서드 성능 측정
  System.out.println("ForkJoin sum done in : " 
      + measureSumPerf(ForkJoinSumCalculator::forkJoinSum, 10_000_000) + "msecs");
  ```
- 실제 코드 수행시 **병렬 스트림**을 수행할 때보다 성능이 나쁜데,
  - 이는 `ForkJoinSumCalculator` 태스크에서 사용할 수 있도록
  - 전체 스트림을 `long[]`으로 변환했기 때문

### 7.2.2 Fork/Join Framework를 제대로 사용하는 방법
- 쉽게 사용할 수는 있는 방식이나, **주의**해서 사용
- 효과적으로 사용하는 방법
  - `join` 메서드를 **태스크**에 호출하면
    - 태스크가 생산하는 **결과**가 준비될 때까지 **호출자 블록**
    - 두 서브태스크가 모두 시작된 다음에 `join`을 호출해야 함
  - `RecusiveTask`내에서는 `ForkJoinPool::invoke` 메서드를 사용하지 않아야 함
    - 대신 `compute`, `fork` 메서드를 직접 호출할 수 있음
    - 순차 코드에서 **병렬 계산**을 시작할 때만 `invoke`를 사용
  - 서브 태스크에 `fork` 메서드를 호출해서 `ForkJoinPool`의 일정 조절 가능
    - 왼쪽/오른쪽 작업에 모두 `fork`를 호출하지 않고,
    - 한쪽 작업에는 `compute`를 호출하는 것이 효율적
    - 두 서브 태스크의 **한 태스크**에는
      - 같은 스레드를 **재사용**할 수 있기 때문에
      - 풀에서 불필요한 태스크를 할당하는 **오버헤드 회피** 가능
  - `fork/join framework`를 이용하는 **병렬 계산**은 **디버깅 하기 어려움**
    - `fork`라 불리는 다른 스레드에서 `compute`를 호출하기 때문에 `StackTrace`가 의미 없음
  - 멀티 코어에 `fork/join framework`를 사용하는 것이, 무조건 순차 처리보다 **빠르지 않음**
    - 병렬 처리로 성능 개선시, 여러 독립적인 서브 태스크 분할이 가능해야 함
    - 성능 비교시 고려할 점
      - JIT 컴파일러에 의해 **최적화**되려면, 몇 차례의 **준비 과정** 및 **실행 과정**을 거쳐야 함
    - 성능 측정시, 지금까지 살펴본 하니스에서 그랬던 것처럼,
      - 여러 번 프로그램을 실행한 결과 측정이 필요
    - 컴파일러 최적화는 **병렬 버전** 보다는 **순차 버전**에 집중될 수 있음
      - 순차 버전에서는 죽은 코드를 분석해서, 사용되지 않는 계산은 아예 삭제하는 등의 최적화를 달성하기 쉬움
- 포크 조인 분할 전략에서는
  - 주어진 서브태스크를 **더 분할할 것인지 결정할 기준**을 정해야 함
- 다음 절에는 **분할 기준**에 대한 몇가지 힌트 제공

### 7.2.3. 작업 훔치기
- `ForkJoinSumCalculator` 예제에서는
  - 덧셈을 수행할 숫자가 **만 개 이하면** 서브태스크 분할을 종료했음
- 기준 값을 바꿔가면서, 실험해보는 방법 외에는
  - 좋은 기준을 찾을 방법이 없음
- **코어 개수**와 상관없이, **적절한 크기로 분할된 많은 태스크**를 포킹하는 것이 바람직
  - 이론적으로는 **코어 개수**만큼 병렬화된 태스크로 작업 부하를 분할 하면
    - 모든 CPU 코어에서 태스크를 실행, 같은 시간에 종료?
  - 하지만 이보다 복잡한 시나리오에서는 작업 완료 시간이 크게 달라질 수 있음
    - `분할 기법이 효율적이지 않거나`, `디스크 접근 속도가 저하되었거나`, `외부 서비스와 협력하는 과정에서 지연` 등
- `fork/join framework`에서는 **작업 훔치기**(work stealing) 기법으로 문제 해결
  - 이 기법에서는 `ForkJoinPool`의 모든 스레드를 **거의 공정하게 분할**
  - 각 스레드는 **자신**에게 할당된 태스크를 포함하는 **이중 연결 리스트**를 참조하면서
    - 작업이 끝날 때마다, **큐의 헤드**에서 다른 태스크를 가져와서 처리
  - 할 일을 마친 스레드는, 다른 스레드의 **queue tail**에서 작업을 훔쳐옴
  - 모든 태스크가 작업을 끝낼때, 모든 큐가 **비어 있을 때**까지 이 과정을 반복
  - 태스크의 크기를 작게 나누어야
    - 작업자 스레드 간의 작업 부하를 비슷한 수준까지 유지 가능
- 풀에 있는 **작업자 스레드**의 태스크를 **재분배**하고 **균형**을 맞출 때
  - 작업 훔치기 알고리즘을 사용함
- 작업자의 **큐**에 있는 태스크를
  - 두개의 서브 태스크로 분할 했을 때, 둘 중 하나의 태스크를 다른 **유휴 작업자**가 훔쳐갈 수 있음
- 주어진 태스크를 **순차 실행**할 단계가 될 때까지 이 과정을 **재귀적**으로 반복