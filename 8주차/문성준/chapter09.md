# 9. 리팩터링, 테스팅, 디버깅

<aside>
📖 이 장의 내용
- 람다 표현식으로 코드 리팩터링하기
- 람다 표현식이 객체지향 설계 패턴에 미치는 영향
- 람다 표현식 테스팅
- 람다 표현식과 스트림 API 사용 코드 디버깅

</aside>

---

## **9.1 가독성과 유연성을 개선하는 리팩터링**

람다 표현식은 익명 클래스보다 코드를 좀 더 간결하게 만든다.
람다, 메서드 참조, 스트림 등의 기능을 이용해서 더 가독성이 좋고 유연한 코드로 리팩터링하는 방법을 살펴보자.

### **9.1.1 코드 가독성 개선**

가독성을 개선한다는 것은 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 만드는 것을 의미한다.
코드 가독성을 높이려면 코드의 문서화를 잘 하고, 표준 코딩 규칙을 준수해야한다.

### **9.1.2 익명 클래스를 람다 표현식으로 리팩터링하기**

왜 익명 클래스를 람다 표현식으로 리팩터링 하는 것일까 ? → 익명클래스는 코드를 장황하게 만들고 쉽게 에러를 일으키기 때문이다. 즉, 익명클래스를 람다 표현식으로 변환하면 간결하고 가독성 좋은 코드를 구현할 수 있다.

하지만 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다. 아래 주의점을 주의 깊게 보자.

**익명클래스를 람다표현식으로 변환할 때 주의점**

1. 익명클래스에서 this는 익명클래스 자신을 가리키지만 람다에서 this는 람다를 감싸는 클래스를 가리킨다.
(즉, 익명 클래스에서 사용한 this와 super는 람다 표현식에서 다른 의미를 갖는다)

2. 익명클래스는 감싸고있는 클래스의 변수를 가릴 수 있지만, 람다 표현식으로는 가릴 수 없다.

3. 익명클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.
→ 익명 클래스는 인스턴스화할 때 명시적으로 형식이 정해지는 반면 람다의 형식은 콘텍스트에 따라 달라지기 때문이다. 이 말도 어려울 수 있다. 
다시 말해, 람다 표현식이 어떤 함수형 인터페이스를 구현하는지에 따라 다른 결과를 가져올 수 있기 때문에 '콘텍스트 오버로딩'에 따른 모호함이 발생할 수 있다는 것이다.

```jsx
interface Task {
public void execute();
}

public static void doSomething(Runnable r) { r.run(); }
public static void doSomething(Task a) { r.excute(); }

//익명클래스로 전달 가능
doSomethig(new Task() {
public void execute() {
System.out.println("Danger danger!!");
}
});

//Runnable과 Task 모두 대상 형식이 가능하므로 모호함 발생
doSomething(() -> System.out.println("Danger danger!!"));

//명시적 형변환을 사용해서 모호함을 제거할 수 있음
doSomething((Task)() -> System.out.println("Danger danger!!"));
```

### **9.1.3 람다 표현식을 메서드 참조로 리팩터링하기**

메서드 참조를 사용하면 메서드명으로 코드의 의도를 명확히 알릴 수 있다.

또한 `comparing`과 `maxBy`같은 정적 헬퍼 메서드를 활용하는 것도 좋다.

```jsx
inventory.sort(
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())); //비교구현에 신경써야함
inventory.sort(comparing(Apple::getWeight)); //코드가 문제 자체를 설명
```

람다표현식과 저수준 리듀싱 연산을 조합하는것보다 Collectors API를 사용하면 코드의 의도가 더 명확해진다.

```jsx
//람다 + 저수준 리듀싱 사용
int totalaCalories = menu.stream().map(Dish::getCalories)
.reduce(0, (c1, c2) -> c1 + c2);

//내장 컬렉터 사용
int totalaCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

### **9.1.4 명령형 데이터 처리를 스트림으로 리팩터링하기**

스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여준다. 스트림은 쇼트서킷과 게으름이라는 강력한 최적화뿐 아니라 멀티코어 아키텍처를 활용할 수 있는 지름길을 제공

```jsx
List<String> dishNames = new ArrayList<>();
for(Dish dish : menu) {
if(dish.getCalories() > 300 ) {
dishNames.add(dish.getName());
}
}

