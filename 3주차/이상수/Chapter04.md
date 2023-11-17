# Chapter04 스트림 소개

## 컬렉션의 역할

컬렉션은 자바에서 빠져서는 안되는 데이터를 그룹화하는 도구다.

모든 자바 애플리케이션에서 만들고 처리하고 있는 컬렉션이지만, 이 컬렉션에서 지원해주는 연산을 지원하지않는다.

때문에 우리가 해당 연산을 처리할 때 데이터를 처리하는 코드를 구현하는데, 해당 구현 방식이 다른 개발자가 봤을 때 명시적이지 않아 명확하지 않을 수 있다. 또 많은 요소를 포함하는 컬렉션은 처리 성능을 높이려면 병렬처리가 불가피한데, 병렬 처리 코드는 어렵다.

헛둘님

## 스트림이란 무엇인가

스트림은 자바 8 API에 추가된 기능이다.

### 스트림 API의 장점

- 컬렉션 `데이터 반복`을 편리하게 처리할 수 있다.
- 데이터를 멀티스레드를 활용하지 않아도 `투명하게 병렬`로 처리할 수 있다.
- `선언형`으로 코드를 구현할 수 있다.
    - 루프와 if조건문 등의 블록 구현이 필요없음
- 여러 빌딩 블록 연산을 연결해서 `복잡한 데이터 처리 파이프라인`을 만들 수 있다.
- 선언형과, 블록연산을 파이프라인으로 연결하기 때문에 `가독성과 명확성`이 유지된다.

<aside>
💡 p138 스트림api는 매우 비싼연산이다.의 예시?

한 블록에 수많은 데이터처리 연산이 필요하기 때문에 비싼 연산이라고 생각했습니다.

</aside>

gpt답변

![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/13a50f75-de7c-4701-a9ce-c8c8342f6cb9)


### 자바 8의 스트림API의 특징

- 선언형 : 더 간결하고 가독성이 좋아진다.
- 조립할 수 있음 : 유연성이 좋아진다.
- 병렬화 : 성능이 좋아진다.

## 스트림 시작하기

자바 8부터 Collections에서 스트림을 반환하는 stream() 메서드가 추가되었다.

스트림을 정의하자면 `데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소` 이다.

- **연속된 요소** : 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공한다.
    - 컬렉션의 주제는 `데이터` 스트림의 주제는 `계산` 이다.
- **소스** : 스트림은 컬렉션, 배열 등의 자원 데이터 제공 소스를 소비할 뿐이다. 소스를 그대로 사용하기 때문에 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지된다.
- **데이터 처리 연산** : 데이터베이스와 비슷한 연산을 지원한다. filter, map, reduce, find, match, sort… 등으로 데이터를 조작할 수 있는 연산을 지원한다.
- `파이프라이닝(Pipelining)` : 스트림 연산 대부분은 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다.
    - `게으름`, `쇼트서킷` 등과 같은 최적화도 얻을 수 있다.(5장 설명)
    - 연산 파이프라인은 데이터 소스에 적용하는 데이터베이스 질의와 비슷하다.
- `내부 반복` : 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원한다.

### 스트림의 특징을 예제에 적용

```java
import static java.util.stream.Collectors.toList;
public static List<String> getLowCaloricDishesNamesInJava8(List<Dish> dishes) {
    return dishes.stream() // 메뉴(요리 리스트)에서 스트림을 획득
				// 파이프라인 연산 시작
        .filter(d -> d.getCalories() < 400) // 1. 고칼로리 요리를 필터링으로 제외
        .sorted(comparing(Dish::getCalories)) // 2. 요리명 추출
        .map(Dish::getName) // 3. 선착순 세개만 선택
        .collect(toList()); // 4. 을 통해 나온 결과를 다른 리스트로 저장
  }
```

```java
!!
.collect(toList()) -> add, remove 가능
.collect(Collectors.toUnmodfiableList()) -> 자바10
.toList() -> 자바16

불변 리스트, 변환가능한 리스트
List <- new ArrayList 변환가능
List.of() Arrays.asList() -> 불변리스트 set가능
toUnmodfiableList -> 불변리스트지만 List.of와 다름
```

