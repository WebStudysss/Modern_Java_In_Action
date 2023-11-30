<aside>
📖 이 장의 내용
- Collectors 클래스로 컬렉션을 만들고 사용하기
- 하나의 값으로 데이터 스트림 리듀스하기
- 특별한 리듀싱 요약 연산
- 데이터 그룹화와 분할
- 자신만의 커스텀 컬렉터 개발

</aside>

> 컬렉션(`Collection`), 컬렉터(`Collector`), `collect` 는 다 다른 것이다.
> 

---

`Stream`에 `toList`를 사용하는 대신 더 범용적인 컬렉터 파라미터를 `collect` 메서드에 전달함으로써 원하는 연산을 간결하게 구현할 수 있음을 지금부터 배우게 될 것이다.

```java
// 맛보기
Map<Currency, List<Transaction>> transactionsByCurrencies =
	transacitons.stream()
							.collect(groupingBy(Transaction::getCurrency));
```

---

# 컬렉터란 ?

명령형 프로그래밍에 비해 함수형 프로그래밍이 훠~~~얼씬 더 편리하다.

함수형 프로그래밍에선 **무엇**을 원하는지 직접 명시할 수 있어서 어떤 방법으로 이를 얻을지는 신경 쓸 필요가 없다.
다수준 multilevel 로 그룹화를 수행할 때 명령형과 함수형 프로그래밍의 차이점이 더욱 두드러집니다.
명령형 코드는 문제를 해결하는 과정에서 다중 루프와 조건문을 추가하여 가독성 및 유지보수성이 크게 떨어진다.
but 함수형 프로그래밍은 필요한 컬렉터를 쉽게 추가하여 해결할 수 있습니다.

**고급 리듀싱 기능을 수행하는 컬렉터**

훌륭하게 설게된 함수형 api의 또 다른 장점으로 높은 수준의 조합성과 재사용성을 꼽을 수 있다.
collect로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다는 점이 컬렉터의 최대 강점이다.
collect에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리한다.

Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할지 결정한다.
Collectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 팩토리 메서드를 제공한다. 예를 들어 가장 많이 사용하는 직관적인 정적 메서드로 toList(스트림의 모든 요소를 리스트로 수집한다)가 있다.

**미리 정의된 컬렉터**

미리 정의된 컬렉터, 즉 groupingBy 같이 collectors 클래스에서 제공하는 팩토리 메서드의 기능을 설명한다.
Collectors에서 제공하는 메서드의 기능은 크게 세 가지로 구분할 수 있다.

- 스트림 요소를 하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

컬렉터 : 리듀싱과 요약 관련 기능을 수행한다.

---

### 리듀싱과 요약

컬렉터(Stream.collect 메서드의 인수)로 스트림의 항목을 컬렉션으로 재구성 할 수 있다.
`counting()` : 팩토리 메서드가 반환하는 컬렉터

**Optional의 역할**

자바8의 `Optional`은 값을 포함하거나 또는 포함하지 않을 수 있는 컨테이너 `optional`을 제공한다.
해당 내용은 11장에서 자세히 다룰 것이다.
또한 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 사용된다.
이러한 연산을 **요약** 연산이라고 부른다.

**요약 연산**

`Collectors` 클래스는 `Collectors.summingInt`라는 특별한 요약 팩토리 메서드를 제공한다.
`summingInt`는 객체를 int로 매핑하는 함수를 인수로 받는다. `summingInt`의 인수로 전달된 함수는 객체를 `int`로 매핑한 컬렉터를 반환한다.
그리고 `summingInt`가 `collect` 메서드로 전달되면 요약 작업을 수행한다.

