## DSL

DSL은 특정 비즈니스 도메인의 문제를 해결하려고 만든 언어이다. 즉, 특정 비즈니스 도메인을 인터페이스로 만든 API라고 생각하면 된다.

DSL은 범용 프로그래밍 언어가 아니다. DSL에서 동작과 용어는 특정 도메인에 국한되므로 다른 문제는 걱정할 필요가 없고 오직 자신의 앞에 놓인 문제를 어떻게 해결할지에만 집중할 수 있다.

DSL 장점

- 간결함
- 가독성
- 유지보수
- 높은 수준의 추상화
- 관심사분리
- 집중

DSL 단점

- 설계의 어려움
- 개발 비용
- 추가 우회 계층
- 호스팅 언어의 한계


### JVM에서 이용할 수 있는 다른 DSL 해결책

DSL의 카테고리를 구분하는 가장 흔한 방법은 내부 DSL과 외부 DSL을 나누는 것이다. 내부 DSL은 순수 자바 코드같은 기존 호스팅 언어를 기반으로 구현하는 반면, 외부 DSL은 호스팅 언어와는 독립적으로 자체의 문법을 가진다.

### 내부 DSL

순수 자바코드 같은 기존 호스팅 언어를 기반으로 구현한 DSL
유연성이 떨어지기 때문에 간단하고 표현력 있는 DSL을 만드는데 한계가 있었지만 람다 표현식이 등장하면서 어느정도 해결될 수 있음
순수 자바로 DSL 을 구현하면 다음과 같은 장점이 존재한다.

- 외부 DSL에 비해 새로운 패턴과 기술을 배워 DSL을 구현하는 노력이 감소
- 순수 자바로 DSL 을 구현하면 나머지 코드와 함께 컴파일 가능. (다른 언어의 컴파일러 또는 외부 DSL을 만드는 도구를 사용하지 않아도 됨)
- 새로운 언어를 배우거나 복잡한 외부 도구를 배울 필요가 없음
- DSL 사용자는 기존의 자바 IDE로 자동 완성, 리팩터링 기능 그대로 사용 가능
- 추가 DSL을 쉽게 합칠 수 있음

### 다중 DSL

자바는 아니지만 JVM에서 실행되며 더 유연하고 표현력이 가능한 언어(ex. 스칼라, 그루비…)로 구현한 DSL 코틀린, 실론 같이 스칼라와 호환성이 유지되며 단순하고 쉽게 배울 수 있는 새 언어도 존재한다.
이 언어들은 모두 자바보다 젋으며 제약을 줄이고, 간편한 문법을 지향하도록 설계되었다.
DSL 친화적이지만 다음과 같은 단점들이 존재한다.

- 새로운 프로그래밍 언어를 배우거나 누군가 해당 기술을 지니고 있어야 함
- 두 개 이상의 언어가 혼재하므로 여러 컴파일러로 소스를 빌드하도록 빌드 과정을 개선해야 함
- JVM 에서 실행되지만 자바와 호환성이 완벽하지 않은 경우가 많음 (성능 손실 가능)

### 외부 DSL

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
  

## 최신 자바 API의 작은 DSL

네이티브 자바 API 에는 자바 새로운 기능의 장점들이 적용되었다.
람다와 메서드 참조를 이용한 DSL이 코드의 가독성, 재사용성, 결합성을 높일 수 있다.

### 스트림 API는 컬렉션을 조작하는 DSL

Stream 인터페이스는 네이티브 자바 API에 작은 내부 DSL을 적용한 좋은 예다.
컬렉션의 항목을 필터, 정렬, 변환, 그룹화, 조작하는 작지만 강력한 DSL로 볼 수 있다.

```java
List<String> errors = Files.lines(Paths.get(fileName))
    						.filter(line -> line.startWith("ERROR"))
    						.limit(40)
    						.collect(toList());
```

스트림 API의 플루언트 형식은 잘 설계된 DSL의 또 다른 특징이며, 모든 중간 연산은 게으르며 다른 연산으로 파이프라인될 수 있는 스트림으로 반환된다.

### 데이터를 수집하는 DSL인 Collectors

Collector 인터페이스는 데이터 수집을 수행하는 DSL로 간주할 수 있다.
특히 다중 필드 정렬을 지원하도록 합쳐질 수 있으며, Collectors는 다중 수준 그룹화를 달성할 수 있도록 합쳐질 수 있다.

## 자바로 DSL을 만드는 패턴과 기법

DSL은 특정 도메인 모델에 적용할 친화적이고 가독성 높은 API를 제공한다.

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

주어진 가격에서 주어진 양의 주식을 사거나 파는 거래클래스이다.

```java
public class Trade {

  public enum Type {
    BUY,
    SELL
  }

  private Type type;
  private Stock stock;
  private int quantity;
  private double price;

  // getter, setter 생략
}

```

고객이 요청한 한 개 이상의 거래의 주문

