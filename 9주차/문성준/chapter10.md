# 10. 람다를 이용한 도메인 전용 언어

<aside>
📖 <목차>

</aside>

---

모던 자바 인 액션 10장에서는 도메인 전용 언어(domain specific languages, DSL) 에 대해 소개하고 있다.
DSL 이 무엇인지 알아보고 람다를 이용하여 효과적인 자바 기반 DSL 을 구현하는 방법에 대해 알아본다.

개발팀과 도메인 전문가가 이해할 수 있는 코드는 생산성과 직결되기 때문에 코드는 읽기 쉽고 이해하기 쉬워야 한다. 도메인 전용 언어(DSL)는 특정 도메인을 대상으로 만들어진 프로그래밍 언어로 이 문제를 해결할 수 있다. (ex. 빌드 과정을 표현하는 dsl 인 메이븐, 앤트 등)

---

# 10.1 도메인 전용 언어

DSL(domain specific languages)은 범용 프로그래밍 언어가 아니라 특정 비즈니스 도메인의 문제를 해결하려고 만든 언어다.

특정 도메인에만 국한되므로 오직 자신의 문제를 어떻게 해결할지에만 집중할 수 있고 복잡성을 잘 다룰 수 있다.

- DSL을 개발하기 위해 필요한 점
    - 의사 소통의 왕: 프로그래머가 아닌 사람도 이해할 수 있도록 코드의 의도가 명확히 전달되어야 함(코드가 비즈니스 요구사항에 부합하는지 확인 가능)
    - 한 번 코드를 구현하지만 여러번 읽는다: 가독성은 유지보수의 핵심. 동료가 쉽게 이해할 수 있도록 코드 구현 필수

### **장점**

- 간결함
    - 비즈니스 로직을 간편하게 캡슐화하므로 반복을 피하고 코드를 간결하게 만들 수 있음
- 가독성
    - 도메인 영역의 용어를 사용하므로 비 도메인 전문가도 쉽게 이해 가능
    - 조직 구성원 간에 코드와 도메인 영역이 공유될 수 있음
- 유지보수
    - 잘 설계된 DSL로 구현하면 쉽게 유지보수하고 변경 가능
- 높은 수준의 추상화
    - DSL 은 도메인과 같은 추상화 수준에서 동작하므로 도메인의 문제와 직접적으로 관련되지 않은 세부 사항을 숨김
- 집중
    - 도메인의 규칙을 표현할 목적으로 설계되었으므로 프로그래머가 특정 코드에 집중 가능 (생산성 향상)
- 관심사 분리
    - 애플리케이션의 인프라구조와 관련된 문제와 독립적으로 비즈니스 관련된 코드에서 집중 가능 (유지보수 향상)

### **단점**

- DLS 설계의 어려움
    - 간결하게 제한적인 언어에 도메인 지식을 담기 어려움
- 개발 비용
    - 코드에 DSL을 추가하는 작업은 초기 프로젝트에 많은 비용과 시간이 소모
    - DSL 유지보수와 변경은 프로젝트에 부담
- 추가 우회 계층
    - DSL은 추가적인 계층으로 도메인 모델을 감싸며 계층을 최대한 작게 만들어 성능 문제 회피
- 새로 배워야 하는 언어
    - DSL 을 프로젝트에 추가하면 배워야 하는 언어도 증가
- 호스팅 언어 한계
    - 자바 같은 범용 프로그래밍 언어는 장황하고 엄격한 문법을 가지므로 사용자 친화적 DSL 만들기 어려움 (람다 표현식으로 어느정도 해결)

---

### **내부 DSL**

순수 자바코드 같은 기존 호스팅 언어를 기반으로 구현한 DSL을 말한다.

유연성이 떨어지기 때문에 간단하고 표현력 있는 DSL을 만드는데 한계가 있었지만 람다 표현식이 등장하면서 어느정도 해결될 수 있게 되었다.

순수 자바로 DSL 을 구현하면 다음과 같은 장점이 존재