```java
// 메뉴리스트의 총 칼로리 계산
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

`summing<데이터타입>` : 데이터 타입에 `long`, `double`도 당연히 가능하다. 
`collect(averagingInt())` : 평균

`Summarizing` : 두 개 이상의 연산을 한 번에 수행해야할 때 쓰이며 `Summarizing`가 반환하는 컬렉터를 사용할 수 있다. 요약 연산으로 모든 요소를 가져온다.

**문자열 연결**

컬렉터에 `joining` 팩토리 메서드를 이용하면 스트림의 각 객체에 `toString` 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결해서 반환한다.
`joining` 메서드는 내부적으로 `StringBuilder`를 이용해서 문자열을 하나로 만든다.
`joining`엔 구분 문자열을 넣을 수 있다. (예 :`collect(joining(”, “));`)

**항등 함수**

자신을 그대로 반환한다.

<aside>
👉 **collect와 reduce**
`collect` 메서드는 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드인 반면 
`reduce`는 두 값을 하나로 도출하는 불변형 연산이라는 점에서 의미론적인 문제가 일어난다.
`reduce` 메서드는 누적자로 사용된 리스트를 변환시키므로 잘못 사용한 것이다.

여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가져버리므로 리듀싱 연산을 병렬로 수행할 수 없다는 점도 문제다. 이 문제를 해결하려면 매번 새로운 리스트를 할당해야 하고 따라서 객체를 할당하느라 성능이 저하될 것이다.
즉, 가변 컨테이너 관련 작업이면서 병렬성을 확보하려면 collect 메서드로 리듀싱 연산을 구현해야한다.

</aside>

**컬렉션 프레임워크 유연성 : 같은 연산도 다양한 방식으로 수행 가능하다**

`reducing` 컬렉터를 사용한 이전 예제에서 람다 표현식 대신 `Integer` 클래스의 `sum` 메서드 참조를 이용하면 코드를 좀 더 단순화할 수 있다. 

```java
int totalCalories = menu.stream().collect(reducing(0, // 초깃값
													Dish::getCalories, // 합계함수
													Integer::sum)); // 변환함수
```

> 필자의 생각정리 : 스트림을 쓰면, 웬만한 로직은 데이터를 일단 스트림 한다 = 병렬로 쭉 데이터를 좌우로 세우고, 합계 등 연산 함수를 통해 데이터를 가공(?) 한다. 그 후, 변환함수를 사용한다. (이때 데이터 타입 결정되는거)
> 

```java
public static <T> Collector<T, ?, Long> counting(){
	return reducing(0L, e -> 1L, Long::sum);}
```

<aside>
👉 **제네릭 와일드 카드 ‘?‘ 사용법**
위 예제에서 counting 팩토리 메서드가 반환하는 컬렉터 시그니처의 두 번째 제네릭 형식으로 와일드카드 `?`이 사용되었다. 자바 컬렉션 프레임워크의 내용이며, `?`는 컬렉터의 누적자 형식이 알려지지 않았음을. 즉, 누적자의 형식이 자유로움을 의미한다.

</aside>

- `orElse`, `orElseGet` : `Optional`의 값을 얻어온다.

---

**무엇이 정답일까?**

지금까지 함수형 프로그래밍으로 하나의 연산을 다양한 방법으로 해결할 수 있음을 보여줬다.
또한 스트림 인터페이스에서 직접 제공하는 메서드를 이용하는 것에 비해 컬렉터를 이용하는 코드가 더 복잡하다는 사실도 보여준다. 컬렉터는 코드가 좀 더 복잡한 대신 재사용성과 커스터마이즈 가능성을 제공하는 높은 수준의 추상화와 일반화를 얻을 수 있다.

적절한 해결책을 고르는 것이 정답이지만 너무 추상적이다. 나는 정답은 없다고 생각한다.
(상황에 따라 적절한 방법을 선택하는 것이 중요하다고 많이 말하지만, 이는 곧 경험이다.)
그러나, 최적의 해는 선택할 수 있는 것이다. 예를 들어 현대 차에서 원하는 옵션으로 구성된 차의 합계를 계산한다면, IntStream을 사용하는 방법이 가독성이 좋고 간결할 것이다. 
또한, IntStream 덕분에 **자동 언박싱** 연산을 수행하거나 Integer을 int로 변환하는 과정을 피할 수 있어 성능까지 좋다.

> **정리**
함수형 프로그래밍은 다양한 해결책을 제공하며, 그 중에서도 스트림과 컬렉터는 각각의 특징과 장단점이 있다. 스트림 인터페이스는 직관적이고 간결한 반면, 컬렉터는 더 높은 수준의 추상화와 일반화를 제공하며, 재사용성과 커스터마이즈 가능성이 더 높다.
> 

---

## **그룹화**

데이터 집합을 하나 이상의 특성으로 분류해서 그룹화하는 연산도 db에서 많이 수행되는 작업입니다.
트랜잭션 통화 그룹화 예제에서 확인했듯이 명령형으로 그룹화를 구현하려면 까다롭고, 할일이 많으며, 에러도 많이 발생합니다.
BUT 자바 8의 함수형을 이용하면 가독성 있는 한 줄의 코드로 그룹화를 구현할 수 있습니다.
팩토리 메서드 `Collectors.groupingBy`를 이용해서 쉽게 그룹화할 수 있습니다.

```java
Map<Dish, Type, List<Dish>> dishesByType = 
	menu.stream()
			.collect(groupingBy(Dish::getType));
