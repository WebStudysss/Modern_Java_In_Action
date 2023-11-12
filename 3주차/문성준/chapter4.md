# 4. 스트림이란?

<aside>
❓ **이번 챕터의 내용**
- 스트림이란 무엇인가 ?
- 컬렉션과 스트림
- 내부 반복과 외부 반복
- 중간 연산과 최종 연산

</aside>

# 스트림이란 무엇인가 ?

- 스트림을 이용하면 선언형으로 컬렉션 데이터를 처리할 수 있다.
- 멀티스레드 코드를 구현하지 않아도 데이터를 **투명하게 병렬로 처리**

### 성능 이득

스트림의 새로운 기능이 소프트웨어공학적으로 다음의 다양한 이득을 준다.

1. 선언형으로 코드를 구현할 수 있다.
제어블록을 사용해서 어떻게 동작을 구현할지 지정할 필요 없이 동작의수행을 지정할 수 있다.
    
    → 이는 기존 코드를 복사하여 붙여 넣는 방식을 사용하지 않고 람다 표현식을 이용해서 필요한 부분만 필터링하는 코드도 쉽게 구현할 수 있다는 말이다.
    
2. `filter`, `sorted`, `map`, `collect` 같은 여러 빌딩 블록 연산을 연결해서 복잡한 데이터 처리 파이프라인을 만들 수 있다 → 여러 연산을 파이프라인으로 연결해도 여전히 가독성과 명확성이 유지된다.

fiter 같은 연산은 **고수준 빌딩 블록**으로 이루어져 있다. 
→ 즉, 스트림API 덕에 우린 데이터 처리 과정을 병렬화하면서 스레드와 락 걱정을 할 필요가 없다.

```java
List<Dish> lowCaloricDishes = new ArrayList<>();
for (Dish dish : menu) { //누적자로 요소 필터링
    if (dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
    }
}
Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
    public int compare(Dish dish1, Dish dish2) { //익명 클래스로 요리 정렬
        return Integer.compare(dish1.getCalories(), dish2.getCalories());
    }
});
List<String> lowCaloricDishesName = new ArrayList<>();
for (Dish dish : lowCaloricDishes) {
    lowCaloricDishesName.add(dish.getName()); //정렬된 리스트를 처리하면서 요리 이름 선택
}
```

위의 코드에서 `lowCaloricDishes`는 컨테이너 역할만 하는 중간 변수, 즉 **가비지 변수.**

```java
List<String> lowCaloricDishesName =
                    menu.stream()
                        .filter(d -> d.getCalories() < 400) //400칼로리 이하의 요리 선택
                        .sorted(comparing(Dish::getCalories)) //칼로리로 요리 정렬
                        .map(Dish::getName) //요리명 추출
                        .collect(toList()); //모든 요리명을 리스트에 저장
```

`stream()`을 `parallelStream()`으로 바꾸면 이 코드를 멀티코어 아키텍처에서 병렬로 실행할 수 있다.

### 두 코드 비교 후, 스트림 API 특징

- 선언형 : 더 간결하고 가독성이 좋아진다.
- 조립할 수 있음 : 유연성이 좋아진다.
- 병렬화 : 성능이 좋아진다.

### 스트림이란

 **데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소**

- 연속된 요소 : 컬렉션에선 (LinkedList, ArrayList 중 무엇을 사용할 것인지에 대한) 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룬다.
But 스트림은 filter, sorted, map 처럼 표현 게산식이 주를 이룬다.
즉, 컬렉션 : 데이터, 스트림 : 계산 이 주체가 되는 것이다.
- 소스 : 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다.
정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지된다.
즉, 리스트로 스트림을 만들면 스트림 요소는 리스트의 요소와 같은 순서를 유지한다.
- 데이터 처리 연산 : 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 db와 비슷한 연산을 지원한다. 예를 들어 filter, map, reduce, find, match, sort 등으로 데이터를 조작할 수 있다.
즉, 스트림 연산은 순차적 또는 병렬로 실행할 수 있다.
- **파이프라이닝** : 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프 라인을 구성할 수 있도록 스트림 자신을 반환한다. 그 덕분에 게으름, 쇼트서킷 같은 최적화도 얻을 수 있다.
연산 파이프라인은 데이터 소스에 적용하는 db 질의와 비슷하다.
- **내부 반복** : 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원한다.