- 외부 DSL에 비해 새로운 패턴과 기술을 배워 DSL을 구현하는 노력이 감소
- 순수 자바로 DSL 을 구현하면 나머지 코드와 함께 컴파일 가능. (다른 언어의 컴파일러 또는 외부 DSL을 만드는 도구를 사용하지 않아도 됨)
- 새로운 언어를 배우거나 복잡한 외부 도구를 배울 필요가 없음
- DSL 사용자는 기존의 자바 IDE로 자동 완성, 리팩터링 기능 그대로 사용 가능
- 추가 DSL을 쉽게 합칠 수 있음

---

### **다중 DSL**

자바는 아니지만 JVM에서 실행되며 더 유연하고 표현력이 가능한 언어(ex. 스칼라, 그루비…)로 구현한 DSL
코틀린, 실론 같이 스칼라와 호환성이 유지되며 단순하고 쉽게 배울 수 있는 새 언어도 존재한다.
이 언어들은 모두 자바보다 젋으며 제약을 줄이고, 간편한 문법을 지향하도록 설계되었다.

즉, JVM 으로 실행 가능한 언어가 100개가 넘는다 -> 간결한데 호환성도 있는 언어 사용 -> DSL은 기반 프로그래밍 언어 영향 받기에 간결한 DSL 생성 가능 -> 생각만해도 좋지 않은가? 대박이다.

**DSL 친화적이지만 다음과 같은 단점들이 존재**

- 새로운 프로그래밍 언어를 배우거나 누군가 해당 기술을 지니고 있어야 함 → ??????
- 두 개 이상의 언어가 혼재하므로 여러 컴파일러로 소스를 빌드하도록 빌드 과정을 개선해야 함
- JVM 에서 실행되지만 자바와 호환성이 완벽하지 않은 경우가 많음 (성능 손실 가능)

---

### **외부 DSL**

스탠드어론(standalone) 이라고 불리우며 호스팅 언어와는 독립적으로 자체의 문법을 가지는 DSL

자신만의 문법과 구문으로 새 언어를 설계해야 하고 언어 파싱, 파서의 결과 분석, 외부 DSL 실행할 코드를 만들어야 한다.

이 방법을 선택해야 한다면 ANTLR 같은 자바 기반 파서 생성기를 이용하면 도움을 받을 수 있음

- 장점
    - 무한한 유연성을 가짐
    - 필요한 특성을 완벽하게 제공하는 언어로 설계 가능
    - 비즈니스 문제를 묘사하고 해결하는 가독성 좋은 언어 선택 가능
    - 자바로 개발된 인프라구조 코드와 비즈니스 코드를 명확하게 분리 가능
- 단점
    - 일반적인 작업이 아니며 쉽게 기술을 얻을 수 없음
    - 작업이 복잡하고 제어 범위를 쉽게 벗어날 수 있음
    - 처음 설계한 목적을 벗어나는 경우가 많음
    - DSL과 호스트 언어 사이에 인공 계층이 생김

---

# 10.2 최신 자바 API의 작은 DSL

네이티브 자바 API 에는 자바 새로운 기능의 장점들이 적용되었다.
스트림 API 를 통해 DSL 이 사용된 예를 확인한다.

### **스트림 API는 컬렉션을 조작하는 DSL**

`Stream` 인터페이스는 네이티브 자바 API 에 작은 내부 DSL을 적용한 좋은 예다.

컬렉션의 항목을 필터, 정렬, 변환, 그룹화, 조작하는 작지만 강력한 DSL 이다.

`Stream` 인터페이스를 이용하여 함수형으로 구현하면 쉽고 간결하다.

스트림 API의 플루언트 형식 또한 설계된 DSL의 특징 중 하나이다.(중간 연산 게으름, 다른 연산으로 파이프라인 가능)

```java
**Files.**lines**(Paths.**get**(**fileName**))
		 .**filter**(**line **->** line**.**startWith**(**"ERROR"**))
		 .**limit**(**40**)
		 .**collect**(**toList**());**
```

---

### **데이터를 수집하는 DSL인 Collectors**

`Collector` 인터페이스는 데이터 수집(수집, 그룹화, 파이션)을 수행하는 DSL로 간주할 수 있다.

특히 다중 필드 정렬을 지원하도록 합쳐질 수 있으며, `Collectors` 는 다중 수준 그룹화를 달성할 수 있도록 합쳐질 수 있다.