// 다음은 Map에 포함된 결과
{FISH = [pr, sa], OTHER =[ff, rice, sf, pizza], MEAT=[pork,beef,checkin]}
```

위 코드를 보면, 스트림의 각 요리에서 `Dish.Type`과 일치하는 모든 요리를 추출하는 함수를 `groupingBy`메서드로 전달했다. 이 함수를 기준으로 스트림이 그룹화되므로 이를 ***분류 함수*** 라고 한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/eb31764d-add1-42a8-8406-8e4723bff578/Untitled.jpeg)

위 로직은 그룹화 연산의 결과로 그룹화 함수가 반환하는 키 그리고 각 키에 대응하는 스트림의 모든 항목 리스트를 값으로 갖는 맵이 반환된다. 메뉴 그룹화 예제에서 키는 요리의 종류고, 값은 해당 종류에 포함되는 모든 요리다.

```java
// 그룹화를 하기 전에 프레디케이트로 필터를 적용
Map<Dish, Type, List<Dish>> caloricDishByType = 
	menu.stream()
			.filter(dish -> dish.getCalories() >500)
			.collect(groupingBy(Dish::getType));
/*
단점이 존재한다. 요리는 맵 형태로 되어있으므로 위 기능을 사용하려면 맵에 코드를 적용해야한다.
또한, 필터 프레디케이트를 만족하는 FISH 종류의 요리는 없으므로 결과 맵에서 해당 키 자체가 사라진다.
이는 중요하다. 키 자체가 사라졌으니까. 이를 해결하려면 Collector안으로 필터 프레디케이트를 이동하면 된다.
*/
Map<Dish, Type, List<Dish>> caloricDishByType = 
	menu.stream()
			.collect(groupingBy(Dish::getType,
							filtering(dish -> dish.getCalories() >500, toList())));
/*
이 프레디케이트로 각 그룹의 요소와 필터링 된 요소를 재그룹화 한다.
이를 통해 목록이 비어있는 FISH 항목도 추가된다.
*/
```

> **코드 정리**
> 
> 
> 필터링 조건을 만족하지 않는 요리 타입(FISH)는 결과 맵에서 완전히 제거된다.
> 필터링을 먼저 진행하므로, 그룹화 이후에 필터링을 적용하려면 추가적으로 맵에 코드를 적용해야 한다.
> 
> 두 번째 코드는 이러한 문제를 해결하기 위해 `filtering` 함수를 `groupingBy` 함수 내부에 적용한다.
> 이렇게 하면 각 그룹별로 필터링을 적용한 후에 그룹화를 진행하므로, 필터링 조건을 만족하지 않는 요리 타입도 결과 맵에 빈 리스트 형태로 포함된다. 이는 필터링 조건에 따라 요리 타입이 완전히 제거되는 것을 방지하며, 그룹화된 요리 리스트에 추가 조작을 적용하기 쉽게 만듭니다.
> ***즉, 그룹화 필터링 방법에 대해 문제점과 해결방법에 대해 말하고자 하는 코드였다.***
> 

**다수준 그룹화**
두 인수를 받는 팩토리 메서드 `Collectors.groupingBy`를 이용해서 항목을 다수준으로 그룹화 할 수 있다.

`groupingBy` 컬렉터는 스트림의 첫 번째 요소를 찾은 이후에야 그룹화 맵에 새로운 키를 게으르게 추가한다.

<aside>
👉 ***게으르게 추가한다*** 는 개념은 특정 키에 대한 그룹이 필요한 시점까지는 그룹화 맵에 해당 키를 추가하지 않고 대기한다는 의미입니다.

예를 들어, 스트림에서 요소를 그룹화할 때 **`groupingBy`**를 사용하면 해당 키에 대한 그룹을 만들게 됩니다. 그러나 이 그룹에 대한 요소가 나타나지 않은 상태에서는 해당 키가 실제로 맵에 추가되지 않습니다. 대신에 요소가 나타날 때까지 대기하다가, 첫 번째 요소가 나타나면 해당 키에 대한 그룹이 맵에 추가됩니다.

이렇게 하면 모든 키에 대한 그룹을 미리 만들어두지 않고, 실제로 필요한 경우에만 생성하여 메모리를 효율적으로 사용할 수 있습니다. 게으르게 추가함으로써 필요하지 않은 작업을 최소화하고, 필요한 시점에만 작업을 수행함으로써 성능을 향상시킬 수 있습니다

</aside>

---

### 분할

분할은 ***분할함수***라 불리는 프레디케이트를  분류 함수로 사용하는 특수한 그룹화 기능입니다.
분할 함수는 불리언을 반환하므로 맵의 키 형식은 Boolean이다.

```java
Map<Boolean, List<Dish>> partitionedMenu =
	menu.stream()
			.collect(partitiningBy(Dish::isVegetarian)); //분할 함수
