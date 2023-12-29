# Chapter10 람다를 이용한 도메인 전용 언어

도메인 전용 언어(Domain-specific languages, DSL)을 사용해 비즈니스 로직을 표현하면 소프트웨어의 영역에서 읽기 쉽고, 이해하기 쉬운 코드를 만들 수 있다.

예를 들어, 메이븐Maven, 앤트Ant 는 빌드 과정을 표현하는 DSL이라고 할 수 있다. 이러한 DSL을 예제와 함께 직접 구현해보며 진행 할 것이다.

# 도메인 전용 언어

DSL은 특정 비즈니스 도메인의 문제를 해결하려고 만든 언어이다.

**DSL이란**

- 특정 비즈니스 도메인을 인터페이스로 만든 API
- 두가지 중요한 특성
    - 의사소통의 왕 : 코드의 의도가 명확히 전달되어야 하며, 프로그래머가 아닌 사람도 이해할 수 있어야 한다.
    - 코드는 한번 구현되지만 여러 번 읽는다 : 가독성은 유지보수의 핵심, 프로그래머가 쉽게 이해할 수 있도록 코드를 구현해야 한다.

## DSL의 장점과 단점

DSL은 코드의 비즈니스 의도를 명확하게 하고 가독성을 높이는 반면, DSL 구현은 코드이므로 올바르게 검증하고 유지보수해야한다. DSL의 장점과 비용을 생각해서 고려해야 할 것이다.

- **장점**
    - **간결함** : 비즈니스 로직을 간편하게 캡슐화 하기 때문에 반복을 피할 수 있고 간결하게 만들어진다.
    - **가독성** : 비 도메인 전문가도 코드를 쉽게 이해할 수 있다.
    - **유지보수 :** 잘 설계된 DSL은 쉽게 유지보수하고 바꿀 수 있다.
    - **높은 수준의 추상화** : 직접적으로 관련되지 않은 세부 사항을 숨길 수 있다.
    - **집중** : 특정 코드에 집중해서 구현할 수 있다.
    - **관심사분리** : 인프라 관련 문제와 독립적으로 비즈니스 코드에 집중하기 쉽다.
- **단점**
    - **DSL 설계의 어려움** : 간결하게 제한적인 언어에 도메인 지식을 담기 어렵다.
    - **개발 비용** : DSL을 추가하는 작업은 많은 비용과 시간이 소모된다.
    - **추가 우회 계층** : 추가적인 계층으로 도메인 모델을 감싼다
    - **새로 배워야 하는 언어** : 한 프로젝트에 여러 언어를 사용한다면 팀이 배워야 할 언어가 더 늘어난다.
    - **호스팅 언어 한계** : 자바같은 범용 프로그래밍 언어는 장황하고 엄격하기 때문에 사용자 친화적인 DSL이 만들어지기 힘들다
    

## JVM에서 이용할 수 있는 다른 DSL 해결책

DSL을 구분하는 방식은 내부 외부를 나누는 것이다. 이 때 구분되는 조건은 내부의 경우 **호스팅 언어**를 기반으로 구현되고, 외부는 **호스팅 언어와 독립적**으로 자체 문법이 있다.

- **내부 DSL**
    - 자바로 구현한 DSL
    
    ```java
    		List<String> numbers = Arrays.asList("one", "two", "three");
    
        System.out.println("Anonymous class:");
        numbers.forEach(new Consumer<String>() {
    
          @Override
          public void accept(String s) {
            System.out.println(s);
          }
    
        });
    
    ->
    
    numbers.forEach(System.out::println);
    ```
    
    - 장점
        - 외부 DSL에 비해 새로운 패턴과 기술을 배워 DSL을 구현하는 노력이 현저하게 줄어든다
        - 개발팀이 새로운 언어를 배울 필요가 없다
        - IDE의 활용을 100% 할 수 있다.
- **다중 DSL**
    - JVM에서 실행되는 언어는 100개가 넘는다. 이중 좀 더 간결하고 명확한 언어를 활용하는 것
    - 단점
        - 새로운 프로그래밍 언어를 잘 알고 있는 인원이 있어야한다.
        - 두 개 이상의 언어가 혼재하므로 여러 컴파일러로 소스를 빌드하도록 개선해야 한다
        - JVM의 호환성이 100%는 아니다
- **외부 DSL**
    - 새로운 언어를 직접 설계하는 방식
    - 비용이 상당히 많이 들고, 설계자의 수준이 높아야한다.
    - 큰 장점은 외부 DSL이 제공하는 **유연성**이다.

# 최신 자바 API의 작은 DSL

첫 자바 DSL은 간결하게 동작하는 코드로 전환되는 Java8버전부터 시작된다.

```java
Collections.sort(persons, new Comparator<Person>(){
	public int compare(Person p1, Person p2){
		return p1.getAge() - p2.getAge();
	}
});

->

Collections.sort(persons, comparing(Person::getAge));

->

persons.sort(comparing(Person::getAge)
	.thenComparing(Person::getName));
```

- 람다 표현식이나, 메서드 참조를 통해 가독성을 챙김으로써 DSL에 적합한 코드로 바꾼 것이다.
- 모든 영역에 가능한 것은 아니지만, 이를 통해 `가독성`, `재사용성`, `결합성` 을 높일 수 있다.