```java
Map<String, Map<Color, List<Car>>> carsByBrandAndColor = 
    cars.stream().collect(grouping(Car::getBrand, groupingBy(Car::getColor)));
    
// 두 Comparators 연결    
Comparator<Person> comparator = 
    comparing(Person::getAge).thenComparing(Person::getName);
// 중첩
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector = 
    groupingBy(Car::getBrand, groupingBy(Car::getColor));
```

---

# 10.3 자바로 DSL을 만드는 패턴과 기법

간단한 도메인 모델을 정희하면서 DSL을 만드는 패턴을 알아본다.

처음에는 주식 가격을 모델링하는 순수 자바 빈즈다.

```java
public class Stock {
    private String symbol;
    private String market;
    public String getSymbol(){
        return symbol;
    }
    public void setSymbol(String symbol) {
        this.symbol = symbol;
    }
    ...
}
public class Trade {
    public enum Type { BUY, SELL }
    
    private Type type;
    private Stock stock;
    private int quantity;
    private double price;
    ...    
}

public class Order {
    private String customer;
    private List<Trade> trades = new ArrayList<>();
    ...
}
```

이제 주문을 생성하는 코드는 다음과 같다.

```java
Order order = new Order();
order.setCustomer("BigBank");

Trade trade1 = new Trade();
trade1.setType(Trade.Type.Buy);

Stock stock1 = new Stock();
stock1.setSymbol("IBM");
stock1.setMarket("NYSE");

trade.setStock(stock1);
...
```

하지만 이러한 코드는 장황하고 비개발자인 도메인 전문가가 이해하고 검증하기 어렵다.

직관적으로 도메인 모델을 반영할 수 있는 DSL 이 필요하다. 
이 책에서는 다음과 같은 DSL 패턴들을 소개하고 있다.

- 메서드 체인
- 중첩된 함수
- 람다 표현식을 이용한 함수 시퀀싱

---

### **메서드 체인**

메서드 체인은 DSL 에서 가장 흔한 방식 중 하나이다.

- 장점
    - 사용자가 지정된 절차에 따라 플루언트 API의 메서드 호출을 강제
    - 파라미터가 빌더 내부로 국한 됨
    - 메서드 이름이 인수의 이름을 대신하여 가독성 개선
    - 문법적 잡음이 최소화
    - 선택형 파라미터와 잘 동작
    - 정적 메서드 사용을 최소화하거나 없앨 수 있음
- 단점
    - 빌더를 구현해야 함
    - 상위 수준의 빌더를 하위 수준의 빌더와 연결할 많은 접착 코드가 필요
    - 도멘인 객체의 중첩 구조와 일치하게 들여쓰기를 강제할 수 없음
    

```java
Order order = forCustomer("BigBank")
    .buy(80)
    .stock("IBM")
    .on("NYSE")
    ...
    .end();
```

이러한 코드를 만들기 위해서는 최상 수준 빌더를 만들고 주문을 감싸서 거래를 추가할 수 있도록 해야 한다.

```java
public class MethodChainingOrderBuilder {

    public final Order order = new Order();
    
    private MethodChainingOrderBuilder(String customer) {
        order.setCustomer(customer);
    }
    public static MethodChainingOrderBuilder forCustomer(String customer) {
        return new MethodChainingOrderBuilder(customer);
    } 
    public TradeBuilder buy(int quantity) {
        return new TradeBuilder(this, Trade.Type.SELL, quantity);
    }
    public MethodChainingOrderBuilder addTrade(Trade trade) {
        order.addTrade(trade);
        return this;
    }
    public Order end() {
        return order;
    }
}

public class TradeBuilder {
    private final MethodChainingOrderBuilder builder;
    public final Trade trade = new Trade();
    
    private TradeBuilder(MethodChainingOrderBuilder builder, Trade.Type type, int quantity) {
        this.builder = builder;
        trade.setType(type);
        trade.setQuantity(quantity);
    }
    public StockBuilder stock(String symbol) {
        return new StockBuilder(builder, trade, symbol);
    }
}

public class StockBuilder {
    private final MethodChainingOrderBuilder builder;
    private final Trade trade;
    private final Stock stock = new Stock();
    
    private StockBuilder(MethodChainingOrderBuilder builder, Trade trade, String symbol) {
        this.builder = builder;
        this.trade = trade;
        stock.setSymbol(symbol);
    } 
    public TradeBuilderWithStock on(String market) {
        stock.setMarket(market);
        trade.setStock(stock);
        return new TradeBuilderWithStock(builder, trade);
    }    
}

public class TradeBuilderWithStock {
    private final MethodChainingOrderBuilder builder;
    private final Trade trade;
    
    public TradeBuilderWithStock(MethodChainingOrderBuilder builder, Trade trade) {
        this.builder = builder;
        this.trade = trade;
    }
    public MethodChainingOrderBuilder at(double price) {
        trade.setPrice(price);
        return builder.addTrade(trade);
    }
}
```

