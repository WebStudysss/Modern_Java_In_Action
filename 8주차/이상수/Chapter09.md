# Chapter09 리팩터링, 테스팅, 디버깅

# 시작하며

이번 장에서는 기존 코드를 자바 8 이상의 버전에 맞춰 변경하는 형태로 이루어진 가독성과 유연성에 초점을 맞춘 장이다.
전략(strategy), 템플릿 메서드(template method), 옵저버(observer), 의무 체인(chain of responsibility), 팩토리(factory) 등의 패턴을 학습 예정

# 가독성과 유연성을 개선하는 리팩토링

## 코드 가독성 개선

코드 가독성은 보통 코드를 구현하지 않은 다른 사람들이 코드를 봤을 때 쉽게 이해할 수 있는 코드를 두고 가독성이 좋은 코드 라고 한다. 때문에 이렇게 가독성이 좋은 코드를 어떻게 만들지 리팩토링 예제를 통해 만날 것이다.

## 익명 클래스를 람다 표현식으로 리팩토링하기

```java
Runnable r1  =new Runnable(){
            public void run(){
                System.out.println("Hello");
            }
        };
        
        ->

Runnable r2 = () -> System.out.println("Hello");
```

### 람다 표현식으로 리팩토링할 때 주의점

- 익명 클래스에서 사용한 this와 super는 람다에서 다른 의미를 갖는다.
    - 익명 : 클래스 내의 범위
    - super : 람다를 감싸고 있는 클래스
- 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다(익명 클래스 바깥에 있는 변수네이밍 사용 가능)

## 람다 표현식을 메서드 참조로 리팩토링하기

```java
menu.stream().collect(
              groupingBy(Dish::getType, mapping(dish -> {
                  if(dish.getCalories() <= 400) return CaloricLevel.DIET;
                  else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                  else return CaloricLevel.FAT;},
              toCollection(TreeSet::new))));

->

menu.stream()
	.collect(groupingBy(Dish::getCaloricLevel));

class Dish{
...
	public Grouping.CaloricLevel getCaloricLevel() {
	    if(this.getCalories() <= 400) return Grouping.CaloricLevel.DIET;
	    else if(this.getCalories() <= 700) return Grouping.CaloricLevel.NORMAL;
	    else return Grouping.CaloricLevel.FAT;
	  }
...
}
```

- 위의 코드와 같이 구현에 대한 상세 코드를 클래스에 맡기면, stream의 흐름 파악이 용이해짐
- Collectors에 있는 정적 헬퍼 메서드를 활용하여 간결하게 구현 가능(comparing, maxBy, summingInt 등)

## 명령형 데이터 처리를 스트림으로 리팩토링하기

```java
List<String> dishNames = new ArrayList<>();
for(Dish dish: menu){
	if(dish.getCalories() > 300){
		dishNames.add(dish.getName());
	}
}

->

menu.parallelStream()
	.filter(d -> d.getCalories() > 300)
	.map(Dish::getName)
	.collect(toList());
```

기존의 필터링과 추출을 같이 사용하는 형태의 명령형 패턴 사용 시 코드의 이해가 아래의 stream 형태보다 어렵고, 병렬화 또한 어렵다.

## 코드 유연성 개선

동작 파라미터화를 활용한 조건부 연기 실행(conditional deferred execution), 실행 어라운드(execute around) 패턴으로 리팩토링 하는 방법

- 조건부 연기 실행
    - Logger클래스의 예시(p300)
    - Logger클래스의 정보를 캡슐화, 지연실행되는 람다식 활용
    - 람다식을 사용한 log와 사용하지 않은 log의 차이점
        - 람다식을 사용했을 때는 실제 사용하는 시점에 해당 supplier가 작동
        - 즉, 지연된 계산이 이루어지기 때문에, 자원효율성이나 속도 측면에서 더 좋다.
- 실행 어라운드
    - 준비, 종료 과정을 반복적으로 수행하는 코드를 람다로 변환이 가능

# 람다로 객체지향 디자인 패턴 리팩토링하기

공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는, 검증된 청사진을 제공하는 디자인 패턴에 대해서 배울 예정

- 전략(Strategy)
- 템플릿 메서드(Template Method)
- 옵저버(Observer)
- 의무 체인(Chain Of Responsibility)
- 팩토리(Factory)

## 전략

한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법

