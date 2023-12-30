## 리팩터링을 하는 목적은 무엇일까?
---
프로그래밍에서 코드를 리팩터링 하는 이유는 여러가지가 존재한다. 
프로그램의 구조를 변경하거나 개선하여 코드의 **가독성**, **유지보수성**, **성능**, **확정성** 등을 향상시키는게 가장 큰 이유라고 볼 수 있다. 물론 이 외에도 여러가지 이유로 리팩터링이 이루어진다.

> 마틴 파울러의 리팩토링
리팩터링이란, 소프트웨어의 겉보기 동작은 그대로 유지한 채, 코드를 이해하고 수정하기 쉽도록 내부 구조를 변경하는 기법

단순히 읽기 어려운 코드를 깔끔하게 정리하는 것만이 리팩터링이 아니며, 코드의 내부 구동을 유지한 채로 코드의 성능, 가독성, 확장성을 올리는 과정이라고 생각하면 될 것 같다.

- 람다 표현식으로 리팩터링하기

익명 클래스는 코드를 장황하게 만들고 쉽게 에러를 일이킬 수 있는 가능성을 가지고 있다. 이 부분을 람다 표현식을 이용해서 간결하고, 가독성이 좋은 코드로 구현할 수 있다.

_익명 클래스_

```java
        Runnable runnable = new Runnable() {
            public void run() {
                System.out.println("Hello");
            }
        };
```

_람다 표현식_

```java
		Runnable runnable = () -> System.out.println("Hello");
```

익명 클래스를 람다 표현식으로 상당히 깔끔하게 바꿀 수 있었다. 하지만 모든 익명 클래스를 람다 표현식으로 바꿀 수 있는 것은 아니니 이점은 꼭 명심하길 바란다.

> 모든 익명 클래스를 람다 표현식으로 바꿀 수 없는 이유
1, 익명 클래스에서 this는 익명클래스 자신을 가리킨다. 하지만 람다에서 this는 람다를 감싸는 클래스를 가리킨다.
2, 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다. 하지만 람다 표현식으로는 변수를 가릴 수 없다.
3, 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.


- 람다 표현식을 메서드 참조로 리팩터링하기

람다 표현식도 비교적 짧게 코드로 구성되어 있지만 메서드 참조를 이용하면 가독성을 더 높일 수 있다. 메서드 참조는 메서드명으로 의도를 명확하게 알릴 수 있기 때문이다.

_람다 표현식_

```java
    return menu.stream().collect(
        groupingBy(dish1 -> dish1.getType(),
            filtering(dish -> dish.getCalories() > 500, toList())));
  }
```

_메서드 참조_

```java
  private static Map<Dish.Type, List<Dish>> groupCaloricDishesByType() {
    return menu.stream().collect(
        groupingBy(Dish::getType,
            filtering(Grouping::isHighCalorieDish, toList())));
  }
  }
```

메서드 참조를 이용해서 getType과 isHighCalorieDish을 통해 타입정보와 칼로리가 높은 음식을 가져올려는 것을 쉽게 인지할 수 있다.

- 병렬처리를 위해 스트림으로 리팩터링하기

명령형 코드는 두 가지 패턴으로 엉킨 코드를 말한다. 이런 코드를 접하면 이용자는 전체 구현을 자세히 살펴본 이후에 전체 코드의 의도를 이해할 수 있다. 또한 반복자를 이용한 코드는 병렬로 실행시키기 매우 어렵다

_반복자_

```java
List<String> dishNames = new ArrayList<>();
for(Dish dish: menu){
	if(dish.getCalories() > 300) {
    	dishNames.add(dish.getName());
    }
}
```

위의 코드를 스트림을 통해 반복하면 더 알아보기 쉽게 구현가능하다.

_스트림_

```java
menu.parallelStream()
	.filter(d -> d.getCalories() > 300)
    .map(Dish::getName)
    .collect(toList());
```
코드를 병렬적으로 작성하기도 쉽고 한눈에 무엇을 해야하는 코드인지 이해하기도 쉽다.

