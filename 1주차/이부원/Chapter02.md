### 동작 파라미터화의 개념 
---
동작 파라미터화는 시시각각 변화하는 요구사항에 효과적으로 대응하기 위해 생겨난 개념이다. 즉, 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다.

그렇다면 한번 동작 파리미터화를 코드로 구현해보자!

첫번째 요구사항은 **"녹색 사과만 필요해!"**이다.

```java
    public List<Apple> filterGreenApples(List<Apple> inventory){
        List<Apple> result = new ArrayList<>();

        for (Apple apple: inventory){
            if (GREEN.equals(apple.getColor())){ // 녹색 사과를 선택하는 부분!
                result.add(apple);
            }
        }
        return result;
    }
```

현재 첫번째 요구사항을 만족했다. 하지만 바로 두번째 요구사항인 **"녹색 사과 말고, 빨간 사과가 필요해"**라는 요구사항이 들어온다면 어떻게 해야할까? 물론 그냥 `RED.equals(apple.getColor))`를 사용해도 문제가 없을 것이다. 하지만 이런 설계는 요구사항이 점점 많아진다면 코드의 길이가 길어지게 되면서 가독성이 떨어지게 된다. 이런 상황을 해결하기 위해 색을 파라미터화 시키는 것이 좋다.

- **색을 파라미터화하자**

```java
    public List<Apple> filterColorApples(List<Apple> inventory, Color color){
        List<Apple> result = new ArrayList<>();

        for (Apple apple: inventory){
            if (apple.getColor().equals(color)){
                result.add(apple);
            }
        }
        return result;
    }
    
	// 생략 부분
    List<Apple> greenApples = filterColorApples(inventory, GREEN);
    List<Apple> greenApples = filterColorApples(inventory, RED);
```

이렇게 색을 파라미터로 보내게 된다면 무슨 요구사항이 들어오든 원하는 색으로 필터링을 해줄 수 있게 된다. 즉 필요없는 코드의 중복을 줄이며, 코드를 고쳐줄 필요가 없게 되는 것이며, 아래처럼 메서드만 호출해주면 되는 것이다.

여기서 갑자기 세번째 요구사항이 추가가 됩니다. **"색 이외에도, 무게가 150이상인 사과만 가져오고 싶다"** 임시방편으로 똑같이 위와 같은 코드를 구현해줄 수 있다.

```java
    public List<Apple> filterWeightApples(List<Apple> inventory, int weight){
        List<Apple> result = new ArrayList<>();

        for (Apple apple: inventory){
            if (apple.getWeight() > weight){
                result.add(apple);
            }
        }
        return result;
    }
```

위와같이 따로 메서드를 구현해주면 된다! 하지만 딱 보기에도 색필터링과 무게 필터링이 많이 중복되어 보인다. 반복되는 코드는 가독성을 떨어트리며 불필요한 중복을 야기한다. 그러면 다음에서 모든 기능을 한 메서드에서 필터링해보자!

- **한 곳에서 모두 필터링하기**

```java
    public List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag){
        List<Apple> result = new ArrayList<>();

        for (Apple apple: inventory){
            if ((!flag && apple.getWeight() > weight) || (flag && apple.getColor().equals(color))){
                result.add(apple);
            }
        }
        return result;
    }
    
    // 생략

    List<Apple> greenApples = filterColorApples(inventory, GREEN, 0, true);
    List<Apple> greenApples = filterColorApples(inventory, null, 150, false);
```

한 개의 메서드에 다 정리하긴 했지만 코드가 막 좋다고 볼 수는 없다. boolean 값은 들어간 이유가 무엇이며 요구사항이 바뀌면 유연하게 대응할 수 조차 없다. 이런 문제를 **동작 파라미터화**를 통해 유연성을 얻어보자!


### 동작 파라미터화 구현
---
사과의 조건에 맞게 boolean값을 리턴해주는 방법이 있다. 이렇게 하면 요구사항에 좀 더 유연하게 대응할 수 있는데 한번 확인해보자!

