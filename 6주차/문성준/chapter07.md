# 7. 병렬 데이터 처리와 성능

<aside>
📖 - 병렬 스트림으로 데이터 병렬 처리하기
- 병렬 스트림의 성능 분석
- 포크/조인 프레임워크
- Spliterator로 스트림 데이터 쪼개기

</aside>

자바 7 이전에는 컬렉션을 병렬로 처리하기 어려웠다.
데이터를 서브파트로 분할해야하고, 분할된 서브파트를 각각의 스레드로 할당해야하며, 스레드로 랄당한 다음에는 의도치않은 레이스컨디션(경쟁상태)이 발생하지 않도록 적절한 동기화를 추가해야하고 마지막에 부분 결과를 합쳐야한다. → 어우 상당히 번거롭고 귀찮습니다.

이를 자바 7은 더 쉽게 병렬화를 수행하면서 에러를 최소화할 수 있도록 포크/조인 프레임워크 기능을 제공합니다.

이 chapter의 핵심은 내부적으로 어떻게 처리되는지 알아야 하는 것입니다. 집중해주세요 !

# 병렬 스트림

병렬 스트림이란 ?

- 각각의 스레드에서 처리할 수 도록 스트림 요소를 여러번 청크로 분할한 스트림입니다.

스트림에 parallel 메서드를 호출하면 기존의 함수형 리듀싱 연산이 병렬로 처리된다.

```java
public long sequentialSum(long n) {
  return Stream.iterate(1L, i -> i + 1)
    .limit(n)
    .parallel() //병렬 스트림으로 변환
    .reduce(0L, Long::sum);
  }
```

스트림이 여러 청크로 분할되어 각각 리듀싱 연산을 수행한 후 다시 리듀싱 연산으로 합쳐져 결과를 도출한다.

> `parallel` 을 호출하면 스트림 자체에 변화가 생기는 것이 아니라 이후 연산이 병렬로 수행해야 함을 의미하는 불리언 프래그가 설정되는 것이다.
> 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/8c954fac-e473-49e2-9bab-1bcfbd303445/Untitled.png)

<aside>
❓ 청크를 왜 2개로 나눴을까??
병렬 리듀싱에서는 주로 요소의 인덱스를 기준으로 청크를 나눕니다.
만약 2개의 스레드로 나눈다고 가정하면 위와 같은 로직인데, 왜 2개로 나누는 것일까?
→ 두 개의 청크로 나뉘어지면, 각각의 스레드에서는 나뉜 청크를 병렬로 처리하고, 최종 결과를 얻기 위해 각각의 부분 결과를 합칩니다. 청크의 크기와 나누는 방식은 구현에 따라 다를 수 있습니다.
?????? 그래서 왜 2개로 한 것일까.?

</aside>

---

**병렬 스트림의 주의점**

리듀싱 과정을 시작하는 시점에 전체 숫자 리스트가 준비되지 않았다면, 스트림을 병렬로 처리할 수 있도록 청크로 분할할 수 없다.
→ 이 문장은 스트림을 병렬로 처리하는데 있어서 전체 데이터가 준비되어 있지 않으면 병렬 리듀싱을 시작할 수 없다는 것을 나타냅니다. 병렬 리듀싱은 데이터를 여러 스레드로 분할하여 동시에 처리하기 위한 것이기 때문에, 처리할 데이터가 미리 나눠져 있어야 합니다.

스트림을 병렬로 처리할 때, 병렬 리듀싱은 전체 데이터를 작은 청크로 나누어 각각의 스레드에게 할당하고, 각 스레드는 자신에게 할당된 청크를 독립적으로 처리합니다. 이렇게 병렬로 처리되는 과정에서 전체 데이터가 빠르게 처리될 수 있습니다.

그러나 만약 전체 데이터가 준비되어 있지 않아서 청크로 나눌 수 없는 상황이라면, 병렬 리듀싱을 시작할 수 없습니다. 데이터를 나눌 수 없다면 각각의 스레드에게 할당할 작업이 없기 때문입니다. 전체 데이터를 나누고 청크로 분할하기 위해서는 데이터가 적어도 청크로 나눌 수 있는 크기로 나뉘어져 있어야 합니다.

또한, 병렬 프로그래밍을 오용하면 오히려 전체 프로그램의 성능이 나빠질 수도 있다.
따라서 마법 같은 parallel 메서드를 호출했을 때 내부적으로 어떤 일이 일어나는지 꼮!꼮! 이해해야한다.

---

**병렬 스트림의 주의점**