### 코드 유연성 개선

람다 표현식을 이용하면 동작 파라미터화를 쉽게 구현할 수 있다. 즉, 다양한 람다를 전달해서 다양한 동작을 표현할 수 있다는 것이다.

- 조건부 연기 실행

```java
if (logger.isLoggable(Log.FINER)){
	logger.finer("Problem : " + generateDiagnostic());
}
```

위는 반복문으로 되어있다. Log가 FINER일 경우 if문의 내부 로직을 실행하는 로직이다. 위의 코드에는 문제사항이 있다.

1, logger의 상태가 isLoggable이라는 메서드에 의해 클라이언트 코드로 노출한다.
2, 메시지를 로깅할 때마다 logger 객체의 상태를 매번 확인해야 한다.

```java
logger.log(Level.FINER, "Problem: " + generateDiagnostic());
```

if문을 제거하여 log level을 확인하여 호출할 수 있다. 하지만 모든 문제가 해결되지는 않는다.

```java

logger.log(Level.FINER. () -> "Problem: " + generateDiagnostic());

public void log(Level level, Supplier<String> msgSupplier){
    if(logger.isLoggable(level)){
        log(level, msgSupplier.get()); // 람다 실행
    }
}
```

log 메서드는 logger의 수준이 적절하게 설정되어 있을 때만 인수로 넘겨진 람다를 내부적으로 실행하게 된다. 이로써 객체 상태를 자주 확인하거나, 객체의 일부를 호출하는 상황이라면 내부적으록 객체의 상태를 확인한 후 메서드를 호출한다.

## 람다로 객체지향 디자인 패턴 리팩터링하기

### 전략 패턴

한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법이다. 다양한 기준을 갖는 입력값을 검증하거나, 다양한 파싱 방법을 사용하거나, 입력 형식을 설정하는 등 다양한 시나리오에 전력 패턴이 사용 가능하다.

