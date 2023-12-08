# Chapter07 병렬 데이터 처리와 성능

## 자바의 병렬

자바 7 이전의 버전에서는 병렬의 처리가 어려웠다.
분할을 위한 스레드 할당 → 동기화 추가 → 결과 합치기의 과정을 통해 병렬 처리가 실행되는데, 각 병렬처리를 한 후에 스레드를  합칠 때 동기화를 적절히 이뤄줘야 교착 상태를 피할 수 있었다.
7버전 부터 지원하기 시작한 Fork/Join 프레임워크를 활용하면 해당 문제를 쉽고 효율적으로 해결할 수 있다. 이 Fork/Join 프레임워크를 활용한 스트림 API와 병렬 처리에 대해서 배워보자.

---

## 병렬 스트림

- 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림
- 멀티코어 프로세서가 각각의 청크를 처리하도록 한다
- `Collections.parallelStream()` or `Arrays.stream().parallel()`
    - Pipeline에있는 parallel 상태에 true가 저장된다.
    - 이후 파이프라인 처리할 때 처리할 때 parallel 체크를 통해 해결한다.
- 병렬 처리를 고려할 때는 성능 벤치마킹을 하는 것을 권장
- **성능 최적화**
    - 기본형(primitive) 타입의 경우 기본형 특화 스트림 권장 - 오토박싱의 이유
    - 일반적으로 순차적인 처리가 필요한 스트림의 경우 병렬 비용이 더욱 비싸 느릴 가능성이 높음
    - 순차 처리가 필요없는 경우 findAny같은 순서에 상관 없는 쇼트 서킷, unordered 같은 메서드 사용
    - 하나의 요소를 처리하는 데 드는 비용이 비싸다면 고려
    - 스트림 자료구조의 특성을 확인 ex) LinkedList vs `ArrayList 일반적으로 좋음`
    - 병합 과정의 비용을 생각해야 한다
        
        
        | 소스 | 분해성 |
        | --- | --- |
        | ArrayList | Excellent |
        | LinkedList | Bad |
        | IntStream.range | Excellent |
        | Stream.iterate | Bad |
        | HashSet | Good |
        | TreeSet | Good |
        | Stream.generate | iterate보단 낫다 |

### parallel처리 관련 이미지

![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/642b5d28-19ac-40a3-956a-34b0fed6389f)

![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/efc232f8-8ce8-430b-bc6e-9b95ed04e1b5)

![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/15bfa77f-373e-4215-a09d-feb8f1450e57)

## 포크/조인 프레임워크

- 병렬 작업할 때 사용
- 재귀적으로 큰 작업을 작은 작업으로 분할한 후 서브태스크의 결과를 합쳐서 전체 결과를 만듦
- 스레드 풀(ForkJoinPool)의 작업자 스레드에 분산 할당하는 ExecutorService
    - ExecutorService를 왜 언급하는가?
        
        RecursiveTask를 실제로 활용했을 때 해당 추상 클래스의 부모인 ForkJoinTask에서 ForkJoinPool을 활용해서 실제 활용하고있기 때문
        
        RecursiveTask → ForkJoinTask에서 사용하는 `ForkJoinPool` → `ExcutorService`
        
        ![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/f2bd073c-4231-49e5-9557-6d69be8d177b)

        ![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/4b766947-591f-4728-99bd-7ba104385716)

        
- **분할 정복(divide-and-conquer) 알고리즘**의 병렬화
- **compute() 메서드 오버라이딩** 해서 구현
- 병렬처리 시 주의점
    - `join 호출`은 두 서브태스크가 모두 시작된 다음에 해야한다.
    - ForkJoinPool의 invoke메서드는 병렬 계산 시작하는 시점에서 한번만 사용
    - 한쪽은 fork() 한쪽은 compute() ⇒ 같은 스레드 재사용 피하기 위함
    - 디버깅이 어렵다 ⇒ 스택이 아닌 **다른 스레드** 이기 때문
    - 병렬 처리가 무조건 빠르지 않다.
- 작업 훔치기(work stealing) 특성을 갖고있음
- ForkJoin을 활용한 RecursiveTask 예시

```java
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;
import java.util.stream.LongStream;

public class ForkJoinSumCalculator extends RecursiveTask<Long> {

	//THRESHOLD 이상의 값 까지만 분해
  public static final long THRESHOLD = 10_000;

  private final long[] numbers;
  private final int start;
  private final int end;

  public ForkJoinSumCalculator(long[] numbers) {
    this(numbers, 0, numbers.length);
  }

  private ForkJoinSumCalculator(long[] numbers, int start, int end) {
    this.numbers = numbers;
    this.start = start;
    this.end = end;
  }

  @Override
  protected Long compute() {
    int length = end - start; //해당 태스크에서 더할 배열의 길이
    if (length <= THRESHOLD) {
      return computeSequentially();
    }
		//각 태스크 분리 left, right
    ForkJoinSumCalculator leftTask =
		new ForkJoinSumCalculator(numbers, start, start + length / 2);
		//생성한 태스크 비동기 실행
    leftTask.fork();
    ForkJoinSumCalculator rightTask =
		new ForkJoinSumCalculator(numbers, start + length / 2, end);
		//두번째 태스크 동기 실행
    Long rightResult = rightTask.compute();
		//비동기 실행했던 left의 결과를 읽거나 처리완료 후 읽기까지 대기
    Long leftResult = leftTask.join();
    return leftResult + rightResult;
  }

	//가장 작은 단위일 때 작은 단위로 쪼갠 태스크의 결과를 계산
  private long computeSequentially() {
    long sum = 0;
    for (int i = start; i < end; i++) {
      sum += numbers[i];
    }
    return sum;
  }
	
	public static long forkJoinSum(long n) {
    long[] numbers = LongStream.rangeClosed(1, n).toArray();
    ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
    return FORK_JOIN_POOL.invoke(task);
  }
}

//호출 방법
ForkJoinSumCalculator.forkJoinSum(long n));
```

## Spliterator 인터페이스

- 구성 요소
    - **booleaen tryAdvance(Consumer<? super T> action)**
        - 요소를 하나씩 순차적으로 돌며 요소가 남아있는지 확인
    - **Spliterator<T> trySplit()**
        - 일부 요소를 분할해서 새로운 Spliterator 생성
    - **long estimateSize()**
        - 탐색해야 할 요소 수 정보 제공(RecursiveTask의 THRESHOLD 역할로 보임)
    - **int characteristics()**
        - 다양한 특성 집합을 포함(ORDERED, DISTINCT 그 외 다수)

## 7장을 마치며

- 내부 반복을 통해 명시적으로 스트림 병렬처리
- 병렬처리가 무조건 빠르지 않음 특성과 병렬처리 후의 결과 등을 잘 확인하여 고려할 것
- 병렬 처리에는 포크/조인 프레임워크를 활용한다
- Spliterator를 통해 병렬 처리를 직접 정의할 수 있다.