```java
public class Order {

  private String customer;
  private List<Trade> trades = new ArrayList<>();

  public void addTrade( Trade trade ) {
    trades.add(trade);
  }

  public String getCustomer() {
    return customer;
  }

  public void setCustomer( String customer ) {
    this.customer = customer;
  }

  public double getValue() {
    return trades.stream().mapToDouble( Trade::getValue ).sum();
  }
}
```

인제 주문을 의미하는 객체를 만들어보겠다.

```java
    Order order = new Order();
    order.setCustomer("BigBank");

    Trade trade1 = new Trade();
    trade1.setType(Trade.Type.BUY);

    Stock stock1 = new Stock();
    stock1.setSymbol("IBM");
    stock1.setMarket("NYSE");

    trade1.setStock(stock1);
    trade1.setPrice(125.00);
    trade1.setQuantity(80);
    order.addTrade(trade1);

    Trade trade2 = new Trade();
    trade2.setType(Trade.Type.BUY);

    Stock stock2 = new Stock();
    stock2.setSymbol("GOOGLE");
    stock2.setMarket("NASDAQ");

    trade2.setStock(stock2);
    trade2.setPrice(375.00);
    trade2.setQuantity(50);
    order.addTrade(trade2);

    System.out.println("Plain:");
    System.out.println(order);
```

비개발자가 봤을 경우 이 코드를 이해하고 검증하기는 상당히 어렵다. 조금 더 직접적이고, 직관적으로 도메인 모데을 반영할 수 있는 DSL을 적용해봐야 한다.

### 메서드 체인

전에 배웠던 메서드 체인 디자인 패턴을 이용하여 코드를 좀 더 간략하게 바꿀 수 있다.

```java
        .buy(80).stock("IBM").on("NYSE").at(125.00)
        .sell(50).stock("GOOGLE").on("NASDAQ").at(375.00)
        .end();

    System.out.println(order);
```

상당히 길던 코드가 개선되었다. 결과를 달성하기 위해서는 플루언트 API로 도메인 객체를 만드는 몇개의 빌더를 구현해야 한다.

```java
public class MethodChainingOrderBuilder {

  public final Order order = new Order();

  private MethodChainingOrderBuilder(String customer) {
    order.setCustomer(customer);
  }

  public static MethodChainingOrderBuilder forCustomer(String customer) {
    return new MethodChainingOrderBuilder(customer);
  }

  public Order end() {
    return order;
  }

  public TradeBuilder buy(int quantity) {
    return new TradeBuilder(this, Trade.Type.BUY, quantity);
  }

  public TradeBuilder sell(int quantity) {
    return new TradeBuilder(this, Trade.Type.SELL, quantity);
  }

  private MethodChainingOrderBuilder addTrade(Trade trade) {
    order.addTrade(trade);
    return this;
  }
}
```

주문 빌더의 buy(), sell() 메서드는 다른 주문을 만들어 추가할 수 있도록 자신을 만들어 반환한다.

```java
  public static class TradeBuilder {

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
}
```

빌더를 계속 이어가기 위해 Stock 클래스의 인스턴스를 만들어준다.

```java
  public static class StockBuilder {

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
```
StockBuilder는 주식의 시장을 지정하고, 거래에 주식을 추가하고 최종 빌더를 반환하는 on() 메서드를 정의한다.

```java
  public static class TradeBuilderWithStock {

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

메서드 체인의 장, 단점

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

### 중첩된 함수 이용

중첩된 함수 DSL 패턴은 이름에서 알 수 있듯이 다른 함수 안에 함수를 이용해 도메인 모델을 만든다.

```java
    Order order = order("BigBank",
        buy(80,
            stock("IBM", on("NYSE")),
            at(125.00)),
        sell(50,
            stock("GOOGLE", on("NASDAQ")),
            at(375.00))
    );

    System.out.println(order);
```

구현 코드는 다음과 같다.

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

  public static Trade sell(int quantity, Stock stock, double price) {
    return buildTrade(quantity, stock, price, Trade.Type.SELL);
  }

  private static Trade buildTrade(int quantity, Stock stock, double price, Trade.Type buy) {
    Trade trade = new Trade();
    trade.setQuantity(quantity);
    trade.setType(buy);
    trade.setStock(stock);
    trade.setPrice(price);
    return trade;
  }

  public static double at(double price) {
    return price;
  }

  public static Stock stock(String symbol, String market) {
    Stock stock = new Stock();
    stock.setSymbol(symbol);
    stock.setMarket(market);
    return stock;
  }

  public static String on(String market) {
    return market;
  }

}
```

중첩된 함수 장, 단점

- 장점
  - 도메인 객체 계층 구조에 그대로 반영
  
- 단점
  - 결과 DSL에 더 많은 괄호를 사용
  - 인수 목록을 정적 메서드에 넘겨줘야하는 제약
  - 인수의 의미가 이름이 아니라 위치에 의해 정의 되야함

### 람다 표현식을 이요한 함수 시퀀싱

람다 표현식으로 정의한 함수 시퀀스를 사용한다.

