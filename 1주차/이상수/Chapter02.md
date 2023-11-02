# chapter02 동작 파라미터화 코드 전달하기

## 2장 동작 파라미터화 코드 전달하기

동작 파라미터는

- 리스트의 모든 요소에 대해서 ‘어떤 동작’을 수행할 수 있다.
- 리스트 관련 작업을 끝낸 다음에 ‘어떤 다른 동작’을 수행할 수 있다.
- 에러가 발생하면 ‘정해진 어떤 다른 동작’을 수행할 수 있다.

간단하게 표현하면 메서드의 인수로 원하는 동작을 줄 수 있는 것이다.

## 변화하는 요구사항에 대응하기

파라미터를 추가해가며 요구사항에 대응하기

요구 시나리오#1 - 사과목록에서 녹색 사과만 필터

```java
enum Color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory){
		List<Apple> result = new ArrayList<>();
		for(Apple apple : inventory){
				if(GREEN.equals(apple.getColor()){
						result.add(apple);
				}
		}
		return result;
}
```

요구 시나리오#2 - 사과 목록에서 빨간사과도 필터링

기존에 있던 filterGreenApple()로직과 동일한 filterRedApple() 메서드 추가 후 RED.equals만 변경

```java
filterRedApple(){
		...
		if(RED.equals(apple.getColor())
		...
}
```

요구 시나리오#3 - 색을 파라미터화

```java
public static List<Apple> filterGreenApples(List<Apple> inventory, Color color){
		...
		if(apple.getColor().equals(color))
		...
```