---

### **중첩된 함수 이용**

중첩된 함수 DSL 패턴은 다른 함수 안에 함수를 이용해 도메인 모델을 만든다.

- 장점
    - 중첩 방식이 도메인 객체 계층 구조에 그대로 반영
    - 구현의 장황함을 줄일 수 있음
- 단점
    - 결과 DSL에 더 많은 괄호를 사용
    - 정적 메서드 사용이 빈번
    - 인수 목록을 정적 메서드에 넘겨줘야 함
    - 인수의 의미가 이름이 아니라 위치에 의해 정의 됨
    - 도메인에 선택 사항 필드가 있으면 인수를 생략할 수 있으므로 메서드 오버로딩 필요
    

```java
Order order = order("BigBank",
                    buy(80, stock("IBM", on("NYSE")), at(125.00)),
                    sell(50, stock("GOOGLE", on("NASDAQ")), at(375.00))
                    );
```

구현하는 코드는 다음과 같다.

```java
public class NestedFunctionOrderBuilder {

    public static Order order(String customer, Trade... trades) {
        Order order = new Order();
        order.setCustomer(customer);
        Stream.of(trades).forEach(order::addTrade);
        return order;
    }
    public static Trade buy(int quantity, Stock stock, double price) {
        return buildTrade(quantity, stock, price, Trade.Type.BUY);
    }    
    private static Trade buildTrade(int quantity, Stock stock, double price, Trade.Type type) {
        Trade trade = new Trade();
        trade.setQuantity(quantity);
        trade.setType(type);
        trade.setStock(stock);
        trade.setPrice(price);
        return trade;
    }
    public static double at(double price) {
        return price;
    }
    publid static Stock stock(String Symbol, String market) {
        Stock stock = new Stock();
        stock.setSymbol(symbol);
        stock.setMarket(market);
        return stock;
    }
}
```

---

### **람다 표현식을 이용한 함수 시퀀싱**

람다 표현식으로 정의한 함수 시퀀스를 사용한다.

- 장점
    - 플루언트 방식으로 도메인 객체 정의 가능
    - 중첩 방식이 도메인 객체 계층 구조에 그대로 반영
    - 선택형 파라미터와 잘 동작
    - 정적 메서드를 최소화하거나 없앨 수 있음
    - 빌더의 접착 코드가 없음
- 단점
    - 설정 코드가 필요
    - 람다 표현식 문법에 의한 잡음의 영향을 받음

```java
Order order = order(o -> {
    o.forCustomer("BigBank");
    o.buy(t -> {
        t.quantity(80);
        t.price(125.00);
        t.stock(s -> {
            s.symbol("IBM");
            s.market("NYSE");
        });
    });
})
```

이런 DSL을 만들기 위해서는 람다 표현식을 받아 실행해 도메인 모델을 만드는 빌더를 구현해야 한다.

```java
public class LambdaOrderBuilder {

    private Order order = new Order();
    
    public static Order order(Consumer<LambdaOrderBuilder> consumer) {
        LambdaOrderBuilder builder = new LambdaOrderBuilder();
        consumer.accept(builder);
        return builder.order;
    }
    public void forCustomer(String customer) {
        order.setCustomer(customer);
    }
    public void buy(Consumer<TradeBuilder> consumer) {
        trade(consumer, Trade.Type.BUY);
    }
    private void trade(Consumer<TradeBuilder> consumer, Trade.Type type) {
        TradeBuilder builder = new TradeBuilder();
        builder.trade.setType(type);
        consumer.accept(builder);
        order.addTrade(builder.trade);
    }
}

public class TradeBuilder {

    private Trade trade = new Trade();
    
    public void quantity(int quantity) {
        trade.setQuantity(quantity);
    }
    public void price(double price) {
        trade.setPrice(price);
    }
    public void stock(Consumer<StockBuilder> consumer) {
        StockBuilder builder = new StockBuilder();
        consumer.accept(builder);
        trade.setStock(builder.stock);
    }
}
...
```