![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/061ad6a2-5c73-43be-b776-c3b7e75a6d99)


- 알고리즘을 나타내는 인터페이스(Strategy 인터페이스)
- 다양한 인터페이스 구현체(ConcreteStrategyA, B)
- 전략 객체를 사용하는 한 개 이상의 클라이언트

**문자와 숫자를 구분하는 알고리즘을 사용할 때의 예시**

```java
static private class Validator {

    private final ValidationStrategy strategy;

    public Validator(ValidationStrategy v) {
      strategy = v;
    }

    public boolean validate(String s) {
      return strategy.execute(s);
    }

  }

interface ValidationStrategy {
    boolean execute(String s);
  }

  static private class IsAllLowerCase implements ValidationStrategy {

    @Override
    public boolean execute(String s) {
      return s.matches("[a-z]+");
    }

  }

  static private class IsNumeric implements ValidationStrategy {

    @Override
    public boolean execute(String s) {
      return s.matches("\\d+");
    }

  }

public static void main(String[] args) {
		Validator v1 = new Validator(new IsNumeric());
    System.out.println(v1.validate("aaaa"));
    Validator v2 = new Validator(new IsAllLowerCase());
    System.out.println(v2.validate("bbbb"));
		Validator v3 = new Validator((String s) -> s.matches("\\d+"));
}
```

- interface인 ValidationStrategy를 사용하고 해당 구현체를 main 실행 시점에 불러오는 형태
- 재사용하지 않을 코드라 생각된다면 람다를 통해 한번에 처리해도 괜찮다.

## 템플릿 메서드

알고리즘의 방향성은 일치하지만, 일부를 고쳐야 할 때 사용하는 디자인 패턴

```java
abstract class OnlineBanking {

  public void processCustomer(int id) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy(c);
  }

  abstract void makeCustomerHappy(Customer c);

----

public static void main(String[] args) {
    new OnlineBankingLambda()
			.processCustomer(1337, (Customer c) -> System.out.println("Hello!"));
  }

public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
  }
```

- 동일한 방향성을 가진 알고리즘 `processCustomer` , `makeCustomerHappy`는 은행 별로 동작 방법이 다름
- 람다를 활용하면 템플릿 메서드 패턴에서 사용되는 자잘한 코드를 제거할 수 있다.

## 옵저버

이벤트가 발생했을 때 한 객체(주체 subject)가 다른 객체 리스트(관찰자 observer)에 자동으로 알림을 보내야 하는 상황에 사용

![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/5e786a08-e6a3-4b0b-9e12-a1443ac1a8a4)


- Observer 인터페이스와 Observer의 구현체
- Subject 인터페이스와 Subject의 구현체
    - Observer리스트
    - notifyObservers를 통한 연결
- 람다를 통해 Observer를 직접 구현할  수도 있다.
    
    ```java
    feedLambda.registerObserver((String tweet) -> {
          if (tweet != null && tweet.contains("queen")) {
            System.out.println("Yet another news in London... " + tweet);
          }
        });
    ```
    

## **책임 연쇄 패턴 Chain of Responsibility(의무 체인)**

작업 처리 객체의 체인을 만들 때는 책임 연쇄 패턴을 사용한다. 

![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/d39aad34-20f2-404e-99bd-abc4055c614d)


의무적으로 메서드 동작을 처리한 후 다른 객체에 전달하는 형태로 동작한다. 예제 코드를 통해 확인해보자

```java
		ProcessingObject<String> p1 = new HeaderTextProcessing();
    ProcessingObject<String> p2 = new SpellCheckerProcessing();
    p1.setSuccessor(p2); -> set을 통해 p1의 동작 후 p2으로 보내는 동작 연결
    String result1 = p1.handle("Aren't labdas really sexy?!!");
    System.out.println(result1);
```

- 람다로 사용할 때는 이 패턴이 UnaryOperator를 활용한 함수 체인과 비슷하다. 때문에 해당 형태로 구현이 가능하다
    
    ```java
    UnaryOperator<String> headerProcessing = 
    	(String text) -> "From Raoul, Mario and Alan: " + text;
    
    UnaryOperator<String> spellCheckerProcessing = 
    	(String text) -> text.replaceAll("labda", "lambda");
    
    Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);
    String result2 = pipeline.apply("Aren't labdas really sexy?!!");
    ```
    

## 팩토리

인스턴스 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다.