// 참 값의 키로 맵에서 모든 채식요리 얻기
List<Dish> vegetarianDishes = partitionedMenu.get(true);
```

**분할의 장점**

분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점이다.

- `partioningBy` : 컬렉터를 두 번째 인수로 전달할 수 있는 오버로드된 버전의 메서드이며, `groupingBy` 컬렉터와 마찬가지로 `partitiongBy` 컬렉터도 다른 컬렉터와 조합해서 사용할 수 있으며, 
다수준 분할을 수행할 수 있다.

---

## **Collectors 클래스의 정적 팩토리 메서드**

### 1. toList()

- 팩토리 메서드: `Collectors.toList()`
- 반환형식: `Collector<T, ?, List<T>>`
- 사용 예제
    
    ```java
    javaCopy code
    List<String> stringList = Stream.of("apple", "banana", "orange")
                                   .collect(Collectors.toList());
    ```
    

### 2. toSet()

- 팩토리 메서드: `Collectors.toSet()`
- 반환형식: `Collector<T, ?, Set<T>>`
- 사용 예제
    
    ```java
    javaCopy code
    Set<Integer> integerSet = Stream.of(1, 2, 3, 3, 4, 5)
                                    .collect(Collectors.toSet());
    ```
    

### 3. toMap()

- 팩토리 메서드: `Collectors.toMap(keyMapper, valueMapper)`
- 반환형식: `Collector<T, ?, Map<K,U>>`
- 사용 예제
    
    ```java
    javaCopy code
    Map<String, Integer> lengthMap = Stream.of("apple", "banana", "orange")
                                          .collect(Collectors.toMap(Function.identity(), String::length));
    ```
    

### 4. joining()

- 팩토리 메서드: `Collectors.joining()`
- 반환형식: `Collector<CharSequence, ?, String>`
- 사용 예제
    
    ```java
    javaCopy code
    String result = Stream.of("Java", "is", "awesome")
                         .collect(Collectors.joining(" "));
    ```
    

### 5. counting()

- 팩토리 메서드: `Collectors.counting()`
- 반환형식: `Collector<T, ?, Long>`
- 사용 예제
    
    ```java
    javaCopy code
    long count = Stream.of("apple", "banana", "orange")
                      .collect(Collectors.counting());
    ```
    

### 6. summingInt()

- 팩토리 메서드: `Collectors.summingInt()`
- 반환형식: `Collector<T, ?, Integer>`
- 사용 예제
    
    ```java
    javaCopy code
    int sum = Stream.of(1, 2, 3, 4, 5)
                   .collect(Collectors.summingInt(Integer::intValue));
    ```
    

### 7. averagingInt()

- 팩토리 메서드: `Collectors.averagingInt()`
- 반환형식: `Collector<T, ?, Double>`
- 사용 예제
    
    ```java
    javaCopy code
    double average = Stream.of(1, 2, 3, 4, 5)
                          .collect(Collectors.averagingInt(Integer::intValue));
    ```
    

### 8. summarizingInt()

- 팩토리 메서드: `Collectors.summarizingInt()`
- 반환형식: `Collector<T, ?, IntSummaryStatistics>`
- 사용 예제
    
    ```java
    javaCopy code
    IntSummaryStatistics stats = Stream.of(1, 2, 3, 4, 5)
                                      .collect(Collectors.summarizingInt(Integer::intValue));
    ```
    

### 9. groupingBy()

- 팩토리 메서드: `Collectors.groupingBy(classifier)`
- 반환형식: `Collector<T, ?, Map<K, List<T>>>`
- 사용 예제
    
    ```java
    javaCopy code
    Map<Integer, List<String>> lengthGrouped = Stream.of("apple", "banana", "orange")
                                                   .collect(Collectors.groupingBy(String::length));
    ```
    

### 10. partitioningBy()

- 팩토리 메서드: `Collectors.partitioningBy(predicate)`
- 반환형식: `Collector<T, ?, Map<Boolean, List<T>>>`
- 사용 예제
    
    ```java
    javaCopy code
    Map<Boolean, List<String>> partitioned = Stream.of("apple", "banana", "orange")
                                                  .collect(Collectors.partitioningBy(s -> s.length() > 5));
    ```
    

### 11. collectingAndThen()

- 팩토리 메서드: `Collectors.collectingAndThen(downstream, finisher)`
- 반환형식: `Collector<T, A, R>`
- 사용 예제
    
    ```java
    javaCopy code
    List<String> result = Stream.of("apple", "banana", "orange")
                               .collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList));
    ```
    

### 12. groupingByConcurrent()

- 팩토리 메서드: `Collectors.groupingByConcurrent(classifier)`
- 반환형식: `Collector<T, ?, ConcurrentMap<K, List<T>>>`
- 사용 예제
    
    ```java
    javaCopy code
    ConcurrentMap<Integer, List<String>> lengthGrouped = Stream.of("apple", "banana", "orange")
                                                            .collect(Collectors.groupingByConcurrent(String::length));
    ```
    

### 13. countingAndThen()

- 팩토리 메서드: `Collectors.countingAndThen(downstream, finisher)`
- 반환형식: `Collector<T, ?, R>`
- 사용 예제
    
    ```java
    javaCopy code
    int count = Stream.of("apple", "banana", "orange")
                      .collect(Collectors.countingAndThen(Collectors.toList(), List::size));
    ```
    

### 14. summarizingDouble()

- 팩토리 메서드: `Collectors.summarizingDouble()`
- 반환형식: `Collector<T, ?, DoubleSummaryStatistics>`
- 사용 예제
    
    ```java
    javaCopy code
    DoubleSummaryStatistics stats = Stream.of(1.5, 2.5, 3.5, 4.5, 5.5)
                                          .collect
    (Collectors.summarizingDouble(Double::doubleValue));
    ```
    

### 15. reducing

- 팩토리 메서드 : `reducing`
- 반환형식 : The type produced by the reduction operation
- 사용 예제
    
    ```java
    누적자를 초깃값으로 설정한 다음에 BinaryOperator로 스트림의 각 요소를 반복적으로 누적자와 합쳐
    스트림을 하나의 값으로 리듀싱
    int totalCalories = 
    	menuStream.collect(reducing(0, Dish::getCalories, Integer::sum));
    ```
    

### 16. maxBy / minBy

- 팩토리 메서드 : maxBy / minBy
- 반환 형식 : `Optional<T>`
- 사용 예제
주어진 비교자를 이용해서 스트림의 최소/최댓 값 요소를 `Optional`로 감싼 값을 반환한다.
스트림에 요소가 없을 때는 `Optional.empty()` 반환.
    
    ```java
    Optional<Dish> fattest =
    	menuStrea.collect(maxBy(comparingInt(Dish::getCalories)));
    ```
    
    ```java
    Optional<Dish> lightest =
    	menuStrea.collect(maxBy(comparingInt(Dish::getCalories)));
    ```
    

---

# Collector 인터페이스

Collector 인터페이스는 리듀싱 연산(컬렉터)을  어떻게 구현할지 제공하는 메서드 집합으로 구성된다.

팩토리 메서드인 **`toList`**는 자주 사용하는 컬렉터 중 하나임으로 알아두자.

```java
public interface Collector<T, A, R>{
	Supplier<A> supplier();
	BiConsumer<A, T> accumulator();
	Function<A, R> finisher();
	BinaryOperator<A> combiner();
	Set<Characteristics> charateristics();
}
// Characteristics 외 나머지 4개 메서드는 collect 메서드에서 실행하는 함수를 반환한다.
// Characteristics 는 collect 메서드가 어떤 최적화(예를 들면 병렬화 같은)를 이용해서 리듀싱
// 연한을 수행할 것인지 결정하도록 돕는 힌트 특성 집합을 제공한다.
```

- `T` : 수집될 스트림 항목의 제네릭 형식
- `A` : 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식
- `R` : 수집 연산 결과 객체의 형식(항상 그런 거는 아니지만 거의 컬렉션 형식)

예를 들어 `Stream<T>`의 모든 요소를 `List<T>`로 수집하는 `ToListCollector<T>`라는 클래스를 구현

```java
public class ToListCollector<T> implements Collector<T, Lsit<T>, List<T>>
```

### supplier () : 새로운 결과 컨테이너 만들기

빈 결과로 이루어진 Sipplier를 반환해야 한다. 즉, supplier는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수다. ToListCollector처럼 누적자를 반환하는 컬렉터에서는 빈 누적자가 비어있는 스트림의 수집 과정의 결과가 될 수 있다. (아래 코드는 빈 리스트를 반환한다)

```java
public Supplier<List<T>> supplier(){
	return () -> new ArrayList<T>();
}
// 생성자 참조를 전달하는 방법
public Supplier<List<T>> supplier(){
	return ArrayList::new;
}
```

### accumulator() : 결과 컨테이너에 요소 추가

리듀싱 연산을 수행하는 함수를 반환한다.
스트림에서 n번째 요소를 탐색할 때 두 인수, 즉 누적자와 n번째 요소를 함수에 적용한다.
함수의 반환값은 void, 즉 요소를 탐색하면서 적용하는 함수에 의해 누적자 내부상태가 바뀌므로 누적자가 어떤 값일지 단정할 수 없다. `ToListCollector`에서 `accumulator`가 반환하는 함수는 이미 탐색한 항목을 포함하는 리스트에 현재 항목을 추가하는 연산을 수행한다.

```java
public BiConsumer<List<T>, T> accumulator(){
	return (list, item) -> list.add(item);
}
// 메서드 참조 이용으로 코드 간결화
public Biconsumer<List<T>, T> accumulator(){
	return List::add;
}
```

### finisher () : 최정 변환값을 결과 컨에티너로 적용

`finisher` 메서드는 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 반환하면서 누적 과정을 끝낼 때 호출할 함수를 반환해야 한다. 때로는 `ToListCollector`에서 볼 수 있는 것처럼 누적자 객체가 이미 최정 결과인 상황도 있다. 이런 때는 변환 과정이 필요하지 않으므로 `finisher` 메서드는 항등 함수를 반환한다.

```java
public Function<List<T>, List<T>> finisher(){
	return Function.identity();
}
```

위 세가지 `supplier`, `accumulator`, `finisher` 메서드로 순차적 스트림 리듀싱 기능을 수행할 수 있다.
실제로는 collect가 동작하기 전에 다른 중간 연산과 파이프라인을 구성할 수 있게 해주는 게으른 특성, 병렬실행 등도 고려해야 하므로 스트림 리듀싱 기능 구현은 생각보다 복잡하다.

<aside>
❓ **Q. y ? 그럼 왜 쓰는 걸까  ?**

**A.** 복잡하지만, 사용하면 코드의 간결화 및 가독성 향상을 위해 ?? → 어떻게 ? 
(로직의 분할)알고리즘을 쓰니까, 로직이 좀더 분업화 되어서 좋다 ? → 과연 이게 맞는 생각일까  ? 흠..

스트림 리듀싱을 하는 이유는 함수형 프로그래밍을 활용해 커스터마이징이 가능하기 때문에 ? 
이유

1.  **가독성 및 간결성 향상**
스트림 리듀싱을 사용하면 데이터 처리 과정을 간결하고 가독성 있게 표현할 수 있다.
함수형 프로그래밍의 특성을 활용하여 코드를 간소화하여 가독성  좋게 가능
2. **분업화된 로직**
각각의 리듀싱 단계가 순차적으로 나눠지기 때문에 각 단계의 역할을 명확히 할 수 있다.
이는 유지보수성을 상당히 높이며, 코드의 이해를 용이하게 한다.
3. **병렬처리 및 게으른 연산**
스트림은 병렬 처리를 지원하며, 게으른 특성을 가지고 있다. 이는 필요한 경우에만 요소를 처리하고 중간 연산을 게으르게 평가하여 성능을 최적화할 수 있도록 한다.

But 위와 같은 장점도 있는 반면 몇 가지 주의 사항도 있다.
예를 들어, 스트림이나 컬렉션의 크기가 작거나 단일 스레드에서 수행될 경우에는 일반적인 루프나 다른 방법을 사용하는 것이 더 효율적일 수 있다.

</aside>

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/65443ad4-de9f-4203-abed-d03493089600/Untitled.png)

### combiner () : 두 결과 컨테이너 병합

마지막으로 리듀싱 연산에서 사용할 함수를 반환하는 네 번째 메서드인 cominer를 알아보자.
combiner은 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의한다.
toList의 combiner는 비교적 쉽게 구현할 수 있다. 즉, 스트림의 두 번째 서브파트에서 수집한 항목 리스트를 첫 번째 서브파트 결과 리스트 뒤에 추가하면 된다는 것이다.

```java
public BinaryOperator<List<T>> combiner(){
	return (list1, list2) -> {
			list1.addAll(list2)
			return list1;
	}
}
```

병렬화 리듀싱 과정에서 combiner 메서드 활용

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/d504df27-b29f-49f9-b028-4ff07864ef09/Untitled.png)

`combiner ()`메서드를 이용하면 스트림의 리듀싱을 병렬로 수행할 수 있다.
스트림의 리듀싱을 병렬로 수행할 때 자바 7의 포크/조인 프레임워크와 7장에서 배울 Spliterator을 사용한다.

위 그림은 스트림의 병렬 리듀싱 수행과정이다.

- 스트림을 분할해야 하는지 정의하는 조건이 거짓으로 바뀌기 전까지 원래 스트림을 재귀적으로 분할한다.
(보통 분산된 작업의 크기가 너무 작아지면 병렬 수행의 속도는 순차 수행의 속도보다 느려진다.
즉, 병렬 수행의 효과가 상쇄된다는 것이다. 일반적으로 프로세싱 코어의 개수를 초과하는 병렬 작업은 효율적이지 않다)
- 이제 [보여주는 것처럼](https://www.notion.so/6-4dae4311d6264ee39fda82a0f3c1808f?pvs=21) 모든 서브스트림의 각 요소에 리듀싱 연산을 순차적으로 적용해서 서브스트림을 병렬로 처리할 수 있다.
- 마지막에는 컬렉터의 combiner 메서드가 반환하는 함수로 모든 부분결과를 쌍으로 합친다.
즉, 분할된 모든 서브스트림의 결과를 합치면서 연산이 완료된다.

### Characteristics()

컬렉터의 연산을 정의하는 `Characteristics()`형식의 불변 집합을 반환한다.
`Characteristics()`은 스트림을 병렬로 리듀스할 것인지 그리고 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트를 제공한다. `Characteristics`는 다음 세 항목을 포함하는 ***열거형이다***.

1. `UNORDERED` (비정렬, 순서 x) : 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.
2. `CONCURRENT` (동시성) : 다중 스레드에서 `accumulator` 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다. 컬렉터의 플래그에 `UNORDERED`를 함께 설정하지 않았다면 데이터 소스가 정렬되어 있지 않은(즉, 집합처럼 요소의 순서가 무의미한) 상황에서만 병렬 리듀싱을 수행할 수 있다.
    
    <aside>
    ❓ **Q. ? 이게 무슨 상황이란 거지 ? 
    A.**
    
    `Concurrent`(동시성) 컬렉터의 특징 중 하나는 `accumulator` 함수를 다중 스레드에서 동시에 호출하여 스트림의 병렬 리듀싱을 수행할 수 있다는 것입니다. 그러나 이때 데이터 소스가 정렬되어 있지 않으면(즉, 집합처럼 요소의 순서가 무의미한 경우), `UNORDERED` 플래그를 함께 설정하지 않으면 병렬 리듀싱이 제대로 수행되지 않을 수 있습니다.
    
    여기서 "데이터 소스가 정렬되어 있지 않은 상황"이란, 요소의 순서가 의미가 없거나 정해져 있지 않은 상태를 의미합니다. 예를 들어, 집합(`Set`)이나 해시맵(`HashMap`)과 같이 요소의 순서가 중요하지 않은 경우가 해당합니다. 만약 데이터 소스가 리스트(`List`)와 같이 요소의 순서가 의미가 있는 경우에는 병렬 리듀싱을 수행하기 어려울 수 있습니다.
    
    다음은 이를 설명하는 예시
    
    ```java
    import java.util.*;
    import java.util.concurrent.ConcurrentMap;
    import java.util.stream.Collectors;
    
    public class ParallelReductionExample {
        public static void main(String[] args) {
            // 데이터 소스가 정렬되어 있지 않은 경우 (Set과 같이)
            List<String> words = Arrays.asList("apple", "banana", "orange", "grape");
    
            // Concurrent collector를 사용하여 병렬 리듀싱 수행
            ConcurrentMap<Integer, List<String>> result = words.parallelStream()
                    .collect(Collectors.groupingByConcurrent(String::length));
    
            // 결과 출력
            result.forEach((key, value) -> System.out.println(key + ": " + value));}}
    ```
    
    위의 예시에서 `words` 리스트는 정렬되어 있지 않습니다. 따라서 병렬 리듀싱을 수행할 때, `groupingByConcurrent` 메서드를 사용하여 각 단어의 길이에 따라 그룹화한 결과가 충분한 정확성으로 얻어집니다.
    
    하지만, 만약 `words` 리스트가 정렬되어 있다면, 예를 들어, `Arrays.asList("apple", "banana", "grape", "orange")`와 같이 정렬되어 있다면, 병렬 리듀싱을 수행할 때 올바른 결과를 얻기 위해서는 `**UNORDERED**` 플래그를 명시해주어야 합니다.
    
    </aside>
    
3. `IDENTITY_FINISH` (동등한 완료): `finisher()` 가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 이를 생략할 수 있다. 따라서 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다. 또한 누적자 A를 결과 R로 안전하게 형변환할 수 있다.

지금까지 개발한 `ToListCollector`에서 스트림의 요소를 누적하는 데 사용한 리스트가 최종 결과 형식이므로 추가 변환이 필요 없다. 따라서 `ToListCollector`는 `IDENTITY_FINISH`다.
하지만 리스트의 순서는 상관이 없으므로 `UNORDERED`다. 
마지막으로 `ToListCollector`는 `CONCURRENT`다.
하지만 이미 설명했듯이 요소의 순서가 무의미한 데이터 소스여야 병렬로 실행할 수 있다.

<aside>
❓ Q. 위에 구문이 무슨말이야 ?
A.

1. **IDENTITY_FINISH (동등한 완료)**
    - ToListCollector가 IDENTITY_FINISH 특성을 가진다는 것은 최종 결과 형식으로 사용되는 컬렉터의 최종 변환이 필요하지 않다는 것을 의미합니다. 즉, 누적된 데이터 자체가 최종 결과로 사용될 수 있습니다. 일반적으로 리스트의 경우, 최종 변환이 필요하지 않으므로 IDENTITY_FINISH를 가진다고 할 수 있습니다.
2. **UNORDERED (순서 없음)**
    - ToListCollector가 UNORDERED 특성을 가진다는 것은 리스트의 순서가 중요하지 않다는 것을 의미합니다. 따라서 결과 리스트의 순서는 보장되지 않습니다. 이는 요소의 처리 순서가 중요하지 않은 상황에서 유용하게 사용될 수 있습니다.
3. **CONCURRENT (동시성)**
    - ToListCollector가 CONCURRENT 특성을 가진다는 것은 이 컬렉터가 다중 스레드에서 안전하게 병렬로 사용될 수 있다는 것을 의미합니다. 여러 스레드에서 동시에 누적 작업이 수행될 수 있습니다. 하지만, 앞서 언급한대로 요소의 순서가 무의미한 데이터 소스일 때에만 병렬로 실행할 수 있습니다.

따라서 ToListCollector는 최종 결과 변환이 필요 없고, 리스트 순서가 중요하지 않으며, 다중 스레드에서 안전하게 병렬로 사용될 수 있는 특성을 가지고 있습니다.

</aside>

---