//스트림 API로 변환하면 더 직접적으로 기술 & 쉽게 병렬화 가능
menu.parallelStream()
.filter(d -> d.getCaloires() > 300)
.map(Dish::getName)
.collect(toList());
```

### **9.1.5 코드 유연성 개선**

다양한 람다를 전달해서 다양한 동작을 표현했다. 프레디케이트로 다양한 필터링 기능을 구현하거나 비교자로 다양한 비교 기능을 만들 수 있다 → 즉, 변화하는 요구사항에 대응할 수 있는 코드를 구현할 수 있다.

**함수형 인터페이스 적용**

람다표현식을 이용하려면 함수형 인터페이스를 코드에 추가해야한다.

**조건부 연기 실행**

```jsx
if(logger.isLoggale(Log.FINER)) 
logger.finner("Problem : " + generateDiagnostic());
}
```

위 코드의 문제점은 `logger`의 상태가 `isLoggable` 메서드에 의해 클라이언트 코드로 노출된다. 
또한 메시지를 로깅할때마다 `logger`의 객체를 매번 확인해야 한다. → 코드를 어지럽힐 뿐이다.

```jsx
// 이렇게 메시지를 로깅하기 전에 logger 객체가 적절한 수준으로 설정되었는지 내부적으로 확인하는
// log 메서드를 사용하는 것이 바람직하다.
logger.log(Lelvel.FINER, "Problem : " + generateDiagnostic());
```

덕분에 불필요한 if문을 제거하고 logger의 상태를 노출할 필요도 없어졌지만, logger가 활성화되어 있지 않더라도 항상 로깅 메시지를 평가하게 된다.

다음은 supplier를 인수로 갖는 오버로드된 log 메서드를 통해 해결할 수 있다.

```jsx
logger.log(Level.FINER, () ->  "Problem : " + generateDiagnostic());
```

log 메서드는 logger의 수준이 적절하게 설정되어 있을 때만 인수로 넘겨진 람다를 내부적으로 실행하다.
다음은 log 메서드의 내부구현 코드이다.

```jsx
public void log(Level level, Supplier<String> msgSupplier) {
if(logger.isLoggable(level)) {
log(level, msgSupplier.get()); //람다 실행
}
}
```

이 기법으로 어떤 문제를 해결할 수 있을까? 
만일 클라이언트 코드에서 객체 상태를 자주 확인하거나 (예를 들면 logger의 상태),
객체의 일부 메서드를 호출하는 상황(예를 들면 메시지 로깅)이라면 내부적으로 객체의 상태를 확인한 다음에 메서드를 호출(람다나 메서드 참조를 인수로 사용)하도록 새로운 메서드를 구현하는 것이 좋다.
그러면 코드 가독성이 좋아질 뿐아니라 캡슐화도 강화된다 → 객체 상태가 클라이언트 코드로 노출되지 않기때문

### **실행 어라운드**

매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드가 있다면 람다로 변환할 수 있다.

```jsx
String oneLine = 
	processFile((BufferedReader b) -> b.readline()); //람다 전달
String twoLine = 
	processFile((BufferedReader b) -> b.readline() + b.readline()); //다른 람다 전달
public static String processFile(BufferedReaderProcessor p) throws IOException {
	try(BufferedReader br = new BufferedReader(new fileReader("data.txt"))) {
		return p.process(br); //인수로 전달된 BufferedReaderProcessor 실행
}
}// IOException을 던질 수 있는 람다의 함수형 인터페이스