```java
public long sideEffectSum(long n) {
  Accumlator accumulator = new Accumulator();
  LongStream.rangeClosed(1, n).forEach(accumulator::add);
  return acculator.total;
}

public class Accumulator {
  public long total = 0;
  public void add(long value) { total += value; }
}
```

여러 스레드에서 공유하는 객체의 상태를 바꾸는 forEach 블록 내부에서 add 메서드를 호출하면 올바른 결과값이 나오지 않는다. 병렬 스트림과 병렬 계산에서는 공유된 가변 상태를 피해야하기 때문이다.

---

**병렬 스트림 효과적으로 사용하기\**

- 확신이 서지 않을때는 순차 스트림과 병렬 스트림 구현 시의 성능을 직접 측정해라
- 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소이므로 주의해서 사용해야 하며, 기본형 특화 스트림(IntStream, LongStream, DoubleStream)을 사용하는 것이 좋다.
- limit이나 findFirst처럼 요소의 순서에 의존하는 연산은 병렬 스트림에서 성능이 더 떨어진다. 요소의 순서가 상관없다면 unordered를 호출해서 비정렬된 스트림을 얻은 후 limit을 호출하는 것이 더 효율적이다.
- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하자. 요소 수가 많고 요소 당 연산 비용이 높다면 병렬 스트림으로 성능을 개선할 여지가 있다.
- 병렬화 과정의 부가 비용을 상쇄하지 못할 정도의 소량의 데이터에서는 병렬스트림이 도움되지 않는다.
- 스트림을 구성하는 자료구조가 적절한지 확인한다. ArrayList는 요소를 탐색하지 않고도 분할할 수 있지만 LinkedList는 모든 요소를 탐색해야 분할할 수 있다.
    
    > LinkedList는 모든 요소를 탐색해야 분할이 가능하다.
    > 
- range 팩토리 메서드로 만든 기본형 스트림이나 커스텀 Spliterator를 구현하면 쉽게 분해할 수 있다.
    - 스트림의 특성과 파이프라인 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다. SIZED 스트림은 정확히 같은 크기의 두 스트림으로 분할할 수 있으므로 효과적으로 병렬 처리가 가능하다. 반면 필터 연산이 있으면 스트림의 길이를 예측할 수 없으므로 병렬 처리가 어려워진다.
- 최종 연산의 병합 과정 비용이 비싸다면 병렬 스트림으로 얻은 이익이 상쇄될 수 있다.

아래 표는 스트림 소스와 분해성을 나타낸 것이다. 분해성이 훌륭할 수록 병렬화에 적합한 자료구조이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/e9199e55-2e1f-4060-bde0-f845c4981f5b/Untitled.png)

---

# 7.2 포크/조인 프레임워크

병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할하여 서브태스크로 처리한 뒤, 각각의 결과를 합쳐서 전체 결과로 만드는 방식이며, 서브 테스크를 스레드 풀(ForkJoinPool)의 작업자 스레드에 분산할당 하는 ExecutorService 인터페이스를 구현한다.

스레드풀을 이용하려면 `RecursiveTask<R>`의 서브클래스를 만들어야하고, 여기서 R은 병렬화된 태스크가 생성하는 결과 형식 또는 결과가 없을 때(결과가 없더라도 다른 비지역 구조를 바꿀 수 있다)는 RecursiveAction 형식이다. 즉, RecusiveTask를 정의하려면 추상메서드 compute를 구현해야 한다.

```java
protected abstract R compute();
```

compute 메서드 구현 형식은 분할 정복 알고리즘의 병렬화 버전을 사용한다.

```
if(태스크가 충분히 작거나 더이상 분할할 수 없으면) {
  순차적으로 태스크 계산
} else {
  태스크를두 서브태스크로 분할
  태스크가 다시 서브태스크로 분할되도록 메시지를 재귀적으로 호출
  모든 서브태스크의 연산이 왑료될때까지 대기
  각 서브태스크의 결과를 합침
}
```

---

### 포크/조인 과정

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/5ef254f6-99f6-48d1-9f25-c4e4008e1426/Untitled.png)