```java
public static void main(String[] args) {
    Product p1 = ProductFactory.createProduct("loan"); //생성이 간결하고 캡슐화를 지킴
}

static private class ProductFactory {

    public static Product createProduct(String name) {
      switch (name) {
        case "loan":
          return new Loan();
        case "stock":
          return new Stock();
        case "bond":
          return new Bond();
        default:
          throw new RuntimeException("No such product " + name);
      }
    }

    public static Product createProductLambda(String name) {
      Supplier<Product> p = map.get(name);
      if (p != null) {
        return p.get();
      }
      throw new RuntimeException("No such product " + name);
    }
  }

	public static Product createProductLambda(String name) {
      Supplier<Product> p = map.get(name);
      if (p != null) {
        return p.get();
      }
      throw new RuntimeException("No such product " + name);
    }
```

# 람다 테스팅

## 보이는 람다 표현식의 동작 테스팅

람다는 익명함수이기 때문에 테스트 코드 이름을 호출할 수 없다. 따라서 필요하다면 람다를 필드에 저장해서 재사용, 테스트할 수 있다.

```java
public final static Comparator<Point> compareByXAndThenY = 
	comparing(Point::getX).thenComparing(Point::getY);
```

## 람다를 사용하는 메서드의 동작에 집중

람다를 사용하는 의도는 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것이다. 때문에 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 한다.

- 공개하지 않기 위해 기존의 리스트를 assertEquals를 사용하여 테스팅

## 복잡한 람다를 개별 메서드로 분할

람다가 너무 복잡하게 이루어져 있다면, 해당 부분을 메서드 참조로 변경하는 것이 좋다

## 고차원 함수 테스팅

함수를 인수로 받거나 다른 함수를 반환하는 메서드의 테스팅

```java
@Test
public void testFilter() throws Exception{
	List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
	List<Integer> even = filter(numbers, i -> i % 2 == 0);
	List<Integer> smallerThanThree = filter(numbers, i -> i < 3);
	assertEquals(Arrays.asList(2, 4), even);
	assertEquals(Arrays.asList(1, 2), smallerThanThree);
}
```

# 디버깅

디버깅을 할 때 가장 먼저 확인해야할 `스택 트레이스`, `로깅` 에 대해 알아보기

## 스택 트레이스 확인

예외 발생으로 프로그램 실행이 중단되면 **스택 프레임**에서 `호출 위치`, `호출할 때의 인수값`, `호출된 메서드의 지역변수` 등을 포함한 정보를 얻을 수 있다.

### 람다와 스택 트레이스

람다 표현식은 이름이 없기 때문에 스택 트레이스가 복잡하게 생성된다.

```java
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "modernjavainaction.chap09.Debugging$Point.getX()" because "p" is null
	at modernjavainaction.chap09.Debugging.lambda$main$0(Debugging.java:10)
	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:197)
	...
	at modernjavainaction.chap09.Debugging.main(Debugging.java:10)
```

- Debugging.lambda$main$0 : $0의 의미는 람다 표현식 내부에서 에러가 발생한 것을 뜻한다.
- 아직은 컴파일러가 람다 표현식과 관련한 스택 트레이스를 정확히 표현해주지 못한다.

## 정보 로깅

- peek() 스트림 연산을 통해 각 요소를 소비한 것처럼 동작을 실행한다. 하지만 forEach처럼 실제 스트림의 요소를 소비하지 않는다.

# 마치며

- 람다 표현식으로 가독성이 좋고 유연한 코드를 만들 수 있다.
- 익명 클래스는 람다 표현식으로 바꾸는 것이 좋다. 이때, this, 변수 섀도 등 익명 클래스와 다른점을 확실히 구분짓고 사용해야 한다.
- 메서드 참조로 람다 표현식보다 가독성이 좋은 코드로 구현할 수 있다.
- 람다 표현식으로 전략, 템플릿 메서드, 옵저버, 책임 연쇄 패턴, 팩토리 등의 객체지향 디자인 패턴을 활용할 수 있다.
- 람다 표현식도 단위 테스트를 할 수 있다.
- 복잡한 람다 표현식은 일반 메서드로 재구현할 수 있다.
- 람다 표현식은 디버깅할 때 스택 트레이스에 이상하게 담긴다
- 스트림 파이프라인의 요소 체크용도로 peek를 사용할 수 있다.