```java
import static java.util.Collectors.toList;
List<String> threeHighCaloricDishNames =
		menu.stream() //메뉴에서 스트림 얻기
				.filter(dish -> dish.getCalories() > 300) //고칼로리 요리 필터링
				// 즉, 파이프라인 연산 만들기. 첫 번째로 고칼로리 요리를 필터링한다.
				.map(Dish::getName) // 각각의 요리명 추출
				.limit(3) //정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 축소 truncate
				// 선착순 3개만 선택한다.
				.collect(toList()); //결과를 다른 리스트로 저장
```

- 로직
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/8afebf6e-6474-47a6-b13d-db0c163d5266/Untitled.png)
    

요리 리스트를 포함하는 menu에 stream 메서드를 호출해서 스트림을 얻었다.
여기서 데이터 소스는 요리 리스트(메뉴) → 데이터 소스는 연속된 요소를 스트림에 제공 → 스트림에 filter, map, list, collect로 이어지는 일련의 데이터 처리 연산을 적용 → collect를 제외한 모든 연산은 서로 파이프라인을 형성할 수 있도록 스트림을 반환(파이프 라인은 소스에 적용하는 질의 같은 존재)→ collect 연산으로 파이프라인을 처리해서 결과를 반환(collect는 스트림이 아닌 List 반환)하며, collect를 호출하기 전까지 메서드 호출이 저장되는 효과가 있다.

# **스트림과 컬렉션**

자바의 기존 컬렉션과 새로운 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다. 여기서 ‘**연속된**’이라는 표현은 순서와 상관없이 아무 값에나 접속 하는 것이 아니라 순차적으로 값에 접근한다는 것을 의미한다. 

### 컬렉션과 스트림의 차이

**데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이라고 할 수 있다.**
컬렉션은 현재 자료구조가 포함하는 **모든** 값을 메모리에 저장하는 자료구조다.
즉, 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다(컬렉션에 요소를 추가하거나 컬렉션의 요소를 삭제할 수 있다. 이런 연산을 수행할 때마다 컬렉션의 모든 요소를 메모리에 저장해야 하며 컬렉션에 추가하려는 요소는 미리 계산되어야 한다).

반면 스트림은 이론적으로 **요청할 때만 요소를 계산**하는 고정된 자료구조다(스트림에 요소를 추가하거나 스트림에서 요소를 제거할 수 없다). 사용자가 요청하는 값만 스트림에서 추출한다는 것이 핵심이다. 
결과적으로 스트림은 생산자와 소비자 관계를 형성한다. 또한 스트림은 게으르게 만들어지는 컬렉션과 같다. 
즉, 사용자가 데이터를 요청할 때만 값을 계산한다.

반면 컬렉션은 적극적으로 생성된다(생산자 중심: 팔기도 전에 창고를 가득 채움). 
소수 예제를 적용해보면 컬렉션은 끝이 없는 모든 소수를 포함하려 할 것이므로 무한 루프를 돌면서 새로운 소수를 계산하고 추가하기를 반복할 것이다. 결국 소비자는 영원히 결과를 볼 수 없게 된다.

- 스트림과 컬렉션 비유
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/f7395731-2181-4287-9cba-64065388782e/Untitled.png)
    
- 내,외부 반복
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/2c580f56-5b30-44fc-8b38-e6fc4c869797/Untitled.png)
    
    컬렉션과 스트림의 가장 큰 차이는 외부/내부 반복이다.
    컬렉션은 대상(타겟)을 for-each문을 통해 외부반복을 하지만 스트림은 대상(타겟)을 스트림화해 내부반복.
    
    내부반복을 사용하므로 작업을 투명하고 병렬적으로 처리하거나 더 최적화된 다양한 순서로 처리 가능.
    

### 스트림은 단 한번만 소비 할 수 있다.

```java
List<String> title = Arrays.asList(“java8”, “in”, “action”);
Stream<String> s = title.stream();
s.forEach(System.out::println); // title의 각 단어를 출력
s.forEach(System.out::println); // java.lang.IllegalStateException : 스트립이 이미 소비되었거나 닫힘
```

- 스트림과 컬렉션의 철학적 접근
    
    스트림을 시간적으로 흩어진 값의 집합으로 간주할 수 있다. 반면 컬렉션은 특정 시간에 모든 것이 존재하는 공간(컴퓨터 메모리)에 흩어진 값으로 비유할 수 있다.
    

### 외부/내부 반복