![](https://velog.velcdn.com/images/bw1611/post/faac3f46-d015-4b9e-8176-5390b6aabd47/image.png)

```java
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

  static private class Validator {

    private final ValidationStrategy strategy;

    public Validator(ValidationStrategy v) {
      strategy = v;
    }

    public boolean validate(String s) {
      return strategy.execute(s);
    }
}
```
전략 패턴을 구현한 것이다.

1, ValidationStrategy는 execute 메서드를 선언한 전략 인터페이스
2, IsAllLowerCase 및 IsNumeric은 ValidationStrategy 인터페이스를 구현한 구체적인 전략 클래스
3, Validator는 전략(전략 인터페이스)을 사용하여 유효성 검사를 수행하는 컨텍스트 클래스

```java
  public static void main(String[] args) {
    Validator v3 = new Validator((String s) -> s.matches("\\d+"));
    System.out.println(v3.validate("aaaa"));
    Validator v4 = new Validator((String s) -> s.matches("[a-z]+"));
    System.out.println(v4.validate("bbbb"));
  }
```

람다 표현식을 이용한다면 인터페이스를 구현한 구체 클래스를 없애고 생성자를 통해 생성과 동시에 람다 표현식을 통해 코드 조각을 캡슐화할 수 있어 전략 패턴을 대신할 수 있다.
즉, 위의 2번째의 구체적인 클래스를 생략할 수 있는 것이다.

### 탬플릿 메서드

알고리즘의 개요를 제시한 다음에 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 탬플릿 메서드 디자인 패턴을 사용한다.

```java
public abstract class 라면 {
    public void cook(){
        prepareWater();
        inputPowder();
        inputNoodle();
    }
 
    protected void prepareWater(){
        System.out.println("물을 넣는다.");
    }
 
    protected abstract void inputPowder();
 
    protected void inputNoodle(){
        System.out.println("면을 넣는다.");
    }
}
```

```java
public class 신라면 extends 라면 {
    @Override
    protected void inputPowder() {
        System.out.println("신라면 수프를 넣는다.");
    }
}

public class 짜파게티 extends 라면 {
    @Override
    protected void inputPowder() {
        System.out.println("자장 수프를 넣는다.");
    }
}

public class Main {
    public static void main(String[] args) {
        신라면 ramen1 = new 신라면();
        ramen1.cook();
 
        System.out.println();
 
        짜파게티 ramen2 = new 짜파게티();
        ramen2.cook();
    }
}
```

탬플릿 메서드의 장점

1, 중복 코드를 줄일 수 있다.
2, 핵심 로직의 관리가 용이하다.
3, 코드를 객체지향적으로 구성할 수 있다.

위의 코드는 라면의 종류가 늘어날 수록 클래스를 추가 해주어야 하는 수고를 해야한다. 이걸 람다 표현식을 이용하면 그 수고를 덜 수 있다.

```java
    public static void main(String[] args) {
        라면 ramen1 = new 라면();
        ramen1.cook(()-> System.out.println("신라면 수프를 넣는다."));
 
        System.out.println();
 
        라면 ramen2 = new 라면();
        ramen2.cook(()-> System.out.println("자장 수프를 넣는다."));
```

람다 표현식을 이용하면 템플릿 메서드 디자인 패턴에서 발생하는 자잘한 코드를 제거할 수 있다.

### 옵저버 패턴

어떤 이벤트가 발생했을 때 한 객체가 다른 객체 리스트에 자동으로 알림을 보내야 하는 상황에서 옵저버 디자인 패턴을 사용한다.

![](https://velog.velcdn.com/images/bw1611/post/c52f32e8-38e1-413e-8226-06f51408b7f8/image.png)

```java
public class ObserverMain {

  public static void main(String[] args) {
    Feed f = new Feed();
    f.registerObserver(new NYTimes());
    f.registerObserver(new Guardian());
    f.registerObserver(new LeMonde());
    f.notifyObservers("The queen said her favourite book is Java 8 & 9 in Action!");

  }

  interface Observer {
    void inform(String tweet);
  }

  interface Subject {
    void registerObserver(Observer o);
    void notifyObservers(String tweet);
  }

  static private class NYTimes implements Observer {

    @Override
    public void inform(String tweet) {
      if (tweet != null && tweet.contains("money")) {
        System.out.println("Breaking news in NY!" + tweet);
      }
    }

  }

  static private class Guardian implements Observer {

    @Override
    public void inform(String tweet) {
      if (tweet != null && tweet.contains("queen")) {
        System.out.println("Yet another news in London... " + tweet);
      }
    }

  }

  static private class LeMonde implements Observer {

    @Override
    public void inform(String tweet) {
      if (tweet != null && tweet.contains("wine")) {
        System.out.println("Today cheese, wine and news! " + tweet);
      }
    }

  }

  static private class Feed implements Subject {

    private final List<Observer> observers = new ArrayList<>();

    @Override
    public void registerObserver(Observer o) {
      observers.add(o);
    }

    @Override
    public void notifyObservers(String tweet) {
      observers.forEach(o -> o.inform(tweet));
    }
  }

}

```

옵저버 패턴을 통해 다들 옵저버를 구현하여 코드를 작성하였다. 
우선 `observers.add(o)` 에 클래스 들을 넣어준다.
`observers.forEach(o -> o.inform(tweet))`를 통하여 String에 money가 포함되어 있으면 NYTimes가 실행되게 되고 queen이 포함되어 있으면 Guardian이 실행되는 형태이다.

만약 단어를 추가할려면 또 Subject를 상속받아 구현해야할 것이다. 이렇게 반복되는 작업과 구체 클래스를 계속 만들어야 한다는 단점이 존재한다. 이런 단점을 보완하기 위하여 람다 표현식을 사용할 수 있다.

```java
    feedLambda.registerObserver((String tweet) -> {
      if (tweet != null && tweet.contains("money")) {
        System.out.println("Breaking news in NY! " + tweet);
      }
    });
    feedLambda.registerObserver((String tweet) -> {
      if (tweet != null && tweet.contains("queen")) {
        System.out.println("Yet another news in London... " + tweet);
      }
    });
```

람다 표현식을 이용하면 구체 클래스를 만들지 않고도 쉽게 위의 코드를 사용할 수 있다. 즉, 명시적으로 인스턴스화하지 않고 람다 표현식으로 직접 전달하여 실행할 동작을 지정한 것이다.
하지만 람다 표현식을 무조건 사용해야 하는 것은 아니다. 상황에 맞는 표현식을 사용하는 것이 가장 바람직하다.

### 의무 체인

한 객체가 어떤 작업을 처리한 후 다른 객체로 결과를 전달하고, 다른 객체도 해야할 작업을 처리한 후 다른 객체로 전달하는 것을 의무 체인이라고 한다.

![](https://velog.velcdn.com/images/bw1611/post/c13fb2ff-89ae-4c01-9e5e-7fb25a52d857/image.png)

의무 체인 패턴또한 메서드를 생성할 필요 없이 쉽게 람다 표현식을 사용하여 생략할 수 있는 부분들을 만들 수 있다.

```java
public class ChainOfResponsibilityMain {

  public static void main(String[] args) {
    ProcessingObject<String> p1 = new HeaderTextProcessing();
    ProcessingObject<String> p2 = new SpellCheckerProcessing();
    p1.setSuccessor(p2);
    String result1 = p1.handle("Aren't labdas really sexy?!!");
    System.out.println(result1);
    
  private static abstract class ProcessingObject<T> {

    protected ProcessingObject<T> successor;

    public void setSuccessor(ProcessingObject<T> successor) {
      this.successor = successor;
    }

    public T handle(T input) {
      T r = handleWork(input);
      if (successor != null) {
        return successor.handle(r);
      }
      return r;
    }

    abstract protected T handleWork(T input);

  }

  private static class HeaderTextProcessing extends ProcessingObject<String> {

    @Override
    public String handleWork(String text) {
      return "From Raoul, Mario and Alan: " + text;
    }

  }

  private static class SpellCheckerProcessing extends ProcessingObject<String> {

    @Override
    public String handleWork(String text) {
      return text.replaceAll("labda", "lambda");
    }
  }
}
```

```java

public class ChainOfResponsibilityMain {

  public static void main(String[] args) {
    UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan: " + text;
    UnaryOperator<String> spellCheckerProcessing = (String text) -> text.replaceAll("labda", "lambda");
    Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);
    String result2 = pipeline.apply("Aren't labdas really sexy?!!");
    System.out.println(result2);
  }
}
```

위와 같이 람다를 이용하면 코드를 쉽게 좀 더 가독성이 있게 수정할 수 있으며 로직을 쉽게 알아볼 수 있다.

## 람다 테스팅

- 테스트 클래스 생성

```java
public class Exam {
    private final int x;
    private final int y;

    public Exam(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }
    
    public Point moveRightBy(int x){
        return new Point(this.x + x, this.y);
    }
    
    public final static Comparator<Point> compareByXThenY 
            = Comparator.comparing(Point::getX).thenComparing(Point::getY);
}
```

1, 람다 표현식을 이용한 동작 테스팅

람다를 사용해서 할 단위 테스트를 진행할 경우 익명이므로 테스트 코드 이름을 호출할 수 없다. 따라서 람다를 필드에 저장해서 재사용할 수 있으며 람다의 로직을 테스트할 수 있다.
Comparator 객체 compareByXThenY에 다양한 인수로 compare 메서드를 호출하면서 예상대로 동작하는지 테스트할 수 있다.


```java
@Test
public void testComparingTwoPoints() throws Exception {
	Point p1 = new Point(10, 15);
    Point p2 = new Point(10, 20);
    int result = Point.compareByXAndThenY.compare(p1, p2);
    assertTrue(result < 0);
}
```

2, 람다를 사용하는 메서드의 동작에 집중

람다의 의도에 따라 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화했다. 이렇게 하기 위해서는 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 한다.(람다를 표현하지 않으면서도 람다 표현식을 검증하기 위하여)

```java
    public static List<Exam> moveAllPointRightBy(List<Exam> exams, int x){
        return exams.stream()
                .map(p -> new Exam(p.getX() + x, p.getY()))
                .collect(Collectors.toList());
    }
```

```java
    @Test
    public void testMoveAllPointRightBy() throws Exception{
        List<Exam> exam = Arrays.asList(new Exam(5, 5), new Exam(10, 5));
        List<Exam> expected = Arrays.asList(new Exam(15, 5), new Exam(20, 5));
        List<Exam> exams = Exam.moveAllPointRightBy(exam, 10);
        assertEquals(expected, exams);
    }
```

## 디버깅

람다를 테스트하면서 코드에 문제를 발견할 수 있는데, 이럴경우 코드의 디버깅이 필요하다. 코드를 디버깅할 때 두가지를 체크해야한다.

- 스택 트레이스
- 로깅

하지만 기존의 디버깅 기법은 람다에서는 사용하지 못한다. 어떻게 해결해야 할까?

### 스택 트레이스 확인

어디가 문제인지 찾기 위해서는 스택 프레임에서 정보를 찾을 수 있다. 메서드를 호출할 때마다 프로그램에서의 호출 위치, 인수 값, 호출된 메서드의 지역 변수 등을 포함한 호출 정보가 생성되며 이들 정보는 스택 프레임에 저장된다. 결과로 프레임별로 보여주는 스택 트레이스를 얻을 수 있다.

하지만 람다는 이름이 없기 때문에 조금 복잡한 스택 트레이스가 만들어진다. 

```java
    public static void main(String[] args) {
        List<Exam> exams = Arrays.asList(new Exam(12, 2), null);
        exams.stream()
                .map(p -> p.getX()).forEach(System.out::println);
    }
```

오류 코드

```
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "modernjavainaction.chap09.Exam.getX()" because "p" is null
	at modernjavainaction.chap09.Exam2.lambda$main$0(Exam2.java:10)
	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:197)
	at java.base/java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:992)
	at java.base/java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:509)
	at java.base/java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:499)
	at java.base/java.util.stream.ForEachOps$ForEachOp.evaluateSequential(ForEachOps.java:150)
	at java.base/java.util.stream.ForEachOps$ForEachOp$OfRef.evaluateSequential(ForEachOps.java:173)
	at java.base/java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234)
	at java.base/java.util.stream.ReferencePipeline.forEach(ReferencePipeline.java:596)
	at modernjavainaction.chap09.Exam2.main(Exam2.java:10)
```

`chap09.Exam2.lambda$main$0(Exam2.java:10)` 이와 같이 이상한 문자는 람다 표현식의 내부에서 에러가 발생했음을 가리킨다.

어느 메서드에서 에러가 발생했는지 확인하기 위해서는 메서드 참조를 하면된다.

```java
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3);

        numbers.stream()
                .map(Exam2::divide).forEach(System.out::println);
    }

    private static int divide(int n) {
        return n / 0;
    }
```

```
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at modernjavainaction.chap09.Exam2.divide(Exam2.java:15)
```

divide에서 문제가 나왔다는 걸 확인할 수 있다. 하지만 여전히 람다를 디버깅하는 것은 쉽지 않고 어려운 문제로 남아있긴하다.

### 정보 로깅

peek이라는 스트림 연산을 통해 정보 로깅을 할 수 있다. peek은 스트림의 각 요소를 소비한 것처럼 동작을 실행한다. (실제로는 스트림의 요소를 소비하지 않는다.)

```java
        List<Integer> numbers = Arrays.asList(1, 2, 3);

        List<Integer> collect = numbers.stream()
                .peek(x -> System.out.println("from stream : " + x))
                .map(x -> x + 17)
                .peek(x -> System.out.println("after map " + x))
                .collect(Collectors.toList());
```

```
from stream : 1
after map 18
from stream : 2
after map 19
from stream : 3
after map 20
complete : [18, 19, 20]
```

실행 결과를 보면 파이프라인의 단계별 상태를 확인할 수 있다.