---

### **조합하기**

중첩된 함수 패턴과 람다 기법을 혼용하면 다음과 같이 사용할 수 있다.

- 장점
    - 가독성 향상
- 단점
    - 여러 기법을 혼용했기 때문에 사용자가 DSL을 배우는 시간이 오래 걸림

```java
Order order = 
    forCustomer("BigBank", 
                buy(t -> t.quantity(80)
                          .stock("IBM")
                          .on("NYSE")
                          .at(125.00)),
                sell(t -> t.quantity(50)
                           .stock("GOOGLE")
                           .on("NASDAQ")
                           .at(125.00)));
```

이 코드를 사용하기 위한 빌더 코드는 다음과 같다.

```java
public class MixedBuilder {

    public static Order forCustomer(String customer, TradeBuilder... builders) {
        Order order = new Order();
        order.setCustomer(customer);
        Stream.of(builders).forEach(b -> order.addTrade(b.trade));
        return order;
    } 
    public static TradeBuilder buy(Consumer<TradeBuilder> consumer) {
        return builderTrade(consumer, Trade.Type.BUY);
    }
    private static TradeBuilder buildTrade(Consumer<TradeBuilder> consumer, Trade.Type type) {
        TradeBuilder builder = new TradeBuilder();
        builder.trade.setType(buy);
        consumer.accept(builder);
        return builder;
    }
}

public class TradeBuilder {
    
    private Trade trade = new Trade();
    
    public TradeBuilder quantity(int quantity) {
        trade.setQuantity(quantity);
        return this;
    }
    public TradeBuilder at(double price) {
        trade.setPrice(price);
        return this;
    }
    public StockBuilder stock(String symbol) {
        return new StockBuilder(this, trade, symbol);
    }
}

public class StockBuilder {
    
    private final TradeBuilder builder;
    private final Trade trade;
    private final Stock stock = new Stock();
    
    public StockBuilder(TradeBuilder builder, Trade trade, String symbol) {
        this.builder = builder;
        this.trade = trade;
        stock.setSymbol(symbol);
    }
    public TradeBuilder on(String market) {
        stock.setMarket(market);
        trade.setStock(stock);
        return builder;
    }
}
```

---

### **DSL에 메서드 참조 사용하기**

주문의 총 합에 세금을 추가해 최종값을 계산하는 기능을 추가한다.

```java
public class Tax {
    public static double regional(double value) {
        return value * 1.1;
    }
    public static double general(double value) {
        return value * 1.3;
    }
    public static double surcharge(double value) {
        return value * 1.05;
    }
}
```

함수형 기능을 이용하여 간결하고 유연한 방식으로 구현한다.

```java
public class TaxCalculator {

    public DoubleUnaryOperator taxFunction = d -> d;
    
    public TaxCalculator with(DoubleUnaryOperator f) {
        taxFunction = taxFunction.andThen(f);
        return this;
    }
    public double calculate(Order order) {
        return taxFunction.applyAsDouble(order.getValue());
    }
}

double value = new TaxCalculator().with(Tax::regional)
                                  .with(Tax::surcharge)
                                  .calculate(order);
```

---

# 10.4 실생활의 자바 8 DSL

DSL을 개발하는데 사용하는 유용한 패턴에 대해 알아봤으니
이 패턴들이 얼마나 사용되고 있는지 살펴본다.

### **jOOQ**

jOOQ는 SQL을 구현하는 내부적 DSL로 자바에 직접 내장된 형식 안전 언어다.

jOOQ DSL 구현하는 데에는 메서드 체인 패턴을 사용했다.

SQL 문법을 흉내내려면 선택적 파라미터를 허용하고 정해진 순서대로 특정 메서드가 호출되어야 하기 때문에 메서드 체인 패턴의 특성이 필요하다.