public interface BufferedReaderProcessor {
	string process(BufferedReader b) throws IOException;
} // 이 인터페이스 덕분에 람다로 버퍼리더 객체의 동작을 결정할 수 있다.
```

## **9.2 람다로 객체지향 디자인 패턴 리팩터링하기**

람다를 이용하면 이전에 디자인 패턴으로 해결하던 문제를 더 쉽고 간단하게 해결할 수 있다.
람다 표현식으로 기존의 많은 객체지향 디자인 패턴을 제거하거나 간결하게 재구현할 수 있다.
앞으로 배울 다섯 가지 패턴은 각 디자인 패턴에서 람다를 어떻게 활용할 수 있는지 알 수 있다.

### **9.2.1 전략**

전략패턴은 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법이며, 세 부분으로 구성된다.

![IMG_0877.jpeg](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/03cf9afc-197b-4a6b-ac82-86c5362e9b9c/IMG_0877.jpeg)

```jsx
//String 문자열을 검증하는 인터페이스
public interface ValidationStrategy {
boolean execute(String s);
}

//위에서 정의한 인터페이스를 구현하는 클래스를 하나 이상 정의
public class IsAllLowerCase implements ValidationStrategy {
public boolean execute(String s) {
return s.matches("[a-z]+");
}
}

public class IsNumeric implements ValidationStrategy {
public boolean execute(String s) {
return s.matches("\\d+");
}
}

//전략객체를 사용하는 클래스이며, 지금까지 구현한 클래스를 다양한 검증 전략으로 활용할 수 있따.
public class Validator {
private final ValidationStrategy strategy;
public Validator(ValidationStrategy v) {
this.strategy = v;
}
public boolean validate(String s) {
return strategy.execute(s);
	}
}
Validator numericValidator = new Validator(new IsNumeric());
boolean b1 = numericValidator.validate("AAA"); //false
Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
boolean b2 = lowerCaseValidator.validate("bbb"); //true
```

**람다 표현식 사용**

다양한 전략을 구현하는 새로운 클래스를 구현할 필요 없이 람다 표현식을 직접 전달하면 코드가 간결해진다.
ValidationStrategy는 함수형 인터페이스이며 Predicate<String>과 같은 함수 디스크립터를 갖고 있기 때문.

```jsx
Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+")); //람다를 직접 전달
boolean b1 = numericValidator.validate("AAA");
Validator lowerCaseValidator = new Validator((String s) -> s.matches("\\d+");
boolean b2 = lowerCaseValidator.validate("bbb");
```

### **9.2.2 템플릿 메서드**

알고리즘의 개요를 제시한 다음에 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 템플릿 메서드 디자인 패턴을 사용한다.
→ 너무 추상적이다. 이는, 템플릿 메서드는 ‘이 알고리즘을 사용하고 싶은데 그대로는 안 되고 조금 고쳐야 하는’ 상황에 적합하다는 것이다.

```jsx
abstract class OnlineBanking {
	public void processCustomer(int id) {
		Customer c = Database.getCustomerWithId(id);
		makeCustomerHappy(c);
	}
	abstract void makeCustomerHappy(Customer c);
}
```

processCustomer 메서드는 온라인 뱅킹 알고리즘이 해야 할 일을 보여준다.
각각의 지점은 OnlineBanking 클래스를 상속받아 makeCustomerHappy 메서드가 원하는 동작을 수행하도록 구현할 수 있다.

**람다 표현식 사용**

```java
//Consumer<Customer> 형식의 두번째 인수를 추가
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
	Customer c = Database.getCustomerWithId(id);
	makeCustomerHappy.accept(c);

}