```java
  public void lambda() {
    Order order = LambdaOrderBuilder.order( o -> {
      o.forCustomer( "BigBank" );
      o.buy( t -> {
        t.quantity(80);
        t.price(125.00);
        t.stock(s -> {
          s.symbol("IBM");
          s.market("NYSE");
        });
      });
      o.sell( t -> {
        t.quantity(50);
        t.price(375.00);
        t.stock(s -> {
          s.symbol("GOOGLE");
          s.market("NASDAQ");
        });
      });
    });

    System.out.println(order);
  }
```
DSL을 만들려면 람다 표현식을 받아 실행해 도메인 모델을 만들어 내는 여러 빌더를 구현해야 한다. (메서드 체인 패턴을 이용해 만들려는 객체의 중간 상태를 유지한다.)

아래는 람다 표현식을 이용한 구현 코드이다.

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

  public void sell(Consumer<TradeBuilder> consumer) {
    trade(consumer, Trade.Type.SELL);
  }

  private void trade(Consumer<TradeBuilder> consumer, Trade.Type type) {
    TradeBuilder builder = new TradeBuilder();
    builder.trade.setType(type);
    consumer.accept(builder);
    order.addTrade(builder.trade);
  }

  public static class TradeBuilder {

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

  public static class StockBuilder {

    private Stock stock = new Stock();

    public void symbol(String symbol) {
      stock.setSymbol(symbol);
    }

    public void market(String market) {
      stock.setMarket(market);
    }
  }
}
```
람다 표현식의 장, 단점

- 장점
  - 메서드 체인 패턴처럼 플루언트 방식으로 거래 주문 정의 가능
  - 도메인 객체의 계층 구조를 유지함
- 단점
  - 많은 설정 코드가 필요하며 DSL 자체가 자바 8 람다 표현식의 문법에 의존하기 때문에 쓸모 없는 코드에 영향을 받음
  
가장 좋은 방법은 상황에 따른 좋은 패턴을 적용하는 것이다. 이러한 이유로 한가지 패턴만 고집해서 사용하는 것보다 여러 패턴을 적용해서 해결하는 것이 더 좋다. (또한 람다 표현식을 메서드 참조를 이용해서 더 간편하게 구성할 수 있다.)

## 실생활의 자바 8 DSL

DSL을 개발하는데 사용하는 유용한 패턴을 살펴봤고 각각의 장단점도 확인했으니 이 패턴들이 얼마나 사용되는지 확인해보며 내용을 정리해보겠다. SQL 매핑 도구, 동작 주도 개발 프레임워크, 엔터프라이즈 통합 패턴을 구현하는 도구 세 가지 자바 라이브러리를 확인해보겠다.

### jOOQ

SQL은 DSL의 가장 흔히, 광범위하게 사용되는 분야이다.
jOOQ는 SQL을 구현하는 내부적 DSL로 자바에 직접 내장된 형식의 안전 언어이다.

```SQL
SELECT * FROM BOOK 
WHERE BOOK.PUBLISHED_IN = 2016 
ORDER BY BOOK.TITLE
```

jOOQ DSL 이용
```java
create.selectFrom(BOOK)
      .where(BOOK.PUBLISHED_IN.eq(2016))
      .orderBy(BOOK.TITLE)
```

스트림 API와 조합해 사용할 수 있는 것이 jOQQ DSL의 또 다른 장점이다. 이를 이용해 한 개의 플루언트구문으로 데이터를 메모리에서 조작할 수 있다.

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

### 큐컴버

동작 주도 개발은 테스트 주도 개발의 확장으로 다양한 비즈니스 시나리오를 구조적으로 서술하는 간단한 도메인 전용 스크립팅 언어를 사용한다.
BDD는 우선 순위에 따른, 확인할 수 있는 비즈니스 가치를 전달하는 개발 노력에 집중하며 비즈니스 어휘를 공유함으로 도메인 전문가와 프로그래머 사이의 간격을 줄인다.

- 전체 조건 정의 : Given
- 실질 호출 : When
- 테스트 케이스 결과 확인 : Then

```
Feature: Buy stock
  Scenario: Buy 10 IBM stocks
    Given the price of a "IBM" stock is 125$
    When I buy 10 "IBM"
    Then the order value should be 1250$
```

큐컴버 어노테이션을 이용한 테스트 케이스 구현

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

또한 람다로도 간략하게 구성할 수 있다.

```java
    public BuyStocksSteps() {
        Given("^the price of a \"(.*?)\" stock is (\\d+)\\$$"), 
              (String stockName, int unitPrice) -> {
                  stockUnitValues.put(stockName, unitPrice);
              });
        ...
    }
```

### 스프링 통합

스프링 통합은 유명한 엔터프라이즈 통합 패턴을 지우너할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모델을 확장한다. 스프링 통합의 핵심 목표는 비동기, 메시지 주도 아키텍처를 쉽게 적용할 수 있게 돕는 것이다.
스프링 통합은 스프링 기반 애플리케이션 내의 경량의 원격, 메시징, 스케쥴링을 지원한다.
