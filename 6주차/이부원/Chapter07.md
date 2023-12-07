## 병렬 스트림
---
Stream interface를 사용하면 paralleStream을 호출하여 간단하게 병렬 스트림을 사용할 수 있다. 병렬 스트림을 활용하면 컬렉션의 전체 요소 처리 시간을 줄여주기 때문에 작업 처리 시간을 줄일 수 있다.

외부 반복 - for 문


```java
  /**
   * 순차 리듀싱
   */
  public long sequentialSum(long n) {
      return Stream.iterate(1L, i -> i + 1)
              .limit(n)
              .reduce(0L, Long::sum);
  }

  /**
   * 외부 반복
   */
  public static long iterativeSum(long n) {
    long result = 0;
    for (long i = 0; i <= n; i++) {
      result += i;
    }
    return result;
  }
```

병렬 스트림 - parallel()
```java
  public static long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1).limit(n).parallel()
            .reduce(Long::sum).get();
  }
```

![](https://velog.velcdn.com/images/bw1611/post/d02be03d-b6d7-45fa-8596-fc4477e18f8b/image.png)

위의 순차 리듀싱을 parallel을 이용하여 병렬처리하였다고 과연 효율이 좋아질까? 대답은 "NO"다. 위 코드에서 병렬화 스트림으로 변환해서 생기는 이점은 없을 것이다. 이유는 순차 스트림에서 parallel을 호출해봐야 스트림 자체에는 변화가 없다. 내부적으로만 이후 연산이 병렬로 수행되야 함을 의미하는 플래그가 설정되는 것이다. 그래서 반대로 병렬 스트림을 순차 스트림을 바꾸는 sequential() 메서드도 존재한다.

> - 병렬 스트림의 ForkJoinPool
병렬 스트림은 내부적으로 ForkJoinPool을 사용하여 작업을 병렬로 처리한다. ForkJoinPool은 포크-조인 프레임워크를 제공하여 작업을 작은 작업 단위로 분할하고 병렬 처리하는데 사용된다. Parallel() 메서드는 단순히 스트림 파이프라인에 병렬 수행 힌트를 제공하며, 실제로 수행되는 것은 스트림 파이프라인의 나머지 부분에서 일어난다. 내부적으로 ForkJoinPool에 의해 관리되며, 작은 작업으로 분할되어 병렬로 처리된다.

- JMH로 성능 테스트 결과

for문을 이용한 반복문

```java
  @Benchmark
  public long iterativeSum() {
    long result = 0;
    for (long i = 1L; i <= N; i++) {
      result += i;
    }
    return result;
  }
```

![](https://velog.velcdn.com/images/bw1611/post/6bd90aac-c454-4575-a42e-6c91ba6ace59/image.png)



병렬 스트림을 이용한 반복문
```java
  @Benchmark
  public long parallelSum() {
    return Stream.iterate(1L, i -> i + 1).limit(N).parallel().reduce(0L, Long::sum);
  }
```

![](https://velog.velcdn.com/images/bw1611/post/86056f9e-281a-4a8d-878b-93ea4e3a6016/image.png)

예상했던 대로 병렬 스트림이 횔씬 느린것을 확인할 수 있다.
더 느린 결과가 나온데는 2가지 이유가 있다.

- 반복 결과로 박싱된 객체가 만들어지므로 숫자를 더하려면 언박싱을 해야함
- 반복 작업은 병렬로 수행할 수 있는 독립 단위로 나누기 어려움

여기서 두번째 경우는 쉽게 넘길 수 없는 부분이다. 리듀싱 과정을 시작하는 시점에선 전체 숫자 리스트가 준비되지 않았기에 병렬로 처리할 수 있도록 청크를 분할할 수 없다. 스트림을 병렬로 처리되도록 플래그까지 지정했지만, 실질적으로는 순차처리 방식과 크게 다른적이 없이 스레드 할당에 비용만 추가되어 오버헤드만 증가하게 된것이다.
이것을 해결하기 위해서는 병렬로 수행될 수 있는 스트림 모델이 필요하다. (iterate 연산은 청크로 분할하기 어렵다.)


### 더 특화된 메서드 사용하기

iterate 대신하여 LongStream.rangeClosed 메서드를 사용하면 좋다.

- 기본형 long을 직접 사용하므로 박싱과 언박싱 오버헤드가 사라진다.
- rangeClosed()는 쉽게 청크로 분할할 수 있는 숫자 범위를 생성한다.

```java
  @Benchmark
  public long parallelRangedSum() {
    return LongStream.rangeClosed(1, N).parallel().reduce(0L, Long::sum);
  }
```

![](https://velog.velcdn.com/images/bw1611/post/b38f3b43-e752-400d-8cb0-264511751fc6/image.png)



확실히 성능이 좋아진 것을 확인할 수 있다. 실행 시간에서 볼 수 있듯이 언박싱이 생기지 않으며 청크 단위가 확실히 분할되는 것으로 보인다. 올바른 자료구조를 선택해야지 병렬 실행도 최적의 성능을 발휘할 수 있는 것을 확인할 수 있었다.

### 병렬 스트림 올바른 사용법

```java
  public static long sideEffectSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add);
    return accumulator.total;
  }

  public static long sideEffectParallelSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
    return accumulator.total;
  }
```

![](https://velog.velcdn.com/images/bw1611/post/2a5323f4-ac89-45f9-80fc-fd8c24ba94bd/image.png)


위 코드를 순차적으로 처리했을 때는 문제가 생기지 않지만 병렬로 실행하였을 경우 효율도 좋지 않지만 우선 제대로 된 값이 나오지 않는다. 이유가 무엇일까? 여러 스레드에서 동시에 누적자 total += value를 실행하면서 문제가 발생하는 것이다. 결국 여러 스레드에서 공유하는 객체의 상태를 바꾸는 foreach 블록 내부에서 add 메서드를 호출하면서 문제가 발생하는 것이다. (병렬 계산에서는 공유된 가변 상태를 피해야 한다.)

> 아토믹 연산
여러 스레드들이 있을 때 하나의 공유 데이터에 대한 연산에서 한 스레드가 write 작업을 하고 있을 때, 다른 스레드가 read하는 것을 막기 위해서 하드웨어에서 지원하는 방법

병렬 스트림 효과적으로 사용하기위해서는 아래와 같은 사항들을 지켜야한다.

- 박싱을 주의해야 한다.
- limit 또는 findFirst 처럼 요소의 순서에 의존하는 연산은 병렬 스트림에서 사용하기에 비싼 비용을 사용해야하기 때문에 순차 스트림보다 효율이 떨어진다.
- 스트림에서 수행하는 전체 파이프라인 연산의 비용을 고려해야 한다.
- 소량의 데이터에서는 병렬 스트림이 그닥 도움이 되지 못한다.
- 확신이 서지 않다면 효율을 직접 체크해 보는것이 좋다.
- 스트림을 구성하는 자료구조가 적절한지 확인해야 한다.
- 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정에서 성능이 달라진다.
- 최종 연산의 병합 과정 비용을 살펴야한다.
- 병렬 스트림이 수행되는 내부 인프라구조도 살펴봐야 한다.

## 포크/조인 프레임워크
---
병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었다. 하나의 작업을 작은 단위로 나눠서 여러 쓰레드가 동시에 처리하는 것을 쉽게 만들어 준다. (분할 정복 알고리즘의 병렬화 버전)

포크/조인 프레임워크에서는 서브태스크를 스레드 풀(ForkJoinPool)의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다.

> ExecutorService
java.util.concurrent.Executors와 java.util.concurrent.ExecutorService를 이용하면 간단히 쓰레드풀을 생성하여 병렬처리를 할 수 있습니다.
ExecutorService에 Task만 지정해주면 알아서 ThreadPool을 이용해 Task를 실행하고 관리합니다.

### RecursiveTask 활용

스레드 풀을 이용하기 위해 `RecursiveTask<R>`의 서브클래스를 만든다.

![](https://velog.velcdn.com/images/bw1611/post/6d1519cb-e407-4b94-b053-1d88297321cc/image.png)

```java
  @Override
  protected Long compute() {
    int length = end - start; // task에서 더할 배열의 길이

    if (length <= THRESHOLD) {
      // 기준값과 같거나 작으면 순차적으로 결과 계산
      return computeSequentially();
    }
    // 분할정복? 배열의 첫 번째 절반을 더하도록 서브태스크 생성
    ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length / 2);
    // ForkJoinPool의 다른 스레드로 새로 생성한 태스크를 비동기로 실행
    leftTask.fork();
    // 배열의 나머지 절반 더하는 서브태스크 생성
    ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length / 2, end);Estimated Size: 10
    // 두번째 서브태스크 돌기 실행, 추가 분할 발생할 수 있다.
    Long rightResult = rightTask.compute();
    // 첫번째 서브테스크 결과를 읽거나 아직 결과가 없으면 기다림
    Long leftResult = leftTask.join();
    // 두 서브테스크의 결과를 조합한 값이 이 테스크의 결과값
    return leftResult + rightResult;
  }
```

위는 compute 추상 메서드를 오버라이딩한 부분이다. 태스크를 서브태스크로 분할하는 로직과 더 이상 분할할 수 없을 때 개별 서브태스크의 결과를 생산할 알고리즘을 정의한다. 그림으로 보면 이해하기 더 쉽다.

```
    long answer = forkJoinSum(10);
    System.out.println(answer); // 결과 : 55
```

실행 결과 제대로 된 값이 나온 것을 확인할 수 있다.

일반적으로 애플리케이션에서는 둘 이상의 ForkJoinPool을 사용하지 않는다. 이유는 소프트웨어의 필요한 곳에서 언제든 가져다 쓸 수 있도록 ForkJoinPool을 한 번만 인스턴스화해서 정적 필드에 싱글톤으로 저장한다.
ForkJoinPool의 인스턴스를 생성하면서 인수가 없는 기본 생성자를 이용했는데, 이는 JVM에서 이용할 수 있는 모든 프로세서가 자유롭게 폴에 접근할 수 있음을 의미한다. 정확하게는 Runtime.availableProcessors의 반환값으로 폴에 사용할 스레드 수를 결정한다.


![](https://velog.velcdn.com/images/bw1611/post/c2b23ce9-f8bb-408d-bbcd-4619f35264b4/image.png)

### 포크/조인 프레임워크를 제대로 사용하는 방법

- 두 서브태스크가 모두 시작된 다음에 join을 호출해 사용해야 한다.

모두 시작되지 않고 실행하게 되면 각각의 서브태스크가 다른 태스크가 끝나길 기다리는 일이 발생하며 원래 순차 알고리즘보다 느리고 복잡한 프로그램이 될 수 있다.

- RecursiveTask 안에서는 ForkJoinPool의 invoke 메서드를 사용하지 말아야 한다

invoke 대신 compute나 fork 메서드를 직접 호출할 수 있으며, 순차 코드에서 병렬 계산을 시작할 때만 invoke를 사용

- fork 메서드를 호출해 ForkJoinPool의 일정을 조절할 수 있다.

양쪽 작업에서 fork 메서드를 사용하는 것보다 compute를 호출하는 것이 효율적이다. 두 서브 태스크의 한 태스크에만 같은 스레드를 재사용할 수 있으므로 pool에서 불필요한 태스크를 할당하는 오버헤드를 피할 수 있다.

- 포크/조인 프레임워크의 병렬 계산은 디버깅이 어렵다.

fork라 불리는 다른 스레드에서 compute를 호출하므로 스택 트레이스가 도움이 되지 않는다.

> 스택 트레이스란?
프로그램의 실행 과정에서 호출된 메서드들의 순서와 위치를 나타내는 것이다. 예외가 발생한 지점부터 호출 스택의 상위 메서드들까지의 정보를 담고 있으므로 예외가 발생했을 떄 발생한 원일을 추적하고 디버깅하는데 매우 유용하다.

- 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠르지 않다

각 서브태스크의 실행시간은 새로운 태스크를 포깅하는 데 드는 시간보다 길어야 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 빠르다. 그러므로 알맞은 상황에 맞춰 사용하는 것이 좋다.

### 작업 훔치기
포크/조인 프레임워크에서는 작업 훔치기라는 알고리즘 기법을 사용한다. 이 기법을 통해 ForkJoinPool의 모든 스레드를 거의 공정하게 분배한다.
각각의 스레드는 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하면서 작업이 끝날 때마다 큐의 헤드에서 다른 태스크를 가져와서 작업을 처리한다.

만약 한 스레드가 다른 스레드보다 할당된 태스크를 빨리 처리했다면, 유휴 상태로 바뀌는 것이 아니라 다른 스래드 큐의 꼬리에서 작업을 훔쳐오는 것이다. 모든 큐가 끝날 때 까지 이런 작업을 반복하기 때문에 부하를 비슷하게 유지할 수 있다.

![](https://velog.velcdn.com/images/bw1611/post/cc26d33d-daa2-46f1-963a-51ee6fb0baed/image.png)


## Spliterator 인터페이스
---
자바 8에서 Spliterator라는 인터페이스를 제공한다. Iterator처럼 소스의 요소 탐색 기능을 제공하는 것은 같지만 병렬 작업에 특화되어 있다. 

```java
public interface Spliterator<T> {
    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();
    long estimateSize();
    int characteristics();
    ...
}
```
**tryAdvance()** : 요소를 순차적으로 소비하면서 탐색해야 할 요소가 남아있으면 true를 반환한다. 
**trySplit()** : 일부 요소를 분할해서 두 번째 Spliterator를 생성하는 메서드이다.
**estimateSize()** : 탐색해야 할 요소 수 정보를 제공한다. (탐색해야 할 요소 수가 정확하지 않더라도 제공된 값을 이용해 더 쉽고 공평하게 분할 가능)
**characteristics()** : Spliterator 자체의 특성 집합을 포함하는 int를 반환한다. 메서드의 속성으로는 ORDERED, DISTINCT, SORTED, SIZED, NONNULL, IMMUTABLE, CONCURRENT, SUBSIZED 가 있다. 각 특성은 어떤 Spliterator 객체인가에 따라 다르며 그에 따른 각 메서드들의 내부적인 동작이 다를 수 있습니다.


- Spliterator 분할 과정
![](https://velog.velcdn.com/images/bw1611/post/1c134479-dc18-4eaa-85f7-75b7b6f77ead/image.png)

- tryAdvance() 메서드 활용
```java

        // 간단한 정수 리스트 생성
        List<Integer> numbers = new ArrayList<>();
        Set<Integer> setNumbers = new HashSet<>();
        for (int i = 1; i <= 10; i++) {
            numbers.add(i);
            setNumbers.add(i);
        }
        
        Spliterator<Integer> spliterator = numbers.spliterator();
        boolean hasNext;
        do {
            hasNext = spliterator.tryAdvance(System.out::println);
            System.out.println(hasNext);
        } while (hasNext);
```

```결과
1
true
2
true
3
true
4
true
5
true
6
true
7
true
8
true
9
true
10
true
false
```

요소가 남아있으면 true를 없다면 false값이 출력되는 것을 확인해볼 수 있다.

- trySplit() 메서드 활용
```java
        // Spliterator 둘로 나누기
        Spliterator<Integer> trySpliterator = numbers.spliterator();
        Spliterator<Integer> secondHalf = trySpliterator.trySplit();

        // 첫 번째 Spliterator 요소 소비
        System.out.println("First Spliterator:");
        trySpliterator.forEachRemaining(System.out::println);

        // 두 번째 Spliterator 요소 소비
        System.out.println("Second Spliterator:");
        secondHalf.forEachRemaining(System.out::println);
```

```
First Spliterator:
6
7
8
9
10
Second Spliterator:
1
2
3
4
5
```

두가지의 요소로 나눠지는 것을 확인할 수 있다. ORDERED 특성이 있는 경우에는 새로 생성된 스플리터가 앞쪽 부분을 나타내고, 기존 스플리터는 뒷쪽 부분을 나타냅니다. 만약 ORDERED의 특성이 없다면 다른 결과를 확인할 수 있을 것이다.

- estimateSize() 메서드 활용
```java
        Spliterator<Integer> esiSpliterator = numbers.spliterator();

        // Spliterator의 예상 크기 출력
        System.out.println("Estimated Size: " + esiSpliterator.estimateSize());
```
```
Estimated Size: 10
```

탐색해야할 요소의 수를 출력하므로 size()랑 조금 비슷한 개념인 것 같다.


- characteristics() 메서드 활용

```java

        // 리스트의 Spliterator 생성
        int setCharacteristics = setNumbers.spliterator().characteristics();
        int characteristics = numbers.spliterator().characteristics();

        // Set의 경우 DISTINCT=1, SIZED=64 의 합인 65를 반환합니다.
        System.out.println("Spliterator Characteristics: " + characteristics);
        // List의 경우 ORDERED=16, SIZED=64, SUBSIZED=16384 의 합인 16464를 반환합니다.
        System.out.println("setCharacteristics = " + setCharacteristics);

```

서로 다른 특성이 Spliterator에 적용이되어 메서드의 내부 동작이 달라지는 것을 확인할 수 있었다.



**참고자료**

[Stream API 병렬 데이터 처리하기](https://catsbi.oopy.io/0428be55-8c8d-40a2-923a-acc738d74a14)