//람다표현식을 직접 전달 가능
new OnlineBankingLambda().processCustomer(1337, (Customer c) -> 
	System.out.println("Hello" + c.getName());
```

### **9.2.3 옵저버**

어떤 이벤트가 발생했을 때 한 객체(주제)가 다른 객체 리스트(옵저버)에 자동으로 알림을 보내야하는 상황에서 옵저버 디자인 패턴을 사용한다.
실제 옵저버 패턴으로는 트위터 같은 커스터마이즈된 알림 시스템을 설계하고 구현할 수 있다.
다양한 신문 매체(뉴욕타임스, 가디언, 르몽드)가 뉴스 트윗을 구독하고 있으며 특정 키워드를 포함하는 트윗이 등록되면 알림을 받고 싶어한다. 
우선 다양한 옵저버를 그룹화할 Observer 인터페이스가 필요하다. Observer 인터페이스는 새로운 트윗이 있을 때 주제(Feed)가 호출할 수 있도록 notify라고 하는 하나의 메서드를 제공한다.

```java
//1. 옵저버
//주제가 호출할 수 있도록 notify 메서드 제공
interface Observer {
	void notify(String tweet);
}
Class NYTimes implements Observer {
	public void notify(String tweet) {
		if(tweet != null && tweet.contains("money")) {
			System.out.println("Breaking news in NY!" + tweet);
		}
	}
}
Class Guardian implements Observer {
	public void notify(String tweet) {
		if(tweet != null && tweet.contains("queen")) {
			System.out.println("Yet more news from London..." + tweet);
		}
	}
}

//2. 주제
interface Subject {
	void registerObserver(Observer o);
	void notifyObservers(String tweet);
}
Class Feed implements Subject {
	private final List<Observer> observers = new ArrayList<>();
	public void registerObserver(Observer o) {
		this.observers.add(0);
}
	public void notifyServeres(String tweet) {
		observers.forEach(o -> o.notify(tweet));
	}
} 
/*
 Feed는 트윗을 받았을 때 알림을 보낼 옵저버 리스트를 유지한다. 이제 주제와 옵저버를 연결하는 데모
	app을 만들 수 있따.
*/

Feed f = new Feed();
f.registerObserver(new NYTimes());
f.registerObserver(new Guardian());
f.notifyServeres("The queen said her favorite book is Modern Java in Action!");
// 이제 가디언도 우리의 트윗을 받아볼 수 있게 되었다.
```

**람다 표현식 사용**
옵저버를 명시적으로 인스턴스화하지 않고 람다표현식을 직접 전달해서 실행할 동작을 지정할 수 있다.
(람다는 불필요한 감싸는 코드 제거 전문가이기 때문이다)

```java
f.registerObserver((String tweet) -> {
	if(tweet != null && tweet.contains("money")) {
		System.out.println("Breaking news in NY!" + tweet);
	}
});
f.registerObserver((String tweet) -> {
	if(tweet != null && tweet.contains("queen")) {
		System.out.println("Yet more news from London..." + tweet);
	}
});
```

람다 표현식은 항상 사용하는 것일까?
→ 실행해야 할 동작이 비교적 간단하면 람다표현식을 사용해 불필요한 코드를 제거하고,
옵저버가 상태를 가지며, 여러 메스드를 정의하는 등 복잡하다면 람다 표현식보다 기존의 클래스 구현방식이 낫다.

### **9.2.4 책임 연쇄 패턴 Chain of Responsibility**

작업 처리 객체의 체인(동작 체인 등)을 만들 떄는 의무 체인 패턴을 사용한다.

```java
public abstract class processingObject<T> {
	protected ProcessingObject<T> successor;
	public void setSuccessor(ProcessingObject<T> successor) {
		this.successor = successor;
}
public T handle(T input) {
	T r = handleWork(input);
	if(successor != null) {
		return successor.handle(r);
		}
	return r;
	}
abstract protected T handleWork(T input);
}
```

handle 메서드는 일부 작업을 어떻게 처리해야할지 전체적으로 기술한다.

ProcessingObject 클래스를 상속받아 handleWork 메서드를 구현하여 다양한 종류의 작업 처리 객체를 만들 수 있다.

```java
public class HeaderTextProcessing extends ProcessingObject<String> {
	public String handleWork(String text) {
		return "From Raoul, Mario and Alan : " + text;
	}
}
public class SpellCheckerProcessing extends ProcessingObject<String> {
	public String handleWork(String text) {
		return text.replaceAll("labda", "lambda");
	}
}
// 두 작업 처리 객체를 연결해서 작업 체인을 만들 수 있다.
ProcessingObject<String> p1 = new HeaderTextProcessing();
ProcessingObject<String> p2 = new SpellCheckerProcessing();
p1.setSuccessor(p2); //두 작업처리 객체를 연결
String result = p1.handle("Aren't labdas really sexy?!");
System.out.println(result); 
// From Raoul, Mario and Alan : Aren't labdas really sexy?! 출력
```

**람다 표현식 사용**

함수 체인(함수조합)과 비슷하다 ! 작업처리 객체를 `Function<String,String>`, 더 정확히 표현하자면`UnaryOperator<String>` 형식의 인스턴스를 표현할 수 있으며, `andThen` 메서드로 이들 함수를 조합해서 체인을 만들 수 있다.

```java
UnaryOperator<String> headerProcessing =
	(String text) -> "From Raoul, Mario and Alan : " + text; // 1번째 작업처리 객체
UnaryOperator<String> spellCheckerProcessing =
	(String text) -> text.replaceAll("labda", "lambda"); //2번째 작업처리 객체
Function<String, String> pipeline = 
	headerProcessing.andThen(spellCheckerProcessing); // 동작 체인으로 두 함수 조합
String result = pipeline.apply("Aren't labdas really sexy?!");
```

### **9.2.5 팩토리**

인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다.

```java
// 은행에서 취급하는 다양한 상품을 만드는 팩토리 클래스
public class ProdectFactor {
	public static Product createProduct(String name) {
		switch(name) {
			case "loan" : return new Loan();
			case "stock" : return new Stock();
			case "bond" : return new Bond();
			default : throw new RuntimeException("No Such product" + name);
		}
	}
}
```

여기서 Loan, Stock, Bond는 모두 Product의 서브 형식이다. 생성자와 설정을 외부로 노출하지 않음으로써 클라이언트가 단순하게 상품을 생산할 수 있다. `product p = ProductFactory.createProduct("loan");`
이는 부가적인 기능일 뿐 위 코드의 진짜 장점은 생산자와 설정을 외부로 노출하지 않음으로써 (캡슐화) 클라이언트가 단순하게 상품을 생산할 수 있다는 것이다.

**람다 표현식 사용**

```java
// Loan 생성자를 사용하는 코드
Supplier<Product> loanSupplier = Loan::new;
Loan loan = loanSupplier.get();
```

위 코드와 같이 생성자도 메서드 참조처럼 접근할 수 있다.

```java
//상품명과 생성자를 연결하는 Map 코드로 재구현
final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
	map.put("loan", Loan::new);
	map.put("stock", Stock::new);
	map.put("bond", Bond::new);
}

