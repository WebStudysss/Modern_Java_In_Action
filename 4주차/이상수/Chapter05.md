# Chapter05 스트림 활용

## 필터링

**스트림의 요소를 선택하는 방법 필터링**

스트림 인터페이스가 지원하는 filter 메서드의 필터링 방법

- **Predicate**
    - 불린을 반환하는 함수를 인수로 받아서 필터링할 수 있다.
        - stream().filter(Dish::isVegetarian)
        - stream().filter(menu.price > 1000)
- 고유 요소 - **distinct()**
    - 고유 여부는 스트림에서 만든 객체의 hashCode, equals로 결정된다.

## 슬라이싱

**스트림의 요소를 선택하거나 스킵하는 스트림**

- **Predicate** - java9
    - takeWhile() - false가 등장하는 시점 직전까지의 데이터를 슬라이싱
    - dropWhile() - false 요소를 발견한 값 이후 슬라이싱
- **스트림 축소**
    - .limit(n) - 발견하는 요소의 개수 발견 시 종료 → 정렬되지 않은 값에도 사용하기 용이함
- **요소 건너뛰기**
    - .skip(n) - 발견하는 n개의 요소 스킵

## 매핑

**특정 객체에서 특정 데이터를 선택하는 작업**

- 스트림의 각 요소에 함수 적용
    
    ```java
    List<Menu> menu ...
    
    List<String> dishNames = menu.stream()
    														.map(Dish::getName)
    														.collect(toList());
    => Stream<Menu> -> .map(Dish::getName) -> Stream<String>
    ```
    
- 평면화
    - flatMap() → Stream의 객체를 감싸는 것이 아니라 객체 자체를 반환하도록 한다.
    
    ```java
    words.stream()
    		.map(word -> word.split(""))//각 단어를 개별 문자를 포함하는 배열로 변환
    																//Stream<String[]> or Stream<Stream<String>>
    		.map(Arrays::stream) //String[]배열을 stream으로 감싸기
    		.distinct()
    		.collect(toList());
    		//List<Stream<String>> 반환 -> 실패
    
    words.stream()
    		.map(word -> word.split(""))
    		.flatMap(Arrays::stream) // flatMap을 통해 만들어진 Stream<Stream<String>>을
    														//Stream<String>으로 평면화
    		.distinct()
    		.collect(toList());
    ```
    

flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다.(스트림의 언박싱 개념으로 이해)

## 검색

- 특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리 기법 반환값은 Boolean
    - allMatch - 모두만족
    - anyMatch - 하나라도만족
    - noneMatch - 모두만족x
    - findFirst
    - findAny
- 쇼트서킷
    
    limit, allMatch, noneMatch, findFirst, findAny등의 Stream연산은 스트림의 모든 요소를 처리하지 않고도 결과 반환이 가능하다. 이러한 연산들을 `쇼트서킷` 연산이라고 부른다.
    

## 요소 검색

findAny 메서드는 현재 스트림에 있는 임의의 요소를 반환한다.

```java
Dish dish = menu.stream()
		.filter(Dish::isVegetarian)
		.findAny();
 -> 컴파일 에러 (형태 불일치 findAny() -> Optional<Dish> 반환)
```

쇼트서킷을 통해 결과를 바로 받았지만, 이번에는 Optional이라는 개념이 나온다.

### Optional이란?

만약 List<Menu> menu 안에 아무 값도 없었다면, 또는 filter를 통해 야채를 걸렀을 때 무조건 유효한 값이 나오느냐? 묻는다면 나오지 않을 수도 있다. 아무 값이 없으면 즉시 null pointer 에러가 발생할 수 있기 때문에 null 을 반환하지 않도록 검사할 수 있는 클래스인 Optional이 자바8 버전 부터 등장하게 된 것이다.

- **isPresent()** : Optional이 값을 갖고 있다면 true 반환
- **ifPresent**(Cunsumer<T> block) : 값이 있을 때 주어진 블록을 실행한다.
- **T get**() : 값이 존재하면 값을 반환, 값이 없으면 NoSuchElementException 발생
- **T orElse**(T other) : 값이 있으면 값을 반환하고, 값이 없으면 기본값을 반환

### 첫 번째 요소 찾기

