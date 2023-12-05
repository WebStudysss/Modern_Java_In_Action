### 외부반복과 내부반복
---
- 외부 반복 : 컬렉션 반복을 명시적으로 관리
```java
List<Dish> vegetarianDisheds = new ArrayList<>();
for(Dish d : menu) {
	if(d.isVegetarian()) {
    	vegetarianDisheds.add(d);
    }
}
```

- 내부 반복 : 스트림 API를 이용해 데이터 컬렉션 반복을 내부적으로 처리
```java
List<Dish> vegetarianDishes = menu.stream()
	.filter(Dish::isVegetarian)
    .collect(toList());
```

데이터를 어떻게 처리할지 내부적으로 스트림 API가 결정한다. 외부 반복과 내부 반복의 가장 큰 차이는 병렬로 실행할지 여부를 결정하는 것인데 외부 반복은 단일 스레드로 구현되어 있기 때문에 병렬로 실행하지 못한다.

### 필터링
---
- filter()

Stream에서 filter 메서드는 predicate를 인수로 받아서 일치하는 모든 요소를 포함하는 스트림을 반환한다.
```java
List<Dish> vegetarianDishes = menu.stream()
	.filter(Dish::isVegetarian)
    .collect(toList());
```

![](https://velog.velcdn.com/images/bw1611/post/f81f1bf3-9512-4874-83a5-709ca9cc80d7/image.png)

- distinct()

고유 요소로 이루어진 스트림을 반환하는 메서드, 즉 중복된 값을 지우고 한개의 값만 가지고 온다.

### 슬라이싱
---
- takeWhile()

```java
		Stream.of(1,2,3,4,5,6,7,8,9)
                .filter(n -> n%2 == 0)
                .forEach(System.out::println); // 결과 : 2 4 6 8

        Stream.of(2,4,3,4,5,6,7,8,9)
                .takeWhile(n -> n%2 == 0)
                .forEach(System.out::println); // 결과 : 2 4
```
takeWhile은 false가 등장하는 순간부터 반복을 멈추고 뒷부분은 잘라낸다. 그에 비해 filter는 false 값을 만나더라도 정해진 반복은 끝까지 도는 것을 확인할 수 있다. 즉, takeWhile은 filter와 다르게 false 값을 만나면 바로 중단한다.


- dropWhile()

```java
    Stream.of(2,4,3,4,5,6,7,8,9)
            .dropWhile(n -> n%2 == 0)
            .forEach(System.out::println); // 결과 : 3 4 5 6 7 8 9
```

dropWhile은 takeWhile과 반대로 false 값이 나오면 전까지의 값은 모두 버리고(작업 중단) 남은 요소를 가지고 온다. 또한 무한 스트림에서도 동작을 한다.

```java
    Stream.iterate(1, i -> i + 1)
            .dropWhile(i -> i < 5)
            .limit(50) // limit을 설정해두지 않으면 무한히 반복한다(무한 스트림)
            .forEach(System.out::println);
```

- limit()

```java
- limit()
    Stream.iterate(1, i -> i + 1)
            .dropWhile(i -> i < 5)
            .limit(3)
            .forEach(System.out::println); // 결과 : 5 6 7
```

limit 값 이하의 크기를 갖는 새로운 스트림을 반환한다. limit이 3이라면 최대 3개를 반환할 수 있다.

- skip()

처음 n개의 요소를 제외한 스트림을 반환하는 메서드이다.

```java
    Stream.of(1,2,3,4,5,6,7,8,9)
            .skip(3)
            .forEach(System.out::println); // 결과 : 4 5 6 7 8 9
```

limit과 반대로 skip은 인자로 받은 값 만큼을 건너뛰고 뒤에 값부터 출력한다.

### 매핑
---

- map()

함수를 인수로 받는 map 메서드다, 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다.
```java
        List<String> list = Arrays.asList("a1", "a2", "b1", "b2", "c2", "c1", "c3")
                .stream()
                .map(String::toUpperCase).toList(); // -> [A1, A2. B1, --- C3]
```

즉, Stream의 요소를 다른 형태로 변경이 가능합니다. 인자로 전달되는 함수를 구현하여 요소를 어떻게 변경할지 설정한다.

만약 ["Hello", "World"] 리스트가 있다면 Split을 사용하여 ["H", "e", "l", "l", "o", --- "l", "d"]와 같이 만들 수 있다고 생각할 것이다. 하지만 아래와 같이 구현한다면 문제가 있다.

```java
    List<String[]> collect = words.stream()
            .map(w -> w.split(""))
            .distinct()
            .collect(toList());
    for (String[] strings : collect) {
      System.out.println(Arrays.toString(strings));
    }
```

결과
```
[H, e, l, l, o]
[W, o, r, l, d]
```

반환형식을 보면 `List<String[]>`으로 반환되는 점이 문제인 것이다. 그렇다면 `Stream<String>` 으로 반환하기 위해서는 어떻게 해야할까?

- flatMap()

flatMap을 사용하여 문제를 해결할 수 있다.

```java
    words.stream()
        .flatMap((String line) -> Arrays.stream(line.split("")))
        .distinct()
        .forEach(System.out::println);
```

flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑을 한다. 즉 하나의 평면화된 스트림으로 반환을 하며, 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다.

### 검색과 매칭
---

-anyMatch()

스트림에서 적어도 한 요소와 일치하는지 확인할 때 사용하는 메서드
```java
        boolean isTrue = new ArrayList<>(Arrays.asList(10, 20, 30, 40, 50))
                .stream()
                .anyMatch(n -> n > 30);
        System.out.println(isTrue); // 결과 : true
```

리스트 중 하나라도 조건을 만족하면 true를 반환하고 아무도 만족하지 못하면 false를 반환한다.

- allMatch()

anyMatch()와 달리 모든 요소가 주어진 조건을 만족하는지 검사한다.

```java
        boolean isTrue = new ArrayList<>(Arrays.asList(10, 20, 30, 40, 50))
                .stream()
                .allMatch(n -> n > 30);
        System.out.println(isTrue); // 결과 : false
```

하나라도 만족하지 못하면 false값을 반환하고 모든 값이 만족하면 true값 반환한다.

- nonMatch()

allMatch()와 반대 연산을 수행하는 메서드로 조건을 만족하는 요소가 없는지 확인한다.
```java
        boolean isTrue = new ArrayList<>(Arrays.asList(20, 30, 10, 30, 20))
                .stream()
                .noneMatch(n -> n > 30);
        System.out.println(isTrue);
```
하나라도 만족하면 false값을 모든 값이 만족하지 못하면 true값을 반환한다. (allMatch()와 정확히 반대라고 생각하면 편한다.)

anyMatch, allMatch, nonMatch는 스트림의 쇼트서킷 기법을 수행한다.

> Java의 쇼트서킷이란?
논리연산자 AND, OR 을 나타내기 위해 부호 &&,  || 을 사용하는 것을 의미한다.

- findAny()

현재 스트림에서 임의의 요소를 반환한다.

```java
        Optional<Integer> any = new ArrayList<>(Arrays.asList(20, 30, 10, 34, 20))
                .stream()
                .filter(n -> n > 20)
                .findAny();
        System.out.println(any); // 결과 : 30
```

위의 반환타입을 보면 Optional을 확인해볼 수 있는데 findAny값에는 null이 들어갈 수 있기 때문에 NPE 버그를 피하기 위하여 Optional로 반환형식이 정해졌다.

- findFirst()

리스트 또는 정렬된 연속의 데이터로부터 생성된 스트림에는 논리적인 아이템순서가 정해져 있을 수 있기 때문에 첫번째 요소를 찾는 findFirst 메서드가 있다.

```java
Optional<Integer> any = new ArrayList<>(Arrays.asList(20, 30, 10, 34, 20))
                .stream()
                .filter(n -> n > 20)
                .findFirst();
        System.out.println(any); // 결과 : 30
```

### 리듀싱
---
조금 더 복잡한 질의를 표현할 수 있는 것이 리듀싱이다. 예를 들어, 컬렉션 내부에 대해 총합을 구한다거나 최대값, 최소값을 구하는 연산을 확인할 수 있다.

```java

	// 람다 형식
    List<Integer> numbers = Arrays.asList(3, 4, 5, 1, 2);
    int sum = numbers.stream().reduce(0, (a, b) -> a + b);
    System.out.println(sum); // 결과 : 15
    
    // 메서드 참조
    int sum2 = numbers.stream().reduce(0, Integer::sum);
```

위와 같이 모든 연산 `"+", "-", "*", "/", "%"` 이 가능하다.

```java
	// 람다 형식
    int max = numbers.stream().reduce(0, (a, b) -> Integer.max(a, b));
    System.out.println(max); // 결과 : 5
    
    // 메서드 참조
    Optional<Integer> max2 = numbers.stream().reduce(Integer::max);
```

위의 람다형식은 Optional을 반환되지 않지만 메서드참조형식으로 진행할 경우 Optional로 반환되는 것을 확인해 볼 수 있는데 이유는 스트림이 비어있을 경우 null값을 반환하기 위해서 입니다. (비어있을 경우 명시적처리가 어려움)

![](https://velog.velcdn.com/images/bw1611/post/0390e00f-544b-418a-8c07-01421f6a41e3/image.png)


### 숫자형 스트림
---

```java
    Integer reduce = transactions.stream()
            .map(Transaction::getValue)
            .reduce(0, Integer::sum); // 이 부분에서 박싱이 실행됨
```

map메서드가 `Stream<T>`를 생성하기 때문에 기본형 원시타입 Integer가 파라미터로 들어오면 오토박싱이 발생하게 된다. 이런 숫자 스트림을 효율적으로 처리하기 위하여 Stream에서는 기본형 특화 스트림을 제공한다

- **기본형 특화 스트림**

```java
    Integer sum = transactions.stream()
            .mapToInt(Transaction::getValue)
            .sum();

    System.out.println(sum);
```

위와 같이 mapToInt를 통하여 `Stream<T>` 대신 특화된 Integer스트림을 반환할 수 있기 때문에 sum() 메서드를 사용할 수 있다.

기본형 특화 스트림에 종류에는 `DoubleStream, IntStream, LongStream`이 있다.

```java
    Stream<Integer> boxed = transactions.stream()
            .mapToInt(Transaction::getValue)
            .boxed();
```

반대로 기본형 특화 스트림으로 바꾼 것을 boxed()를 사용하여 Stream형태로 다시 박싱해줄 수 있다.

- **OptionalInt**

최대값을 찾을 때 0이라는 값 때문에 잘못된 결과가 도출될 수 있는 경우가 있다. 요소가 없는 상황과 실제 최대값이 0인 상황을 구별하기 위해서 사용한다. (OptionalInt, OptionalLong, OptionalDouble 등이 있다.)

```java
    OptionalInt max = transactions.stream()
            .mapToInt(Transaction::getValue)
            .max();
```

- **숫자범위**
특정 범위의 숫자를 이용해야하는 경우 사용할 수 있다.
  - ranged()
  시작값과 종료값이 결과에 포함되지 않는 범위
    
  - rangedClosed()
  시작값과 종료값이 결과에 포함되는 범위
  
```java
    List<Integer> collect = IntStream.range(0, 5)
            .mapToObj(k -> k * 2)
            .collect(toList()); // 0, 2, 4, 6, 8

        List<Integer> collect = IntStream.rangedClosed(0, 5)
            .mapToObj(k -> k * 2)
            .collect(toList()); // 0, 2, 4, 6, 8, 10
```

### 스트림 만들기
---
일련의 값, 배열, 파일, 함수를 이용한 무한 스트림 만들기 등 다양한 방식으로도 스트림을 사용 가능하다.

- Stream. ofNullable (null이 될 수 있는 객체 스트림화)

명시적으로 확인해야 했던 null 값을 ofNullable을 통해 쉽게 구현할 수 있다.

```java
    Map<String, List<String>> map = new HashMap<>();
    map.put("metropolitan", Arrays.asList("Seoul", "Incheon"));
    map.put("Jeolla", List.of("Gwangju"));
    map.put("Gyeongsang", Arrays.asList("Ulsan", "Daegu", "Busan"));

    Stream<String> cityNamesStream =
            Stream.ofNullable(String.join("", map.get("metropolitan")));
    cityNamesStream.forEach(System.out::println); // SeoulIncheon

    cityNamesStream = Stream.ofNullable(String.join("", map.get("Gangwon")));
    cityNamesStream.forEach(System.out::println); // 빈 스트림 NPE 발생
```

- 배열로 스트림 만들기
```java
	int[] numbers = {2, 3, 5, 7};
    int sum = Arrays.stream(numbers).sum(); // 17
```

정적 메서드 Arrays.stream을 이용하여 스트림을 만들 수 있다.

- 함수로 무한 스트림 만들기

함수에서 스트림을 만들 수 있는 두 정적 메서드 Stream.iterate / Stream.generate 를 제공한다. 이 두 연산을 이용하여 무한 스트림을 만들 수 있다. (고정되지 않은 스트림)

- Stream.iterate
```java
    Stream.iterate(0, n -> n + 2)
            .limit(5)
            .forEach(System.out::println); // 0, 2, 4, 6, 8
```

요청 받을 때마다 값을 생산하여 무한 스트림을 생산한다. (0 + 2 = 2 + 2 = 4 이런식으로?) 또한 predicate 형식도 지원한다

```java
    Stream.iterate(0, n -> n < 20, n -> n + 2)
            .forEach(System.out::println);
```

- Stream.generate

iterate와 달리 generate는 값을 연속적으로 생산하지 않는다는 차이점이 있다. 또한 seed도 사용하지 않는다.

```java
    Stream.generate(Math::random)
            .limit(10)
            .forEach(System.out::println);
```