//Map을 이용해 다양한 상품을 인스턴스화
public static Product createProduct(String name) {
	Supplier<product> p = map.get(name);
	if(p != null) return p.get();
	throw new RuntimeException("No Such product" + name);
}
```

createProduct가 상품 생성자,로 여러 인수를 전달하는 상황에서는 이 기법을 적용하기 어렵다. 단순한 Supplier 함수형 인터페이스로는 이 문제를 해결할 수 없다.
예를 들어 세 인수를 받는 상품의 생성자가 있따고 가정한다면, 세 인수를 지원하려면 특별한 함수형 인터페이스를 만들어야한다. 결국 Map의 시그니처가 복잡해진다는 것이다.

```java
public interface TriFunction<T, U, V, R>{
	R apply(T t, U u, V v);
}
Map<String, TriFunction<Integer, Integer, String, Product>> map 
	= new HashMap<>();
```

## **9.3 람다 테스팅**

일반적으로 프로그램이 의도대로 동작하는지 확인할 수 있는 단위테스팅을 진행한다.
우리는 소스 코드의 일부가 예상된 결과를 도출할 것이라 단언하는 테스트 케이스를 구현한다.

### **9.3.1 보이는 람다 표현식의 동작 테스팅**

람다 표현식은 함수형 인터페이스의 인스턴스를 생성하므로, 생성된 인스턴스의 동작으로 람다 표현식을 테스트할 수 있다. 하지만 람다는 익명이므로 테스트 코드 이름을 호출할 수 없다.
따라서 필요하다면 람다를 필드에 저장해서 재사용할 수 있으며 람다의 로직을 테스트할 수 있다.

```java
// 메서드를 호출하는 것처럼 람다를 사용할 수 있다.
public class Point {
	public final static Comparator<Point> compareByXAndThenY =
		comparing(Point::getX).thenComparing(Point::getY);
	...
}

