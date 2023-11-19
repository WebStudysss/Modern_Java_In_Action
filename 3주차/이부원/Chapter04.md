### Stream
---
자바 애플리케이션에서 Collections은 처리하는 과정을 포함하며 데이터를 그룹화하고 처리할 수 있으며 대부분의 프로그래밍 작업에서 사용된다. 예를 들어, 가격이 3,000만원 이하의 자동차 정보를 조회할 때는 아래처럼 SQL문으로 작성한다.

```sql
Select * from Car Where price < 3,000
```

이 질의어 안에서는 우리가 기대하는 것이 무엇인지 정확히 알 수 있으며 직접 표현할 수 있다. SQL에서는 질의를 어떻게 구현해야 할지 명시가 필요 없으며 구현은 자동으로 제공된다. Collections으로 이와 같은 기능을 만들 수 있지않을까? 만약 많은 요소가 있다면 Collections으로는 어떻게 처리해야할까? 성능을 높이기 위해서는 멀티코어 아키텍처를 활용해서 병렬로 컬렉션의 요소를 처리해야 한다. 하지만 병렬처리 코드를 구현하는 것은 단순 반복 처리 코드에 비해 복잡하고 어려우며, 디버깅도 어렵다.

이를 해결하기 위해 만들어진 것이 Stream이다. 그렇다면 Stream은 어떻게 활용할까?

### Stream이란 무엇인가?
---
스트림은 Java 8 API에 추가된 새로운 기능이며, 스트림을 이용하면 선언형으로 컬렉션 데이터로 처리할 수 있다. 또한 스트림을 이용하면 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다.

**Java 7의 기존 코드**

```java
  public static List<String> getLowCaloricDishesNamesInJava7(List<Dish> dishes) {
    List<Dish> lowCaloricDishes = new ArrayList<>();
    
    for (Dish d : dishes) {
      if (d.getCalories() < 400) {
        lowCaloricDishes.add(d);
      }
    }
    
    List<String> lowCaloricDishesName = new ArrayList<>();
    
    Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
      @Override
      public int compare(Dish d1, Dish d2) {
        return Integer.compare(d1.getCalories(), d2.getCalories());
      }
    });
    
    for (Dish d : lowCaloricDishes) {
      lowCaloricDishesName.add(d.getName());
    }
    
    return lowCaloricDishesName;
  }
```

**Java 8 Stream 적용**
```java
  public static List<String> getLowCaloricDishesNamesInJava8(List<Dish> dishes) {
    return dishes.stream()
        .filter(d -> d.getCalories() < 400)
        .sorted(comparing(Dish::getCalories))
        .map(Dish::getName)
        .collect(toList());
  }
```

위의 코드는 칼로리가 400이하이면서 칼로리가 낮은 순으로 오름차순으로 정렬하는 코드였다. Java7에서는 코드의 길이가 상당히 길지만 Stream을 적용하면 무려 10줄이상 줄일 수 있는 것을 확인할 수 있다. 또한 `dishes.stream()` 이 부분을 `dishes.parallelStream()`으로 바꾼다면 병렬로 실행 가능하다.
filter(,sorted, map, collect 등) 같은 연산은 고수준 빌딩 블록으로 이루어져 있으므로 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용할 수 있다. 결과적으로 데이터 처리 과정을 병렬화하면서 스레드와 락을 걱정할 필요가 없습니다.

> 고수준 빌딩 블록방식이란?
소프트웨어 개발에서 사용되는 접근 방식 중 하나로, 작업을 더 작은 더 추상화된 구성 요소로 나누어 작업하는 수행 방법, 이러한 방법은 코드의 재사용성과 유지보수성을 향상시키는데 도움이 됨

Java 8 Stream API의 특징은 3가지로 요약할 수 있다.
- 선언형 : 더 간결하고 가독성이 좋음
- 조립할 수 있음 : 유연성이 좋음
- 병렬화 : 성능이 좋아짐


