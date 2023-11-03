### Java 8
---
Java 8은 자바 역사상 가장 큰 변화를 맞이한 버전이라고 볼 수 있다.
자바는 병렬 실행 환경을 쉽게 관리하고 에러가 덜 발생하는 방향으로 진화하려고 노력하고 있으며. java 8에서는 병렬 실행을 새롭고 단순한 방식으로 접근할 수 있는 방법을 제공하였다.

```java
       Collections.sort(inventory, new Comparator<Apple>() {
            @Override
            public int compare(Apple a1, Apple a2) {
                return a1,getWeogtj().compareTo(a2.getWeight());
            }
        });
```

> 만약 Comparator의 개념을 잘 모른다면 [Comparator의 개념](https://velog.io/@bw1611/Comparator%EC%9D%98-%EA%B0%9C%EB%85%90)을 찾아보도록하자!

이렇게 복잡한 코드를 java 8을 이용하면 쉽게 바꿀 수 있다.

```java
iventory.sort(comparing(Apple::getWeiht));
```
사과의 무개를 비교하여 정렬하는 코드인지만 알고 넘어가도록 해보자!

### Java 8의 기능에는 무엇이 있을까?
---

- Stream API
- Method에 코드를 전달하는 기법
- Interface의 defalut Method

그렇다면 Stream을 사용하는 이유가 뭘까? 바로 이야기하자면 Stream은 병렬 연산을 지원하는 API이다. 즉, 스트림을 이용하면 에러를 자주 일이키는 멀티코어 CPU를 이용하는 것보다 비용이 횔씬 비싼 Synchronized를 사용하지 않아도 된다.

> - Synchronized는 무엇일까?
다중 스레드 환경에서 데이터 동기화를 관리하기 위한 키워드 또는 메커니즘입니다. 즉 현재 데이터를 사용하고 있는 해당 스레드를 제외하고 나머지 스레드들은 데이터에 접근 할 수 없도록 막는 개념이다.

Java 8은 병렬성을 활용하는 코드, 간결한 코드를 구현할 수 잇도록 제공해주는 개념을 공부할 수 있다.

**3가지의 개념**
- 스트림 처리
스트림이란 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임이며, `java.tuil.stream` 패키지에 Stream API를 사용할 수 있다. Stream의 핵심은 기존에는 한 번에 한 항목을 처리했지만, 이제 java 8에서는 작업을 고수준으로 추상화해서 일련의 Stream으로 만들어 처리할 수 있다. 스레드라는 복잡한 작업을 사용하지 않으면서 공짜로 병렬성을 얻을 수 있다.

- 동작 파라미터화로 메서드에 코드 전달하기
동작 파라미터화는 아직 어떻게 실행할 것인지 결정하지 않은 코드 블럭을 의마하며, 다른 말로는 행위 매개변수화 라고도 볼 수 있다. 동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응이 가능하다는 장점이 있다. (우선 동작파라미터화의 개념만 알고 넘어가도록하자!)

- 병렬성과 공유 가변 데이터
스트림 메서드로 전달하는 코드는 다른 코드와 동시에 실행하더라도 안전하게 실행 될 수 있다.

### 자바 함수
---
함수라는 용어는 메서드(특히 정적 메서드)의 의미로 사용된다. 자바에는 기본 값(int, double, String 등)과 객체라는 값이 존재하는데, 함수가 필요한 이유가 무엇일까? 프로그래밍에서 값을 바꿀 수 있는 값을 1급 객체(시민) 이라고 부른다.
> 1급 객체의 개념
- 변수나 데이터 구조 안에 담을 수 있다.
- 파라미터로 전달 할 수 있다.
- 반환값으로 사용할 수 있다.
- 할당에 사용된 이름과 무관하게 고유한 구별이 가능하다.

또한 프로그램을 실행하는 동안 이러한 모든 구조체를 자유롭게 전달할 수 없는 부분도 있다. 예를 들어, 메서드, 클래스(구조체)와 같은 부분은 프로그램을 실행하는 동안 자유롭게 보낼 수 없는데, 이렇게 전달할 수 없는 부분을 2급 객체(시민) 이라고 한다. 하지만 java8에서 이러한 2급 객체를 1급 객체로 바꿀 수 있는 기능을 추가했다.

**메서드와 람다를 1급 객체로**
java 8에서 메서드를 값으로 취급할 수 있도록 하였으며, 메서드를 값으로 취급할 수 있는 기능은 스트림 같은 다른 java 8 기능의 토대를 제공했다.

**메서드 참조**
```java
        File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
            @Override
            public boolean accept(File file) {
                return file.isHidden();
            }
        });
```

Java 8을 통해서 아래처럼 구현할 수 있다.

```java
        File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```

Java 8의 메서드 참조를 사용하면 4줄 코드를 단 1줄로 줄일 수 있다. 메서드 참조를 이용해서 listFiles에 직접 전달할 수 있는 것이다. 더 이상 메서드가 2급 객체가 아닌 1급 객체로 활용되는 것이다.

**람다 : 익명 함수**
메서드를 1급 객체로 취급할 뿐 아니라 람다를 포함하여 함수도 값으로 취급할 수 있다.
(int x) -> x + 1라는 동작을 수행하는 코드와 MyMathUtils라는 클래스를 만든 후 add1이라는 메서드를 호출하면 +1을 해주는 스트림을 만들었는데, 이 부분을 함수를 일급 객체로 넘겨주는 프로그램을 구현한 부분이다.

```java
public class Main {
    public static void main(String[] args) {
    	  // 람다 형식
//        Function<Integer, Integer> addOne = (x) -> x + 1;
//
//
//        int input = 5;
//        int result = addOne.apply(input); // 람다 함수 호출
//        System.out.println("Input: " + input);
//        System.out.println("Result: " + result);

        List<Integer> numbers = new ArrayList<>();

        for (int i = 1; i <= 5; i++){
            numbers.add(i);
        }

        List<Integer> incrementedNumbers = numbers.stream()
                .map(MyMathUtils::add1)
//                .map(x -> MyMathUtils.add1(x)) // lamda
                .collect(Collectors.toList());

        System.out.println(incrementedNumbers);
    }
}

class MyMathUtils {
    public static int add1(int x) {
        return x + 1;
    }
}
```

**메서드 넘겨주기**
```java
class Stream{
    public static void main(String[] args) {
        Apple apple1 = new Apple(160, "GREEN");
        Apple apple2 = new Apple(140, "RED");
        Apple apple3 = new Apple(170, "RED");
        Apple apple4 = new Apple(140, "GREEN");

        Filter filter = new Filter();

        List<Apple> inventory = new ArrayList<>();
        inventory.add(apple1);
        inventory.add(apple2);
        inventory.add(apple3);
        inventory.add(apple4);

        System.out.println(inventory);

//        List<Apple> greenApples = filter.filterGreenApples(inventory);
//        System.out.println(greenApples);

        // 메서드를 전달
        List<Apple> apples = inventory.stream()
                .filter(Filter::isGreenApple).toList();

        System.out.println("color : " + apples);

        // 메서드를 전달
        List<Apple> weighApples = inventory.stream()
                .filter(Filter::isHeavyApple).toList();

        System.out.println("weight : " + weighApples);


        // 람다 형식으로
        List<Apple> weighAndColor = inventory.stream()
                .filter((Apple a)-> a.getWeight() > 150 && "RED".equals(a.getColor()))
                .collect(Collectors.toList());

        System.out.println(weighAndColor);

    }
}

class Filter {
    static String GREEN = "GREEN";

    public static boolean isGreenApple(Apple apple) {
        return GREEN.equals(apple.getColor());
    }

    public static boolean isHeavyApple(Apple apple) {
        return apple.getWeight() > 150;
    }


    public List<Apple> filterGreenApples(List<Apple> inventory){
        List<Apple> result = new ArrayList<>();

        for (Apple apple: inventory){
            if (GREEN.equals(apple.getColor())){
                result.add(apple);
            }
        }
        return result;
    }

    public List<Apple> filterWeightApples(List<Apple> inventory){
        List<Apple> result = new ArrayList<>();

        for (Apple apple: inventory){
            if (apple.getWeight() > 150){
                result.add(apple);
            }
        }
        return result;
    }

}

public class Apple {

    private int weight;
    private String color;

    public Apple(int weight, String color) {
        this.weight = weight;
        this.color = color;
    }

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    @Override
    public String toString() {
        return "{ weight=" + weight + ", color='" + color + " }";
    }
}
```
Apple을 filtering하기 위하여 만든 클래스이다. 우선 Apple이라는 클래스를 만들고 Filter라는 클래스를 통해 조건에 맞는 메서드를 만들어 주었다. 

```java
    public List<Apple> filterGreenApples(List<Apple> inventory){
        List<Apple> result = new ArrayList<>();

        for (Apple apple: inventory){
            if (GREEN.equals(apple.getColor())){
                result.add(apple);
            }
        }
        return result;
    }
```

이런식으로 메서드를 만들어서 `filter.filterGreenApples(inventory)` 호출해주어 해결할 수도 있지만 불필요한 코드가 상당히 길어지게 된다. 이럴경우 메서드를 전달할 수 있는데 아래 코드를 보면 메서드를 전달해줄 수 있는 기능을 확인할 수 있다.

```java
        // 메서드를 전달
        List<Apple> apples = inventory.stream()
                .filter(Filter::isGreenApple).toList();

        System.out.println("color : " + apples);

        // 메서드를 전달
        List<Apple> weighApples = inventory.stream()
                .filter(Filter::isHeavyApple).toList();

        System.out.println("weight : " + weighApples);
```

또한 한번사용할 메서드를 정의하면 불필요한 코드의 길이가 길어지고 가독성도 떨어질 수 있다. 이럴 경우 익명 함수(람다)를 이용하여 선언과 동시에 코드를 구현할 수 있습니다.

```java
        // 람다 형식으로
        List<Apple> weighAndColor = inventory.stream()
                .filter((Apple a)-> a.getWeight() > 150 && "RED".equals(a.getColor()))
                .collect(Collectors.toList());

        System.out.println(weighAndColor);
```

하지만 만약 람다의 함수 길이가 길어지게 된다면 코드의 명확성이 떨어지기 때문에 메서드를 따로 구현하여 코드의 명확성을 우선시하는게 좋다. 또한 병렬성이 중요한 설계에서는 람다보다는 스트림을 사용하는 것이 좋다.

### 스트림 (Stream)
---
스트림 API를 사용하면 컬렉션 API와는 상당히 다른 방식으로 데이터를 처리할 수 있다. for-each 루프를 이용하는 방식은 보통 외부 반복이라고 하며, Stream API를 사용하여 내부에서 모든 데이터를 처리하는 것을 내부 반복이라고 한다.

**멀티스레딩**
멀티스레딩 환경에서 각각의 스레드는 동시에 공유된 데이터에 접근하고, 데이터를 갱신할 수 있다. 하지만 스레드를 잘 제어 못하면 이런 환경은 오히려 독이 될 수 있다. Stream API는 컬렉션을 처리하면서 발생하는 모호함과 반복적 코드 문제를 해결했으며, 멀티코어의 활용 어려움 또한 해결했다. (필터링, 추출, 그룹화)

```java
        // 순차 처리
        List<Apple> weightApples2 = inventory.stream()
                .filter((Apple a) -> a.getWeight() > 150)
                .toList();

        // 병렬 처리
        List<Apple> weightApples3 = inventory.parallelStream()
                .filter((Apple a) -> a.getWeight() > 150)
                .toList();
```

### 디폴트 메서드와 자바 모듈
---
외부에서 만들어진 컴포넌트를 이용해 시스템을 구축하는 경향이 있는데, 지금까지는 자바에서는 특별한 구조가 아닌 평범한 자바 패키지 집합을 포함하는 JAR 파일을 제공하는 것이 전부였다. 이를 해결하기 위해 자바9에서는 패키지의 모음을 포함하는 `모듈`을 정의하였다. 또한 인터페이스를 쉽게 바꿀 수 있도록 `디폴트 메서드`를 지원한다. 

```java
    @SuppressWarnings({"unchecked", "rawtypes"})
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }
```