```java
public class ForkJoinSumCalculator extends java.util.concurrent.RecursiveTask<Long>{
	private final long [] numbers;
	private final int start;
	private final int end;
	private final long THRESHOLD = 10_000;
	
	//main task 생성시 public 생성자
	public ForkJoinSumCalculator(long[] numbers){
		this(numbers, 0, numbers.length);
	}
	
	//recursive subtask 생성시 private 생성자
	private ForkJoinSumCalculator(long[] numbers, int start, int end){
		this.numbers = numbers;
		this.start = start;
		this.end = end;
	}
	
	@Override
	protected Long compute(){
		int length = end - start; // task에서 더할 배열의 길이
		if (length <= THRESHOLD){
			return computeSequentially(); // 기준값보다 작으면 순차적으로 결과를 계산
		}
		ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2);
		leftTask.fork(); // ForkJoinPool의 다른 스레드로 새로 생성한 태스크를 비동기로 실행
		ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
		
		Long rightResult = rightTask.compute(); // 두 번째 서브태스크를 동기 실행, 추가 분할 일어날 수 있음.
		Long leftResult = leftTask.join(); // 첫 번째 서브태스크의 결과를 읽거나 아직 없으면 기다린다.
		
		return leftResult + rightResult;
	}
	
	//분할 더 이상 안될 때 서브태스크 결과 계산해주는 알고리즘.
	private Long computeSequentially(){
		long sum = 0;
		for (int i = start; i < end; i++) {
			sum += numbers[i];
		}
		return sum;
	}
}
```

다음 코드처럼 ForkJoinSumCalculator의 생성자로 원하는 수의 배열을 넘겨줄 수도 있다.

```java
public static long forkJoinSum(long n){
		long[] numbers = LongStream.rangeClosed(1, n).toArray();
		ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
		return new ForkJoinPool().invoke(task);
	}
```

> 위의 코드에서 ForkJoinPool 은 소프트웨어의 필요한 곳에서 언제든 가져다 쓸 수 있도록 ForkJoinPool 을 한 번만 인스턴스화해서 정적 필드에 싱글턴으로 저장한다. 그래서 인수가 없는 디폴트 생성자를 이용해서 JVM 에서 이용할 수 있는 모든 프로세서가 자유롭게 풀에 접근할 수 있음을 의도했다.
> 

위 코드를 실행해보면 병렬 스트림을 이용할 때 보다 성능이 더 나빠졌다. 그 이유는 ForkJoinSumCalculator 태스크에서 사용할 수 있도록 전체 스트림을 long[]으로 변환했기 때문이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/1089ff3e-e2ff-4c3e-80a4-dc220090fb04/Untitled.png)

---

Q. 즉 , 포크 조인 알고리즘은 이진트리 알고리즘이라 유사한 것이네 ?
A.

포크-조인 알고리즘은 일반적으로 이진 트리를 사용하여 병렬 작업을 분할하고 조합하는 방식을 취합니다. 각 노드는 하나의 작업을 나타내며, 이 작업은 일정 크기 미만의 문제 크기에 도달하면 순차적인 방식으로 해결됩니다.

이진 트리의 각 노드에서는 작업을 두 개의 서브태스크로 나누어 포크(분할)하고, 이를 통해 병렬로 처리됩니다. 그리고 나중에 결과를 조합하기 위해 조인 단계에서 각 서브태스크의 결과를 합치거나 결합합니다.

포크-조인 알고리즘이 효과적인 이유 중 하나는 이러한 이진 트리 구조를 통해 태스크를 재귀적으로 나누고 조합하기 때문입니다. 이 구조는 병렬처리에 적합하며, 작업을 동시에 처리하고 합치는 방식으로 전체 작업을 효율적으로 처리할 수 있습니다.

따라서 포크-조인 알고리즘은 이진 트리 구조를 활용하여 병렬 처리를 수행하며, 이를 통해 대규모 작업을 효과적으로 처리할 수 있는 장점을 가지고 있습니다.

---

### 포크/ 조인 제대로 사용하기

- join 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록시킨다. 따라서 두 서브태스크가 모두 시작된 다음에 join을 호출하지 않으면, 각각의 서브태스크가 다른 서브태스크를 기다리는 일이 발생할 수 있다.
- RecursiveTask 내에서는 compute나 fork 메서드를 사용하며, 순차코드에서 병렬 계산을 시작할때만 ForkJoinPool의 invoke 메서드를 사용해야 한다.
- 서브태스크에 fork 메서드를 호출해서 ForkJoinPool의 일정을 조절할 수 있다. 한쪽 작업에만 fork를 호출하고 다른쪽에는 compute를 호출하면, 한 태스크에는 같은 스레드를 재사용할 수 있으므로 불필요한 태스크를 할당하는 오버헤드를 피할 수 있어 더욱 효율적이다.
- 포크/조인 프레임워크의 병렬 계산은 디버깅하기 어렵다. fork라 불리는 스레드에서 compute를 호출하므로 스택 트레이스가 도움이 되지 않는다. (보통 ide로 디버깅할 때 스택 트레이스로 문제가 일어난 과정을 쉽게 확인 가능하다)
- 병렬 처리로 성능을 개선하려면 태스크를 여러 독립적인 서브태스크로 분할할 수 있어야 하며, 각 서브태스크의 실행 시간은 새로운 태스크를 포킹하는 데 드는 시간보다 길어야한다.
또한 컴파일러 최적화는 병렬 버전보다는 순차 버전에 집중될 수 있다는 사실도 기억하자.