컬렉션과 스트림의 또 다른 차이점은 데이터 반복 처리 방법이다.
컬렉션 인터페이스를 사용하려면 사용자가 직접 요소를 반복해야 한다. 이를 **외부반복**이라고 한다. 
반면 스트림 라이브러리는 반복을 알아서 처리하고 결과 스트림값을 어딘가에 저장해주는 **내부반복**을 사용한다. 스트림 라이브러리의 내부 반복은 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다. 
반면 for-each를 이용하는 외부 반복에서는 **병렬성을 스스로 관리**해야 한다.

```java
// 스트림 동작을 사용해 해당 코드를 리팩터링 해보자
List<String> highCaloricDishes = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while(iterator.hasNext()){
	Dish dish = iterator.next();
	if(dish.getCalories() > 300){
			highCaloricDishes.add(d.getName());
	}
}
// ANSWER : filter 패턴 사용
List<String> highCaloricDishes =
	menu.stream()
			.filter(dish -> dish.getCalories() >300)
			.collect(toList());
```

# **스트림 연산**

스트림 인터페이스의 연산을 크게 두 가지로 구분할 수 있다.

```java
List<String> threeHighCaloricDishNames = menu.stream() // 메뉴(요리 리스트)에서 스트림을 얻는다.
																						 .filter(d -> d.getCalories() > 300) // 중간 연산
																						 .map(Dish::getName) // 중간 연산
																						 .limit(3) // 중간 연산
																						 .collect(toList()); // 스트림을 리스트로 변환. 최종 연산
```

- `filter`, `map`, `limit`는 서로 연결되어 파이프라인을 형성한다.
- `collect`로 파이프라인을 실행한 다음에 닫는다.

연결할 수 있는 스트림 연산을 중간 연산이라고 하며, 스트림을 닫는 연산을 최종 연산이라고 한다. 왜 스트림의 연산을 두 가지로 구분하는 것일까?

### **중간 연산**

`filter`나 `sorted` 같은 중간 연산은 다른 스트림을 반환한다. 따라서 여러 중간 연산을 연결해서 질의를 만들 수 있다. 중간 연산의 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다는 것, 즉 게이르다는 것이다. 중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한 번에 처리하기 때문이다.

```java
// 제품 코드에는 이와 같은 출력 코드를 추가하지 않는게 좋다. 그러나 학습용으로는 매우 좋은 기법이다.

List<String> names =
	menu.stream()
	.filter(dish ->{
					System.out.println("filtering" + dish.getName());
					return dish.getCalories() > 300;
})
	.map(dish -> {
					System.out.println("mapping" + dish.getName());
					return dish.getName();
})
.limit(3)
.collect(toList());
System.out.println(names);

// 실행결과
filtering:pork
mapping:pork
filtering:beef
mapping:beef
filtering:chicken
mapping:chicken
[pork, beef, chicken]
```

스트림의 게으른 특성 덕분에 몇 가지 최적화 효과를 얻을 수 있었다. 
첫째, 300칼로리가 넘는 요리는 여러 개지만 오직 처음 3개만 선택되었다. 이는 `limit` 연산 그리고 **쇼트서킷**이라 불리는 기법 덕분이다. 
둘째, `filter`와 `map`은 서로 다른 연산이지만 한 과정으로 병합되었다(이 기법을 **루프 퓨전**이라고 한다).

### **최종 연산**

최종 연산은 스트림 파이프라인에서 결과를 도출한다. 보통 최종 연산에 의해 `List`, `Integer`, `void` 등 스트림 이외의 결과가 반환된다. 예를 들어 파이프라인에서 `forEach`는 소스의 각 요리에 람다를 적용한 다음에 `void`를 반환하는 최종 연산이다. `System.out.println`을 `forEach`에 넘겨주면 `menu`에서 만든 스트림의 모든 요리를 출력한다. → `menu.stream().forEach(System.out::println);`

```java
//중간 연산 
//				반환형식 | 연산의 인수      | 함수 디스크립터
filter   Stream<T>| Predicate<T>  | T -> boolean
map      Stream<R>| Function<T,R> | T -> R
limit    Stream<T>|
sorted   Stream<T>| Comparator<T> | (T,T) -> int
distinct Stream<T>|
// 최종 연산
				// 반환형식 | 목적
forEach  void            | 스트림의 각 요소를 소비하면서 람다를 적용한다. void를 반환한다.
count    long(generic)   | 스트림의 요소 개수를 반환한다. long을 반환한다.
collect                  | 스트림을 리듀스해서 리스트, 맵, 정수 형식의 컬렉션을 만든다.
```

- [중간연산/최종연산 사용법](https://ittrue.tistory.com/165)