```java
SELECT * FROM BOOK 
WHERE BOOK.PUBLISHED_IN = 2016 
ORDER BY BOOK.TITLE
```

위 SQL 질의를 jOOQ DSL 을 이용하면 다음처럼 구현할 수 있다.

```java
create.selectFrom(BOOK)
      .where(BOOK.PUBLISHED_IN.eq(2016))
      .orderBy(BOOK.TITLE)
```

jOOQ DSL은 스트림 API 와 조합해서 사용할 수 있다.

```java
Class.forName("org.h2.Driver");
try(Connection c = 
        getConnection("jdbc:h2:~/sql-goodies-with-mapping", "sa", "")) {
    DSL.using(c)
       .select(BOOK.AUTHOR, BOOK.TITLE)
       .where(BOOK.PUBLISHED_IN.eq(2016))
       .orderBy(BOOK.TITLE)
    .fetch()
    .stream()
    .collect(groupingBy(
        r -> r.getValue(BOOK.AUTHOR),
        LinkedHashMap::new,
        mapping(r -> r.getValue(BOOK.TITLE), toList())))
        .forEach((author, titles) -> System.out.println(author + " is author of " + title));
}
```

---

### **큐컴버**

동작 주도 개발(BDD)은 테스트 주도 개발의 확장으로 다양한 비즈니스 시나리오를 구조적으로 서술하는 도메인 전용 스크립팅 언어를 사용한다.

큐컴버(cucumber)는 BDD 프레임워크로 명령문을 실행할 수 있는 테스트 케이스로 변환하며, 비즈니스 시나리오를 평문으로 구현할 수 있도록 도와준다.

큐컴버는 세가지 구분되는 개념을 사용한다.

- Given: 전제 조건 정의
- When: 시험하려는 도메인 객체의 실질 호출
- Then: 테스트 케이스의 결과를 확인

```java
Feature: Buy stock
  Scenario: Buy 10 IBM stocks
    Given the price of a "IBM" stock is 125$
    When I buy 10 "IBM"
    Then the order value should be 1250$
```

테스트 시나리오를 정의하는 스크립트는 제한된 키워드를 제공하며 자유로운 문장을 구현할 수 있는 외부 DSL을 활용한다.

테스트 케이스의 변수를 정규 표현식으로 캡쳐할 수 있으며, 테스트를 구현하는 메서드로 전달한다.

```java
public class BuyStocksSteps {
    private Map<String, Integer> stockUnitPrices = new HashMap<>();
    private Order order = new Order();
    
    @Given("^the price of a \"(.*?)\" stock is (\\d+)\\$$")
    public void setUnitPrice(String stockName, int unitPrice) {
        stockUnitValues.put(stockName, unitPrice);
    }
    @When("^I buy (\\d+) \"(.*?)\"$")
    public void buyStocks(int quantity, String stockName) {
        Trade trade = new Trade();
        trade.setType(Trade.Type.BUY);
        ...
    }
    @Then("^the order value should be (\\d+)\\$$")
    public void checkOrderValue(int expectedValue) {
        assertEquals(expectedValue, order.getValue());
    }
}
```

어노테이션을 제거하고 다른 문법으로도 개발이 가능하다.

이 방식을 이용하면 코드가 단순해지고 메서드가 무명 람다로 바뀌면서 메서드 이름을 찾는 부담이 없어진다.

```java
public class BuyStocksSteps implements cucumber.api.java8.En {
    private Map<String, Integer> stockUnitPrices = new HashMap<>();
    private Order order = new Order();
    public BuyStocksSteps() {
        Given("^the price of a \"(.*?)\" stock is (\\d+)\\$$"), 
              (String stockName, int unitPrice) -> {
                  stockUnitValues.put(stockName, unitPrice);
              });
        ...
    }
}
```

---

### **스트링 통합**

스프링 통합은 유명한 엔터프라이즈 통합 패턴을 지원할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모델을 확장한다.

복잡한 통합 솔루션 모델을 제공하고 비동기, 메시지 주도 아키텍처를 쉽게 적용할 수 있도록 돕는 것이 스프링 통합의 핵심 목표다.

스프링 통합 DSL 에서도 메서드 체인 패턴이 가장 널리 사용되고 있다.
