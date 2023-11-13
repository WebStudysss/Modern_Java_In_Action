### lamda란 무엇인가?
---
java8에서 추가 된 기능으로 코드를 좀 더 깔끔하게 구현할 수 있다. 람다 표현식은 익명 클래스처럼 이름이 없는 함수면서 메서드를 인수로 전달할 수 있으므로 일단은 람다 표현식이 익명 클래스와 비슷하다고 생각해놓자.

- 람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단수화한 것이라고 할 수 있다.

### lamda
---

**1, lamda의 특징**
- 익명 : 보통 메서드와 달리 이름이 없으므로 익명이라 함
- 함수 : 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 한다. 하지만 메서드 처럼 동작하고 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함
- 전달 : 메서드 인수를 전달 또는 변수로 저장 가능
- 간결성 : 코드가 짧기 때문에 가독성이 좋음

그렇다면 람다는 얼마나 코드를 가독성 좋게 할 수 있는지 아래코드를 살펴보자!

```java
        String[] arrDec = {"30", "4", "3", "8", "10"};
        
        // 람다 표현식
        Arrays.sort(arrDec, (e1, e2) -> (e2 + e1).compareTo(e1 + e2)); // 8, 4, 3, 30, 10
		
        // 익명 클래스
        Arrays.sort(arrDec, new Comparator<String>(){
            @Override
            public int compare(String s1, String s2){
                return (s2 + s1).compareTo(s1 + s2);
            }
        });
```

확실히 둘다 String 배열을 내림차순 해주는 코드지만 람다 표현식이 가독성이 더 좋은 것을 확인할 수 있다.