- 데이터 소스 : 요리 리스트 → `연속된 요소를 스트림에 제공`
- filter, sorted, map, collect : 데이터 처리 연산 - collect()를 제외한 연산은 `파이프라인을 형성할 수 있도록 스트림을 반환`
    - **filter** : 람다를 인수로 스트림에서 특정 요소를 제외시킨다.
    - **map** : 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보를 추출
    - **limit** : 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 크기를 축소
    - **collect :** 스트림을 다른 형식으로 변환한다. toList()는 스트림을 리스트로 변환

## 스트림과 컬렉션

컬렉션과 스트림은 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다.

여기서 `연속된` 이라는 표현은 순차적으로 값에 접근한다는 것을 의미한다.

이 둘은 연속된 값을 접근해서 처리하는 방식인데 무엇이 다른가? 했을 때

데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이라고 꼽을 수 있다.

### 스트림 vs 컬렉션

- 스트림
    - `요청할 때만 요소를 계산`하는 고정된 자료구조
    - 적극적 생성 : 모든 값을 계산할 때까지 기다림
    - `내부 반복(internal iteration)`
- 컬렉션
    - 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다.
    - 게으른(Laziness) 생성 : 필요할 때만 값을 계산
    - `외부 반복(external iteration)`

### 딱 한 번만 탐색할 수 있다.

Iterator와 동일하게 한번만 사용되고 버려진다. 재탐색을 위해서는 초기 데이터 소스에서 새로운 스트림을 만들어야 함

```java
List<String> title = Arrays.asList("Java8", "In", "Action");
Stream<String> s = title.stream();
s.forEach(System.out::println);
s.forEach(System.out::println); -> IllegalStateException 발생!
```

### 외부 반복 보다 내부 반복이 좋다

명시적으로 처리 된 컬렉션 목록을 가져와서 처리하고, 처리한 컬렉션을 다시 다른 컬렉션에 담아서 처리하고… 방식이 상당히 번거롭다.

작업을 실행할 때 명시적으로 값 처리를 바로 바로 담아서 넣어줄 수 있고, 값을 처리할 때 명시적으로 어떤 일을 행하는지 볼 수 있다.

또 병렬처리를 관리하는 부분에서 좋다.

### 예시 ) 외부 반복 vs 내부 반복

```java
<외부 반복>
List<String> highCaloricDishes = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while(iterator.hasNext()){
		Dish dish = iterator.next();
		if(dish.getCalories() > 300){
				highCaloricDishes.add(d.getName());
		}
}

->

<내부 반복>
List<String> highCaloricDish = menu.stream()
		.filter(dish -> dish.getCalories() > 300)
		.map(Dish::getName)
		.collect(toList());
```

## 스트림 연산

이제 스트림의 연산 방식을 예시와 함께 명칭으로 구분을 해보자.

```java
public static List<String> getLowCaloricDishesNamesInJava8(List<Dish> dishes) {
    return dishes.stream() // 요리 리스트에서 스트림 얻기
        .filter(d -> d.getCalories() < 400) // 중간 연산
        .sorted(comparing(Dish::getCalories)) // 중간 연산
        .map(Dish::getName) // 중간 연산
        .collect(toList()); // 최종 연산
  }
```

- 중간 연산
    - 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다는 것, 즉 `게으르다` 중간 연산을 합친 다음에 합쳐진 중간 연산 결과를 기반으로 최종 연산으로 한번에 처리하기 때문이다.
    - 쇼트서킷 기법을 통해 limit연산을 통해 갯수제한이 가능하다
    - `루프 퓨전` : 서로 다른 연산들을 한 과정으로 병합시키기도 한다.
- 최종 연산
    - 중간 연산을 다 마친 후에 나온 결과를 최종 처리하는 연산이다.

### 스트림 이용하기

스트림은 세 가지 방법으로 요약할 수 있다.

- 질의를 수행할 데이터 소스
- 스트림 파이프라인을 구성할 중간 연산 연결
- 스트림 파이프라인을 실행하고 결과를 만들 최종 연산

중간 연산 종류

- filter : Predicate
- map : Function
- limit : 개수제한
- sorted : 정렬
- distinct : 중복 제거

최종 연산 종류

- forEach : 각 요소를 소비하면서 람다 적용
- count : 스트림의 요소 개수 반환
- collect : 스트림을 리듀스해서 컬렉션을 만든다.