일부 스트림은 논리적인 아이템 순서가 정해져 있을 수 있는데 해당 순서에서 첫 번째 요소를 찾으려면 findFirst()를 사용하면 된다.

해당하는 값이 빛을 볼 때는 병렬 스트림을 할 때 발휘할 것이다.

## 리듀싱

**스트림의 요소를 처리해서 값으로 도출하는 연산 ⇒ 리듀싱 연산 or 폴드**

### 요소의 합

```java
int sum = 0;
for(int x : numbers){
		sum += x;
}

->

int sum = numbers.stream()
								.reduce(0, (a, b) -> a + b);
```

- **초깃값(identity)** 0
- 두 요소를 조합해서 새로운 값을 만드는 **BinaryOperator<T>**
    - 누적합, 누적곱, max, min 등등

→ 정수 reduce가 아닌 Map리듀스도 가능한데 이후 7장에서 다룸

## 숫자형 스트림

**기본형을 일반 스트림에 담는다면..**

- **박싱 언박싱 비용 증가**
- 일관된 데이터가 있기에 .sum()추가
- .boxed()
- .mapToInt()
- .rangeClosed()

### 기본형 스트림의 반환형태 OptionalInt

- .max()등 IntStream의 처리 이후 조건에 들어맞는 것이 하나도 없을 때의 경우를 위해 Optional로 반환

## 스트림을 만드는 다양한 방법

- **Stream.of()**
    - Stream.empty()
- **Arrays.stream()**
- **Stream.ofNullable()**
- 무한 스트림
    - **Stream.iterate(seed, UnaryOperator)**
        - `기존의 값을 가진 seed를 활용해서 무한 스트림 생성`
        - limit()
        - takeWhile()
    
    !무한 스트림을 사용할 때 주의점 .limit()같은 **쇼트서킷** 메서드를 활용해야한다.
    
    ```java
    //fibonachi
    Stream.iterate(new int[]{0, 1}, t -> new int[]{t[1], t[0]+t[1]})
    				.limit(20)
    				.forEach(System.out::println);
    ```
    
    - Stream.generate()
        - `기존의 값이 없는 무한 스트림 생성`
        - 물론 똑같이 값을 다른 객체에서 받거나 하는 방식으로 구현할 수는 있겠지만, 객체의 기존 상태를 바꾸지 않는 **불변성**이 깨지게 될 것이다. 이러한 방법은 병렬Safe하지 않기 때문에 권장하지 않는다.

- 필터링
- 슬라이싱
- 매핑
- 검색
- 매칭
- 리듀싱
- 기본형 스트림
- 스트림 생성

### 스트림 연산의 개념에 대한 최종 정리

스트림 API의 동작방식의 특징으로는 크게 세가지

- **Lazy - 게으름**
    - 최초 시작 시 연산을 바로 하는 것이 아닌 파이프라인만 설정해준 후 중간 → 최종 연산을 거쳐감
- **Short Circuit - 끊어진 순회**
    - 각 스트림의 요소에 대한 중간연산 파이프라인을 처리 후 최종연산 → 최종연산의 조건에 부합하는 스트림이 있다면 그 상태에서 **스트림 순회 종료!**
- **Loop Fusion - 혼합된 루프**
    - 중간 연산의 메서드 체인들을 한번에 처리하므로 필터 → 모든객체 필터 → 정렬 → 모든객체 정렬 의 방식이 아님

때문에 

```java
List<Dish> menu = Dish.menu;
		//1, 2, 3, 4
    List<String> names = menu.stream()
            .filter(dish -> {
              System.out.println("filtering:" + dish.getName());
              return dish.getCalories() > 300;
            })
            .map(dish -> {
              System.out.println("mapping:" + dish.getName());
              return dish.getName();
            })
            .sorted()
            .limit(3)
            .collect(Collectors.toList());
    System.out.println(names);
```

쇼트서킷이 들어간다면 sorted같은 중간연산은 사용하기 애매함

이번에 스트림 특성으로 Lazy, Short Circuit, Loop fusion 이렇게 세가지를 좀 알아보았는데, 각 특성 모두가 한번에 활용되는 것은 아니고 용도에 맞게 잘 사용하는 것이 이로울 것 같다.

https://velog.io/@joosing/lazy-java-stream