## 스트림API는 컬렉션을 조작하는 DSL

Stream 인터페이스는 네이티브 자바 API에 작은 내부 DSL을 적용한 좋은 예시, 컬렉션 항목을 `필터링`, `정렬`, `변환`, `그룹화`, `조작` 등의 동작은 간결하게 수행할 수 있고 가독성 또한 좋기 때문이다.

책의 예제(p332)인 파일을 읽어들이는 형태의 긴 코드를 stream API를 활용한다면 아래와 같다.

```java
List<String> errors =
	Files.lines(Paths.get(fileName)) // stream<String>을 반환함
	.filter(line -> line.startsWith("ERROR")) //필터링
	.limit(40) // 40행 제한
	.collect(toList()); // 결과 문자열 리스트화
```

## 데이터를 수집하는 DSL인 Collectors

**DSL관점에서의 Collectors 클래스**

```java
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector = 
	groupingBy(Car::getBrand, groupingBy(Car::getColor));
```

위와 같이 Collectors.groupingBy를 통해 간결해 보이도록 처리할 수 있다.

**그룹화 컬렉터 빌더**

```java
public static class GroupingBuilder<T, D, K> {

    private final Collector<? super T, ?, Map<K, D>> collector;

    public GroupingBuilder(Collector<? super T, ?, Map<K, D>> collector) {
      this.collector = collector;
    }

    public Collector<? super T, ?, Map<K, D>> get() {
      return collector;
    }

    public <J> GroupingBuilder<T, Map<K, D>, J> after(Function<? super T, ? extends J> classifier) {
      return new GroupingBuilder<>(groupingBy(classifier, collector));
    }

    public static <T, D, K> GroupingBuilder<T, List<T>, K> groupOn(Function<? super T, ? extends K> classifier) {
      return new GroupingBuilder<>(groupingBy(classifier));
    }

  }

->

Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>>
	carGroupingCollector = 
		groupOn(Car::getColor).after(Car::getBrand).get();
```

그룹화 함수를 구현해야 하므로 유틸리티 사용 코드가 직관적이지 않다

## 자바로 DSL을 만드는 패턴과 기법

**예제를 통해 이해해보기**(p336)

- 도메인 모델 구성
    - Stock
        - 시장에 주식 가격을 모델링하는 자바 빈
    - Trade
        - 주어진 가격에서 주어진 양의 주식을 사거나 파는 거래
    - Order
        - 고객이 요청한 한 개 이상의 거래 주문

이 처리방식을 일반 코드로 작성한다면 많이 장황해지기 때문에, 이를 직관적으로 표현하기 위해 DSL형태로 진행

## 메서드 체인

DSL의 가장 흔한 방식중 하나이다.

```java
Order order = forCustomer("BihBank")
	.buy(80)
	.stock("IBM")
	.on("NYSE")
	.at(125.00)
	.sell(50)
	.stock("GOOGLE")
	.on("NASDAQ")
	.at(375.00)
	.end();
```

- 위처럼 활용하기 위해 DSL빌더를 추가해야한다.(p340)
- 최상위 수준 빌더를 만들고 주문을 감싸고 한 개 이상의 거래를 주문에 추가할 수 있어야 한다.
- 메서드 체인 형식은 빌더를 이처럼 직접 만들어야 하는 것이 단점이다.

## 중첩된 함수 이용

**중첩된 함수 DSL**은 다른 함수 안에 함수를 이용해 도메인 모델을 만든다.

```java
Order order = 
	order("BigBank",
		buy(80,
			stock("IBM", on("NYSE")), at(125.00)),
		sell(50,
			stock("GOOGLE", on("NASDAQ")), at(375.00))
		);
```

- 빌더를 추가해야 한다(p343)
- 메서드 체이닝 보다 간단하다
- 함수의 중첩 방식이 도메인 객체 계층 구조에 그대로 반영된다.
- DSL에 많은 괄호를 사용해야 하고, 인수목록을 메서드에 넘겨야 한다는 제약이 있다

## 람다 표현식을 이용한 함수 시퀀싱

```java
Order order = 
	order(o -> {
		o.forCustomer("BigBank");
		o.buy(t-> {
			t.quantity(80);
			t.price(125.00);
			t.stock( s-> {
				s.symbol("IBM")
				s.market("NYSE");
			});
		});
		o.sell(t-> {
			t.quantity(50);
			t.price(375.00);
			t.stock( s-> {
				s.symbol("GOOGLE")
				s.market("NASDAQ");
			});
		});
```

- 람다 표현식을 받는 여러 빌더를 구현해야 한다.
- Cunsumer 객체를 빌더가 인수로 받음으로 DSL 사용자가 람다 표현식으로 인수를 구현할 수 있게 되었다.

## 조합하기

- 세가지 DSL 패턴을 조합해서 활용하기도 한다
- DSL이 여러가지 기법을 혼용하기 때문에 사용자가 DSL을 배우는 시간이 오래걸린다.

## DSL에 메서드 참조 사용하기

```java
double value = new TaxCalculator().with(Tax::regional)
		.with(Tax::surcharge)
		.calculate(order);
```

- 읽기 쉽고 코드를 간결하게 만든다.
- Tax 클래스에 추가해도 함수형 TaxCalculator를 바꾸지 않고 바로 사용할 수 있다.

# 실생활의 자바 8 DSL