### 스트림 시작하기
---
> Stream
데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

- **연속된 요소**: 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스 제공, filter, sorted, map처럼 표현 계산식이 주를 이룬다. (컬렉션의 주제는 데이터이고, 스트림은 계산이다.)
- **소스** : 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다.
- **데이터 처리 연산** : 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 DB와 비슷한 연산 지원 (filter, map, reduce, find, match, sort 등)

스트림의 2가지 주요한 특징

- **파이프라이닝** : 스트림은 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 자신을 반환한다. (게으름, 쇼트서킷같은 최적화를 얻을 수 있음)
- **내부 반복** : 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복 지원

```java
    List<String> names = menu.stream()
        .filter(dish -> {
          return dish.getCalories() > 300; <- 파이프라인 연산 만들기 시작
        })
        .map(dish -> {
          return dish.getName();
        })
        .limit(3) // 앞에서 부터 3개만 출력
        .collect(toList()); // 리스트로 저장
    System.out.println(names);
    
    // 메서드 참조 (위와 같은 코드)
    List<String> names = menu.stream()
        .filter(HighCaloriesNames::highCalories)
        .map(HighCaloriesNames::name)
        .limit(3)
        .collect(toList());
    System.out.println(names);
  }

    private static String name(Dish dish) {
        return dish.getName();
    }

    private static boolean highCalories(Dish dish) {
        return dish.getCalories() > 300;
    }
```

데이터 소스 : 요리 리스트 (연속된 요소를 스트림에 제공)
데이터 처리 연산 : filter, map, limit
위의 코드에서 Collect를 제외한 모든 연산은 서로 파이프라인을 형성할 수 있도록 스트림을 반환한다. (Collect는 파이프라인을 처리해서 List로 반환) 또한 스트림 API는 파이프라인을 더 최적화 할 수 있도록 유연성을 제공하는 것을 확인할 수 있다.

### 스트림과 컬렉션
---
컬렉션과 스트림은 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다는 공통점이 있다. 여기서 연속된 데이터는 순서와 상관없이 아무 값에나 접속하는 것이 아니라 순차적으로 값에 접근한다는 것을 의미한다. 하지만 차이점이 존재하는데 데이터를 언제 계산하느냐가 가장 큰 차이다. 
컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조이지만, 스트림은 이론적으로 요청할 때만 요소를 계산하는 고정된 자료구조이다. (스트림은 사용자가 요청하는 값만 스트림에서 추출하며, 스트림은 딱 한번만 탐색할 수 있다.)

```java
	// 스트림은 딱 한번만 탐색이 가능하다.
    List<String> names = Arrays.asList("Java8", "Lambdas", "In", "Action");
    Stream<String> s = names.stream();
    s.forEach(System.out::println);
    // 스트림은 한 번 만 소비할 수 있으므로 아래 행의 주석을 제거하면 IllegalStateException이 발생
    s.forEach(System.out::println);
```

- 외부 반복과 내부 반복

컬렉션 인터페이스를 사용하려면 사용자가 직접 요소를 반복해야하는데 이를 **외부 반복**이라고 한다. 반면 스트림 라이브러리는 **내부 반복**을 사용한다.

```java
// 외부 반복 (for-each를 활용)
List<String> names = new ArrayList<>();
for(Dish dish : menu) {
	names.add(dis.getName());
}

// 내부 반복
List<String> names = menu.stream()
	.map(Dish::getName)
    .collect(toList());
```

> 내부 반복과 외부반복의 차이

내부 반복을 이용하면 작업을 더 투명하게 병렬로 처리할 수 있으며 더 최적화된 다양한 순서로 처리할 수 있다. (병렬성 구현이 자동으로 된다.)

![](https://velog.velcdn.com/images/bw1611/post/8ffaffc6-eeec-4d53-9553-24e1071828bc/image.png)