여기서 `(e1, e2)` 는 "람다 파라미터"이며, `(e2 + e1).compareTo(e1 + e2));`이 부분이 람다 바디라고 할 수 있다. 화살표를 통해 파라미터와 바디를 구분한다고 보면 된다.
> 람다 표현식에는 return이 포함되어 있으므로 따로 return을 명시해주지 않아도 된다. 하지만 코드블록 { } 이 있다면 return 사용을 허용한다. (여기서의 return은 흐름제어문이다.)
![](https://velog.velcdn.com/images/bw1611/post/e123b377-99f4-4446-aa1d-d7b266a619ee/image.png)


그렇다면 람다는 어디에 사용하는것이 적합할까? 함수형 인터페이스라는 문맥에서 람다 표현식을 사용하기에 좋다. 함수형 인터페이스에 대해서 모르겠다면 [함수형 인터페이스란?](https://velog.io/@bw1611/%ED%95%A8%EC%88%98%ED%98%95-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%9D%B4%EB%9E%80) 을 참고해보자!

### 함수형 디스크립터
---
람다 표현식의 시그니처를 서술하는 메서드를 함수 디스크립터라고 부른다.
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
여기서 run은 인수와 반환값이 없으므로 인터페이스는 인수와 반환값이 없는 시그니처이다. 람다 포현식은 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있으며, 함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다는 사실을 기억하자.

```java
public class RunablePrac {
    public static void main(String[] args) {
        new solution().process(() -> System.out.println("this is me"));
    }
}

class solution{
    public static void process(Runnable r){
        r.run();
    }
}
```

`() -> System.out.println("this is me")`은 인수가 없으면 void를 반환하는 람다 표현식이며, 이는 run 메서드의 시그니처와 같다.


### 실행 어라운드 패턴
---
파일처리는 자원을 열고(Open) 자원을 처리(Process)하고 자원을 닫는(Close)과정을 반복한다. 이를 순환패턴이라 하는데, 그렇다면 파일 처리하는 코드는 자원을 설정(Setup)하고 자원을 처리(Process)하고 자원을 정리(CleanUp)하는데 이를 실행 어라운드 패턴이라고 부른다.

```java
    public static String processFileLimited() throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader("data.txt))) {
            return br.readLine();
        }
    }
```

![](https://velog.velcdn.com/images/bw1611/post/9eb9cb85-46d9-4d0c-99fe-83450deddd81/image.png)

람다 표현식을 사용하는 핵심 아이디어는 동작 파라미터화다. 람다 자체가 구현 로직의 표현이라는 점을 잊지말자. 람다를 이용해 핵심 로직을 담당하는 메서드를 동작 파라미터화하면 메서드가 동작을 인수로 받아 처리할 수 있있다. 로직이 변경될 때마다 새로운 클래스나 외부 메서드를 생성하지 않아도 되므로, 불필요한 코드 생산을 방지할 수 있다.

**동작 파라미터화**
그러면 위에 Java 코드를 설정, 정리 과정은 재사용하고 processFile 동작을 파라미터화해서 BufferedReader를 이용해서 다른 동작을 수행할 수 있도록 processFile 메서드로 동작을 전달해야 한다.

```java
        String result = processFile((BufferedReader b) -> b.readLine() + b.readLine());
        System.out.println(twoLines);
```

위와 같이 processFile 메서드에 동작을 전달하여 코드를 두번 출력하게 할 수 있다.

**함수형 인터페이스를 이용해서 동작 전달**
함수형 인터페이스 자리에 람다를 사용할 수 있다. 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달하고 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리 가능하다. processFile 바디 내에서 BufferedReaderProcessor의 process를 호출 가능한 것을 볼 수 있다.

```java
    public static String processFile(BufferedReaderProcessor p) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(FILE))) {
            return p.process(br);
        }
    }

    @FunctionalInterface
    public interface BufferedReaderProcessor {

        String process(BufferedReader b) throws IOException;

    }
```


**람다 전달**
람다를 이용해서 다양한 동작을 processFile 메서드로 전달할 수 있다.
```java
        String result1 = processFile(o -> o.readLine());

        String result2 = processFile((BufferedReader b) -> b.readLine() + b.readLine());
```

![](https://velog.velcdn.com/images/bw1611/post/cc9ec6dd-630f-470a-97c1-f82cd7bdbd4a/image.png)


### 다양한 함수형 인터페이스
---
- **Predicate**

`Predicate<T>`는 test라는 추상 메서드를 정의하며 제네릭 형식 T의 객체를 인수로 받아 boolean으로 반환한다.

```java
public class PredicatePrac {

    public static <T> List<T> filter(List<T> list, Predicate<T> p){
        List<T> result = new ArrayList<>();
        for (T t : list){
            if (p.test(t)){
                result.add(t);
            }
        }
        return result;
    }

    public static void main(String[] args) {
        Predicate<String> nonEmptyStringPredicate = (String s) -> s.equals("hello");
        List<String> listOfStrings = new ArrayList<>();
        listOfStrings.add("hi");
        listOfStrings.add("hello");
        listOfStrings.add("");

        List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
        System.out.println(nonEmpty);
    }
}
```

결과
```
[hello]
```

s = hello 인 문자를 전달하여 listOfStrings에 hello 문자가 일치한다면 result에 추가해주어 리턴해준다. predicate는 주어진 문자가 일치하는지 판단하여 true, false값을 반환하는 역할을 한다.


- **Consumer**

제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다.

```java
public class ConsumerPrac {
    public <T> void forEach(List<T> list, Consumer<T> c){
        for (T t : list){
            c.accept(t);;
        }
    }

    public static void main(String[] args) {
        new ConsumerPrac().forEach(
                Arrays.asList(1, 2, 3, 4, 5),
                (Integer i) -> System.out.println(i)
        );
    }
}
```

결과
```
1
2
3
4
5
```

- **Function**
제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 추상 메서드 apply를 정의한다.

```java
public class FunctionPrac {
    public <T, R> List<R> map(List<T> list, Function<T, R> f){
        List<R> result = new ArrayList<>();
        for (T t : list){
            result.add(f.apply(t));
        }

        return result;
    }

    public static void main(String[] args) {
        List<Integer> arrList = new FunctionPrac().map(
                Arrays.asList("lambdas", "in", "action"),
                (String s) -> s.length()
        );

        System.out.println(arrList);
    }
}
```

결과
```
[7, 2, 6]
```

String을 인수로 받아 s.length()를 통해 s의 길이를 리턴해준다. apply 메서드로 입력값을 받아 출력 값을 계산한다. (T를 입력받아 출력 R을 생성하는 함수를 나타낸다.)

> 제네릭이란?
`List<T> list` 여기서 T를 제네릭 타입이라고 하며 클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법이다. (제네릭 타입에는 래퍼런스 타입밖에 들어가지 못한다.)

- 함수형 인터페이스의 종류

![](https://velog.velcdn.com/images/bw1611/post/be92fdef-7959-41b9-8ccd-ec0eba8fe46b/image.png)

### 형식 검사, 형식 추론, 제약
---

- **형식 검사**

람다가 사용되는 컨텍스트를 이용해서 람다의 형식(type)을 추론할 수 있으며, 어떤 컨텍스트에서 기대되는 람다 표현식의 형식을 대상 형식(target type)이라고 부른다.
(대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다. 결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.)

```java
List<Apple> heavierThan150g = filter(inventory, (apple a) -> a.getWeight() > 150);

//형식을 추론하지 않음
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

//형식을 추론함
Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

1, filter 메서드 선언 확인
2, filter 메서드 두 번째 파라미터 `Predicate<Apple>` 형식(대상 형식) 기대
3, `Predicate<apple>`은 test라는 한 개의 추상 메서드 정의하는 함수형 인터페이스
4, test 메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터를 묘사
5, filter 메서드로 전달된 인수는 이와 같은 요구사항을 만족해야 한다.

![](https://velog.velcdn.com/images/bw1611/post/5304e38a-5882-448a-b6cc-d97d3e24ece5/image.png)

Apple을 인수로 받아 boolean을 반환하므로 유요한 코드이다. 또한 대상 형식이라는 특징 때문에 같은 람다 표현식이어도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.
예를 들어 Callable과 PrivilegedAction 인터페이스는 인수를 받지 않고 제네릭 형식 T를 반환하는 함수를 정의한다. 따라서 다음 할당문은 모두 유효하다. (이럴 경우 사용할 것으로 캐스팅을 해주면 좋다.)

```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

- **지역 변수 사용**

람다 표현식에서는 익명 함수가 하는 것처럼 자유 변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다. 이를 람다 캡처링이라고 부르기도 한다.

```java
int portNumber = 137; // 지역 변수
Runnable r = () -> System.out.println(portNumber);
```

하지만 자유 변수에는 제약이 있다. 인스턴스 변수와 정적 변수를 자유롭게 캡처할 수 있지만, 지역 변수는 명시적으로 final로 선언되어 있어야 하거나 final 선언된 변수와 똑같이 명시되어야 한다.

```java
public class ExceptionPrac {
    static int port = 3333;
    int goPort = 5353;

    public void main() {
        int portNumber = 137; // 지역 변수
        Runnable r = () -> System.out.println(goPort);
        portNumber = 33333; // 두번 할당
        port = 335353;
        goPort = 533353;
    }
}
```
위의 코드는 지역변수만 두번 할당 되면 컴파일오류가 발생하게 된다. 왜 그렇다면 지역변수만 오류가 생길까? 우선 인스턴스 변수는 heap영역 지역 변수는 stack영역에 위치한다. 람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면, 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다. (스택 영역은 스레드마다 가지고 있다.)
따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생겼다.

### 메서드 참조
---
람다 표현식을 축약한 것이라고 생각하면 편하며, 메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전당할 수 있다.
```java
		// 람다 형식
        List<Apple> apples = inventory.stream()
                .filter(apple -> Filter.isGreenApple(apple)).toList();
        // 메서드 참조        
        List<Apple> apples = inventory.stream()
                .filter(Filter::isGreenApple).toList();
```

이와 같이 메서드 참조가 람다형식보다 가독성이 좋을 경우가 있다.
![](https://velog.velcdn.com/images/bw1611/post/8d6bd6a1-964f-4e46-a2e9-ecfa11497264/image.png)

람다를 메서드 참조로 바꾸는 방법을 봐보자.

![](https://velog.velcdn.com/images/bw1611/post/216e6c9c-313e-4c1b-b47b-309eb1fedc08/image.png)

한가지 예를 확인해보자!

```java
List<String> str = Arrays.asList("a", "b", "A", "B");
str.sort((s1, s2) -> s1.compareToIgnoreCase(s2));

// 메서드 참조
str.sort(String::compareToIgnoreCase);
```

- **생성자 참조**
클래스 명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다. Class::new 와 같이 말이다.

```java
// 람다
Supplier<Apple> c1 = () -> new Apple();

// 클래스 참조
Supplier<Apple> c1 = Apple::new;

/**
 * 시그니처를 갖는 생성자는 Function 인터페이스의 시그니처와 같다.
 */
// 람다
Function<Integer,Apple> c2 = (weight) -> new Apple(weight);

// 클래스 참조
Function<Integer,Apple> c2 = Apple::new;
```

또한 인스턴스화 하지 않고 생성장에 접근할 수 있는 기능을 다양한 상황에 응용할 수 있다. 예를 들어 map으로 생성자와 문자열값을 관련시킬 수 있다. (이미지로 대체)

![](https://velog.velcdn.com/images/bw1611/post/dd3a410c-a54d-4014-97ff-90ba3e78196a/image.png)

### 람다와 메서드 참조 활용하기
---
그럼 지금까지 배운 것을 토대로 chapter1~2에서 했던 Apple이라는 클래스를 바까보자. 기존에는 람다로 되어있었기에 조금은 가독성이 떨어지는 코드로 구성되어 있었다.

1, 코드 전달 - 동작 파라미터화

```java
	// sort를 동작 파라미터화
	inventory.sort(new AppleComparator());

    static class AppleComparator implements Comparator<Apple> {

        @Override
        public int compare(Apple a1, Apple a2) {
            return a1.getWeight() - a2.getWeight();
        }

    }
```

2, 익명 클래스
```java
        inventory.sort(new Comparator<Apple>() {

            @Override
            public int compare(Apple a1, Apple a2) {
                return a1.getWeight() - a2.getWeight();
            }
        });
```

익명 클래스를 이용하여 위에 코드를 좀 더 간결하게 수정하였다.

3, 람다 표현식
```java
inventory.sort((a1, a2) -> a1.getWeight() - a2.getWeight());
```

람다 표현식을 이용하여 4줄이 넘는 코드를 1줄로 줄일 수 있는 것을 다시 한번 더 확인할 수 있다. (코드 전달, 함수형 인터페이스를 기대하는 곳 어디에서나 람다 표현식 가능하며, 추상 메서드의 시그니처(함수 디스크립터)는 람다 표현식의 시그니처를 정의)

4, 메서드 참조
```java
inventory.sort(comparing(Apple::getWeight));
```

많은 방식을 거쳐 최적의 코드까지 왔다. 처음 코드를 보고 이 메서드 참조의 코드를 보면 얼마나 코드가 가독성 좋게 바뀌었으며 직접 코드를 힘들게 작성안해도 되는지 알 수 있다. 이것만 보더라도 메서드 참조는 참 훌륭한 기능이다.

### 람다 표현식을 조합할 수 있는 유용한 메서드
---
두개의 predicate를 조합하여 커다란 predicate를만들 수 있다. 또한 한 함수의 결과가 다른 함수의 입력이 되도록 두 함수를 조합할 수도 있다. 어떻게 함수형 인터페이스에서는 이런 기능을 제공하는 것일까? **"디폴트 메서드"** 를 공부하면 이 개념을 알 수 있다.

```java
inventory.sort(comparing(Apple::getWeight).reversed());
```

다른 Comparator 인스턴스를 만들어서 역정렬을 해줄 수 있겟지만, 여기서 reverse를 사용하여 할 수 있는데 reverse는 default method이다.

![](https://velog.velcdn.com/images/bw1611/post/891de39e-77d0-4da5-bb8e-dc04e9b23960/image.png)

근데 만약에 무게가 같다면 어떻게 정렬해야할까?
```java
inventory.sort(comparing(Apple::getWeight)
	.reversed()
    .thenComparing(Apple::getColor));
```

위와 같이 무게가 같다면 색을 추가하여 무게가 같을 때 색으로 정렬을 해줄 수 있다.

- **Predicate의 default method**

negate : Predicate의 결과가 true이면 false로 반대로 false면 true로 반환

```java
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    
    //예시
    Predicate<String> nonEmptyStringPredicate2 = (String s) -> s.equals("hello");
    Predicate<String> notHelloPredicate = nonEmptyStringPredicate2.negate();
```

and : 빨간색이면서 150이상인 사과 (if문의 &&와 같다고 생각한다.)
```java
Predicate<Apple> redAnd150Apple = redApple.and(apple -> apple.getWeight() > 150);
```

or : 빨간색이거나 150이상인 사과 (if 문의 ||와 같다.)
```java
Predicate<Apple> redAnd150Apple = redApple.or(apple -> apple.getWeight() > 150);
```

- **Function의 default Method**

**andThen과 compose**

```java
public class FunctionAndThenComapose {
    public static void main(String[] args) {
        Function<Integer, Integer> f = x -> x + 1; // 1
        Function<Integer, Integer> g = x -> x * 2; // 2
        Function<Integer, Integer> h = f.andThen(g);
        int result = h.apply(1);
        System.out.println(result);

        Function<Integer, Integer> f1 = x -> x + 1; // 2
        Function<Integer, Integer> g1 = x -> x * 2; // 1
        Function<Integer, Integer> h1 = f1.compose(g1);
        int result1 = h1.apply(1);
        System.out.println(result1);
    }
}
```