```java
  interface ApplePredicate {

    boolean test(Apple a);

  }

  static class AppleWeightPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getWeight() > 150;
    }

  }

  static class AppleColorPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getColor() == Color.GREEN;
    }

  }

  static class AppleRedAndHeavyPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getColor() == Color.RED && apple.getWeight() > 150;
    }

  }
```

아래와 같이 상속구조가 될 것이다. 조건에 따라 filter 메서드가 동작이 달라질 수 있다. 이를 전략 디자인 패턴(strategy design pattern)이라고 한다.

![](https://velog.velcdn.com/images/bw1611/post/c3185f55-f130-4d2c-b143-c062eb36eca7/image.png)

> 전략 디자인 패턴
캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법

현재 우리의 상태에서는 ApplePredicate (패밀리), AppleWeightPredicate, AppleColorPredicate가 전략이 되는 것이다. ApplePredicate는 filterApples에서 ApplePredicate 객체를 받아 조건을 검사하도록 메서드를 수정해주도록 해보자. 이렇게하면 동작 파라미터화가 구현이되고, 다양한 동작을 받아서 내부적으로 다양한 동작을 수행하게 된다.

- **추상적 조건으로 필터링**

```java
    List<Apple> redApples = filterApplesByColor(inventory, Color.RED);
    System.out.println(redApples);

    List<Apple> greenApples2 = filter(inventory, new AppleColorPredicate());
    System.out.println(greenApples2);

    List<Apple> heavyApples = filter(inventory, new AppleWeightPredicate());
    System.out.println(heavyApples);

    List<Apple> redAndHeavyApples = filter(inventory, new AppleRedAndHeavyPredicate());
    System.out.println(redAndHeavyApples);

    List<Apple> redApples2 = filter(inventory, new ApplePredicate() {
      @Override
      public boolean test(Apple a) {
        return a.getColor() == Color.RED;
      }
    });
    System.out.println(redApples2);
  }


    public static List<Apple> filter(List<Apple> inventory, ApplePredicate p) { // 동작 파라미터화 구현
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (p.test(apple)) {
                result.add(apple);
            }
        }
        return result;
    }

  // 생략

  interface ApplePredicate {

    boolean test(Apple a);

  }

  static class AppleWeightPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getWeight() > 150;
    }

  }

  static class AppleColorPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getColor() == Color.GREEN;
    }

  }

  static class AppleRedAndHeavyPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getColor() == Color.RED && apple.getWeight() > 150;
    }

  }
```

filter 메서드가 ApplePredicate 객체를 인수 받도록 하였으며, filter 메서드 내부에서 컬렉션을 반복하는 로직과 컬렉션의 요소에 적용할 동작을 분리할 수 있다는 점에서 큰 이득을 볼 수 있다.

한번 **동작 순서**를 알아보겠습니다!

1,
```java
        List<Apple> heavyApples = filter(inventory, new AppleWeightPredicate());
```

AppleWeightPredicate()을 통해 조건을 검사하고 filter로 들어가게 된다.

2,
```java
    public static List<Apple> filter(List<Apple> inventory, ApplePredicate p) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (p.test(apple)) {
                result.add(apple);
            }
        }
        return result;
    }
```

`f (p.test(apple))`문을 통하여 AppleWeightPredicate의 test 메서드에 들어가게 된다.

3,
```java
    static class AppleWeightPredicate implements ApplePredicate {
        @Override
        public boolean test(Apple apple) {
            return apple.getWeight() > 150;
        }
    }
```

여기서 조건을 검사하고 맞는거만 filter에서 result에 담아 return 해주는 동작으로 구현이 되어있다. 이렇게 함으로써 가독성도 좋아졌으며 사용하기도 쉬워졌다. 또한 새로운 요구사항이 추가된다면 메서드를 추가하여 변화에 대응할 수 있다. 죽, 우리는 filter 메서드 동작을 파라미터화 한것이라고 볼 수 있다.
- 동작 파라미터화 그림 예시(filterApples는 우리 코드의 filter이다.)

![](https://velog.velcdn.com/images/bw1611/post/085f8bf7-c699-4313-b275-3bb0ea43b4b1/image.png)

- **한 개의 파라미터, 다양한 동작**

컬렉션의 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 동작 파라미터화의 강점이다. 한 메서드가 다른 동작을 수행하도록 재활용할 수 있다. 하지만 여러 클래스를 구현해서 인스턴스화하는 과정이 복잡하고 거추장스럽게 느낄 수 있다.
</br>

- **복잡한 과정 간소화**

자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 익명 클래스라는 기법을 제공해준다.

> 익명 클래스
자바의 지역 클래스와 비슷한 개념이며, 익명 클래스는 말 그대로 이름이 없는 클래스다. 즉석에서 필요한 구현을 만ㄷ르어서 사용할 수 있다.


- **익명 클래스 사용**

```java
        List<Apple> redApples2 = filter(inventory, new ApplePredicate() {
            @Override
            public boolean test(Apple a) {
                return a.getColor().equals(Color.RED);
            }
        });
```

코드의 길이는 줄어들긴 했지만 여전히 불필요한 코드가 많이 보인다. 결국 객체를 만들고 명시적으로 새로운 동작을 정의하는 메서드를 구현해야 한다는 점은 변하지 않았다. 그렇다면 람다 표현식을 사용해보자.
</br>

- **람다 표현식 사용**

```java
        List<Apple> result = filter(inventory, (Apple apple) -> Color.RED.equals(apple.getColor()));
        System.out.println(result);
```

람다식을 사용하면 이전보다 코드를 횔씬 간단하게 구현할 수 있으며 복잡성을 해결하고, 가독성도 좋아진 것을 확인할 수 있다. (지금까지 코드 중 가장 좋다)

- **리스트 형식으로 추상화(제네릭 활용)**

```java
    public interface Predicate<T> {

        boolean test(T t);
    }
    
        public static <T> List<T> filter(List<T> inventory, Predicate<T> p) {
        List<T> result = new ArrayList<>();
        for (T e : list) {
            if (p.test(e)) {
                result.add(e);
            }
        }
        return result;
    }
```

이렇게 제네릭을 활용하면 사과뿐만 아니라, 바나나, 배, 수박 등의 리스트에 메서드를 사용할 수 있다.


- **Comparator 정렬**

컬렉션 정렬은 반복되는 프로그래밍 작업이며, 요구사항에 쉽게 대응할 수 있는 정렬 동작을 수행할 수 있는 코드이다. Java 8부터는 List에는 sort 메서드가 포함되어 있다. 인터페이스를 갖는 java.util.Comparator 객체를 이용해 sort의 동작을 파라미터화 할 수 있다.

```java
    public interface Comparator<T>{
      int compare(T o1, T o2);
    }
```

이렇게 Comparator를 구현해서 sort 메서드의 동작을 다양하게 할 수 있다.

```java
    inventory.sort(new Comparator<Apple>() {
        public int compare(Apple a1, Apple a2) {
        	return a1.getWeight().compareTo(a2.getWeight());
        }
    });
    
    // 람다 표현식
    inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

이렇게 하면 요구사항에 맞는 Comparator를 만들어 sort 메서드에 전달할 수 있다. (람다 표현식이 더 깔끔하다.)

- **Runnable로 코드 블록 실행하기**

자바의 스레드를 이용하면 병렬로 코드 블록을 실행할 수 있다. Runnable 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있다.

```java
    public interface Runnable{
    	void run();
    }
```

Runnable을 이용해 다양한 동작을 스레드로 실행할 수 있다.

```java
    Thread t = new Thread(new Runnable() {
      public void run() {
      	System.out.println("H W");
      )
    });
    
    // 람다 표현식
    Thread t = new Thread(() -> System.out.println("H W"));
```

**참고 자료**
[모던 자바 인 액션](https://search.shopping.naver.com/book/catalog/32466988102?cat_id=50010920&frm=PBOKMOD&query=%EB%AA%A8%EB%8D%98+%EC%9E%90%EB%B0%94+%EC%9D%B8+%EC%95%A1%EC%85%98&NaPm=ct%3Dlogk2s54%7Cci%3Da23985b023f8308c9fc11c9de7f1083f41c4a019%7Ctr%3Dboknx%7Csn%3D95694%7Chk%3D3cfbba3802c6a710f02b3e36b23f3d3b425926ac)