포크/조인 분할 전략에서는 주어진 서브테스크를 더 분할할 것인지 결정할 기준을 정해야한다. 이는 작업 훔치기를 통해서 일어난다

---

### 작업 훔치기

이론적으로는 CPU의 코어 개수만큼 병렬화된 태스크로 작업부하를 분할하면 모든 코어에서 태스크를 실행할 것이고, 같은 시간에 종료될 것이라고 생각할 수 있다. 하지만 다양한 이유로 각각의 서브태스크의 작업완료 시간이 크게 달라질 수 있다.

포크/조인 프레임워크에서는 작업훔치기(work stealing)라는 기법으로 이 문제를 해결한다. 작업 훔치기 기법에서는 ForkJoinPool의 모든 스레드를 거의 공정하게 분할한다. 각각의 스레드는 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하면서 작업이 끝날때마다 큐의 헤드에서 다른 태스크를 가져와 작업을 처리한다.

즉, 다른 스레드는 바쁘게 일하고 있는데 한 스레드는 할일이 다 떨어진 상황이다.
이때 할일이 없어진 스레드는 유휴 상태로 바뀌는 것이 아니라 다른 스레드 큐의 꼬리에서 작업을 훔쳐온다.
모든 태스크가 작업을 끝낼 때까지, 즉 모든 큐가 빌 때까지 이 과정을 반복한다.
따라서 태스크의 크기를 작게 나누어야 작업자 스레드 간의 작업부하를 비슷한 수준으로 유지할 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/b9169ddc-1517-4c87-b885-fdd718c9ed0d/Untitled.png)

이제, 위처럼 분할 로직을 개발하지 않고 병렬 스트림을 이용해 자동으로 스트림을 분할해주는 과정을 살펴보자.

### **Spliterator 인터페이스**

분할할 수 있는 반복자라는 의미이며, 자동으로 스트림을 부할하는 기법입니다.

```java
public interface Spliterator<T> {
  boolean tryAdvance(Consumer<? super T> action);
  Spliterator<T> trySplit();
  long estimateSize();
  int characteristics();
}
```

여기서 T는 Spliterator가 탐색하는 요소의 형식이다.

- `tryAdvance` 메서드는 Spliterator의 요소를 하나씩 순차적으로 소비하면서 탐색해야 할 요소가 남아있으면 참을 반환한다.
- 반면 `trySplit`메서드는 Spliterator의 일부 요소(자신이 반환한 요소)를 분할해서 두 번째 Spliterator를 생성하는 메서드다.
- Spliterator에서는 `estimateSize` 메서드로 탐색해야 할 요소 수 정보를 제공할 수 있다.
특히 탐색해야 할 요소 수가 정확하진 않더라도 제공된 값을 이용해서 더 쉽고 공평하게 Spliterator를 분할할 수 있다.
- characteristics - Spliterator의 특성을 정의

### **분할 과정**

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/8c427850-c43a-4e3d-87f3-732455b0dedc/Untitled.png)

스트림을 여러 스트림으로 분할하는 과정은 재귀적으로 일어난다.

이 분할 과정은 characteristics 메서드로 정의하는 Spliterator의 특성에 영향을 받는다.

1단계에서 첫 번째 Spliterator에서 trySplit을 호출하면 두 번째 Spliterator가 생성되고, 2단계에서 두 번째 Spliterator에서 trySplit을 호출하면 네 개의 Spliterator가 생성된다. 이는 trySplit가 null이 될때까지 반복한다.

### **Spliterator 특성**

Spliterator는 characteristics라는 추상 메서드도 정의한다. characteristics 메서드는 Spliterator자체의 특성 집합을 포함하는 int를 반환한다. Spliterator를 이용하는 프로그램은 이들 특성을 참고해서 Spliterator를 더 잘 제어하고 최적화할 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/3d068e6c-cd91-4401-8257-589ed5033910/Untitled.png)

### WordCounter 병렬로 수행하기

위 연산을 병렬 스트림으로 처리하면 원하는 결과가 나오지 않는다. 원래 문자열을 임의의 위치에서 둘로 나누다보니 하나의 단어를 둘로 계산하는 상황이 발생할 수 있기 때문이다.

따라서 문자열을 임의의 위치에서 분할하지 않고 단어가 끝나는 위치에서만 분할하도록 trySplit() 메서드를 구현해주면 된다.