//TEST 코드
@Test
public void testComparingTwoPoints() throws Exception {
	Point p1 = new Point(10, 15);
	Point p2 = new Point(10, 20);
	int result = Point.compareByXThenY.compare(p1, p2);
	assertTrue(result < 0);
}
```

### **9.3.2 람다를 사용하는 메서드의 동작에 집중하라**

람다의 목표는 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것이다.
그러려면 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 한다.
람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.

```java
public static List<Point> moveAllPointsRightBy(List<Point> points, int x) {
	return points.stream()
		.map(p -> new Point(p.getX() + x, p.getY()))
		.collect(toList());
}

//TEST 코드
@Test
public void testComparingAllPointsRigtTwoPoints() throws Exception {
	List<Point> points =
		Arrays.asList(new Point(5, 5), new Point(10, 5));
	List<Point> excpectedPoints = 
		Arrays.asList(new Point(15, 5), new Point(20, 5));
	List<Point> newPoinsts = Point.moveAllPointsRightBy(points, 10);
	assertEquals(expectedPoint, newPoints);
}
/*
위 단위 테스트에서 보여주는 것처럼 Point 클래스의 equals 메서드는 중요한 메서드다.
따라서 Object의 기본적인 equals 구현을 그대로 사용하지 않으려면 equals 메서드를 적절하게 구현해야 한다.
*/
```

### **9.3.4 고차원 함수 테스팅**

함수를 인수로 받거나 다른 함수를 반환하는 메서드를 고차원 함수라고 한다.
메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트할 수 있다.

```java
@Test
Public void testFilter() throws Exception {
	List<Integer> numbers = Arrays.asList(1,2,3,4);
	List<Integer> even = filter(numbers, i -> i % 2 == 0);
	List<Integer> smallerThanThree = filter(numbers, i -> i < 3);
	assertEquals(Arrays.asList(2,4), even);
	assertEquals(Arrays.asList(1,2), smallerThanThree);
}
```

### **9.4 디버깅**

람다 표현식과 스트림은 기존의 디버깅 기법(스택트레이스, 로깅)을 무력화한다.

### **9.4.1 스택 트레이스 확인**

**람다와 스택 트레이스**

람다 표현식은 이름이 없기 때문에 복잡한 스택 트레이스가 생성된다.

메서드 참조를 사용해도 스택 트레이스에는 메서드 명이 나타나지 않는다.

메서드 참조를 사용하는 클래스와 같은 곳에 선언되어있는 메서드를 참조할 때는 메서드 참조 이름이 스택 트레이스에 나타난다.
또한 람다 표현식과 관련한 스택 트레이스는 이해하기 어려울 수 있따는 점을 염두에 두자.
이는 미래의 자바 컴파일러가 개선해야 할 부분이다.

### **9.4.2 정보 로깅**

```java
// forEach로 스트림 결과를 출력하거나 로깅할 수 있다.
List<Integer> numbers = Arrays.asList(2,3,4,5);
numbers.stream()
	.map(x -> x + 17)
	.filter(x -> x % 2 == 0)
	.limit(3)
	.forEach(System.out::println);
/*
출력
20
22

forEach를 호출하는 순간 전체 스트림이 소비된다. 
스트림 파이프라인에 적용된 각각의 연산이 어떤 결과를 도출하는지 확인할 수 이씅면 좋을 것 같다.
이는 peek 연산을 활용하면 된다. 아래 자세히 적어뒀따.
*/
```

위 연산에서 `forEach`를 호출하는 순간 전체 스트림이 소비된다.

파이프라인에 적용된 각각의 연산(map, filter, limt)이 어떤 결과를 도출하는지 확인하려면 peek이라는 스트림 연산을 활용할 수 있다.

peek은 스트림의 각 요소를 소비한 것처럼 동작하지만, forEach처럼 실제로 소비하지는 않는다.