요구 시나리오#4 - 과일의 무게 기준으로 구분

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, Color weight){
{
		...
		if(apple.getWeight() > weight) {
		...
```

이렇게 4개의 요구사항 시나리오를 통해 만들어진 각 메서드들은 코드가 계속 중복되고있다.

<aside>
💡 소프트웨어 공학의DRY(Don’t Repeat Yourself)원칙을 어기는 것이다.

</aside>

원칙을 지키기 위해 이 코드를 성능을 개선하기 위해서는 코드를 구현한 메서드 전체 구현을 고쳐야 한다. 이는 엔지니어링적으로 생각했을 때 상당히 비효율적인 방법이다.

반복을 제거하기 위한 방법

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag){
		List<Apple> result = new ArrayList<>();
		for(Apple apple: inventory){
				if((flag && apple.getColor().equals(color)) ||
				(!flag && apple.getWeigth() > weight)){
						result.add(apple);
				}
		}
		return result;
}
```

List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);

List<Apple> heavyApples = filterApples(inventory, null, 150, flase);

로 두 경우 모두 체크가 가능하다.

이 코드는 flag의 이미가 무엇인지 명확히 이해하기도 힘들고, 요구사항이 세부적으로 더 변경되거나 한다면 코드를 수정하기가 곤란해질 것이다.

 

## 2.2 동작 파라미터화

이번엔 사과의 필터를 참 또는 거짓을 반환하는 함수인 프레디케이트를 넣어보자

```java
interface ApplePredicate {

    boolean test(Apple a);

 }

public class AppleHeavyWeightPredcate implements ApplePredicate{
		public boolean test(Apple apple) {
				return apple.getWeight() > 150;
		}
}

public class AppleGreenColorPredicate implements ApplePredicate{
		public boolean test(Apple apple) {
				return GREEN.equals(apple.getColor());
		}
}
```

이렇게 전략적으로 filter를 동작 파라미터로 넣는형식을 전략 디자인 패턴(strategy design pattern)이라고 하고, 주요 체크로직을 캡슐화하여 숨겨둘 수 있고, 내부적으로만 주요로직을 수행하기 때문에 로직 변경이 필요할 때의 범위 설정이 용이하다.

이제 해당 전략을 받는 메서드를 재구현하면

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){

		List<Apple> result = new ArrayList<>();
		for(Apple apple : inventory){
				if(p.test(apple)){
						result.add(apple);
				}
		}
		return result;
}
```

의 코드가 되고 유연성이 늘어난다.

동작 코드를 메서드에 넣음으로써

for문에 의한 탐색 로직이 여러개 있을 필요가 없어졌고, 필요하다면 주요 로직만 ApplePredicate의 구현체로 구현하고, 활용하면 될 것이다.

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate{
		public boolean test(Apple apple){
				return "red".equals(apple.getColor())
						&& apple.getWeight() > 150;
		}
}
```

현재 코드로 봤을 때 매 객체는 p.test(apple)로 test안에 Apple객체를 전달 받아서 구현체를 전달받은 AppleRedAndHeavyPredicate의 구현체인 곳에서 Apple객체를 활용하고 있다.

```java
return "red".equals(apple.getColor())
						&& apple.getWeight() > 150;
```

```java
<주요 비교로직>
return apple.getWeight() > 150;
return "green".equals(apple.getColor());
```

```java
<리스트를 도는 로직>
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){
		List<Apple> result = new ArrayList<>();
		for(Apple apple: inventory){
				if(p.test(apple)){
						result.add(apple);
				}
		}
		return result;
}
```

주요 비교로직이 동작 파라미터화(코드뭉치) 되어

동작 파라미터를 filterApples(ApplePredicate)에 넘겨줘서 해당 판단 비교로직을 if문에 넣는다.

로 정리가 된다.

퀴즈

```java
static class AppleToStringPredicate implements  ApplePredicate{
    
    @Override
    public String toString(Apple apple) {
      return apple.getWeight();
    }
    
  }

  public static void prettyPrintApple(List<Apple> inventory, ApplePredicate p){
    for(Apple apple : inventory){
      String output = p.toString(apple);
      System.out.println(output);
    }
  }
```

## 2.3 복잡한 과정 간소화

이전의 과정에서 코드뭉치를 담는 일을 수행하려면 interface를 선언하고 해당 인터페이스를 구현하는 구현체, 구현체의 내부 로직 구현 등 해야 할 일이 많다.

### 익명클래스

익명클래스를 사용하면 블록 내부에 일회용 클래스를 선언할 수가 있는데, 선언과 동시에 인스턴스화가 된다. 즉 즉석에서 필요할 때 해당 클래스로 구현체를 만들어서 사용할 수도 있다.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate(){
		public boolean test(Apple apple)(
				return RED.equals(apple.getColor());
		}
});
```

하지만 결국 이렇게 클래스를 구현한다면 코드가 많이 복잡해지게 되고, 다른 개발자들에게는 좋지 않은 읽고싶지않은 코드가 될 것이다.

익명 클래스도 나쁜 방법은 아니지만 이를 더 간결하게 활용할 수 있는 람다가 나오고 람다를 주로 활용하게 된 것이다.

### 람다

람다 표현식을 사용하게 되면 아까 만났던 코드가 아래와 같이 변한다

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

상당히 간결해지면서 가독성 또한 좋아졌다.

### 리스트 형식으로 추상화

```java
public interface Predicate<T> {
		boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
		List<T> result = new ArrayList<>();
		for(T e: list){
				if(p.test(e)){
						result.add(e);
				}
			}
		return result;
}
```

```java
List<Apple> redApples = filter(inventory, (Apple apple) → RED.equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) → i % 2 == 0);
```

형식으로 불러오는게 가능하다.

이렇게 직접 함수형 인터페이스를 선언 후 동작파라미터를 T타입으로 넘겨준다면 깔끔한 추상화가 될 것이다.

### 2장을 마치며

<aside>
💡 - 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.

- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄일 수 있다.

- 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다.

</aside>

---

- 가장 대표적인 함수형 인터페이스 4가지

<aside>
💡 자바에서 지원하는 함수형 인터페이스로는 대표적으로 크게 네가지가 있다.

- Supplier<T> - 공급자 : 매개변수는 없고 반환 값만 있다
    - get()
- Consumer<T> - 사용자 : 매개변수는 있고, 반환 값이 없다
    - accept()
- Function<T, R> - 함수 : 일반적인 함수형태, 하나의 매개변수를 받아 결과를 반환
    - apply()
- Predicate<T> - 매개변수 하나를 받아서 boolean타입으로 반환한다
    - test()

T는 제네릭 타입을 뜻하고, R은 리턴 타입을 뜻한다.

추가로 Bi가 붙은 형태가 있는데 다른 방법은 똑같고 매개변수가 두개 들어간다는 의미이다.

</aside>
