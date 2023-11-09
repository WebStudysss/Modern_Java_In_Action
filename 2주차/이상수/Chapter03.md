# Chapter03 람다 표현식

## 목차

- 람다란 무엇인가?
- 어디에, 어떻게 람다를 사용하는가?
- 실행 어라운드 패턴
- 함수형 인터페이스, 형식 추론
- 메서드 참조
- 람다 만들기

---

## 람다란 무엇인가?

### 람다의 특징

- 익명
    - 보통의 메서드와 달리 이름이 없으므로 `익명` 이라 표현한다.
- 함수
    - 특정 클래스에 종속되지 않기 때문에 `함수`라고 부른다. But 메서드의 특성처럼 파라미터 리스트,  바디, 반환 형식, 예외 리스트 등을 포함한다.
- 전달
    - 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
- 간결성
    - 익명 클래스처럼 불필요한 코드를 생성할 필요가 없다.

```java
<익명 클래스 활용>
Comparator<Apple> byWeight = new Comparator<Apple>(){
		public int compare(Apple a1, Apple a2) {
				return a1.getWeight().compareTo(a2.getWeight());
		}
}

->

<람다 활용>
Comparator<Apple> byWeight =
		(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

- 파라미터 리스트
    - ()안의 파라미터 개수
- 화살표
    - 람다의 파라미터와 바디의 구분자
- 람다 바디
    - 두 사과의 무게를 비교

예제를 통한 람다 표현식

```java
1. (String s) → s.length() 

2. (Apple a) → a.getWeight() > 150

3. (int x, int y) → {
		System.out.println("Result:");
		System.out.println(x + y);
}

4. () -> 42

5. (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

## 어디에, 어떻게 람다를 사용할까?

### 함수형 인터페이스

`메서드를 파라미터화 할 수 있는 인터페이스`

함수형 인터페이스의 특징으로는 `추상 메서드를 하나`만 선언하는 것이다.

### 함수형 인터페이스 활용

2장에서 구현했던 필터 메서드

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

람다를 통한 filter사용

```java
List<Apple> greenApples
		= filter(inventory, (Apple a) -> GREEN.equals(a.getColor()));
```

Predicate의 자리에 람다가 들어간다.

> 그래서 함수형 인터페이스 이야기가 왜 나오는데?
> 

람다가 인터페이스의 추상 메서드를 구현할 수 있는 추상 메서드의 구현체(디스크립터)가 될 수 있다.

### 함수 디스크립터

뜻 그대로 함수를 설명하는 것이라 생각하면 된다.

함수 Runnable의 run이 있을 때

파라미터 : null 반환 : void 으로 했을 때 해당 형태만 일치한다면, 해당 함수의 `내부를 어떻게 사용할지 람다로 설명을 하는 것이다.`

이 때 파라미터와 반환이 같은 것을 `같은 시그니처`를 갖는 함수라 볼 수 있는데 시그니처만 같게 한다면 람다를 생성할 수 있다는 뜻이다.

**일치하는 시그니처**

```java
public Callable<String> fetch() {
		return () -> "Tricky example ;-)";
}
```

파라미터 : x

반환 : String

**일치하지 않는 시그니처**

```java
Predicate<Apple> p = (Apple a) -> a.getWeight();
```

파라미터 : Apple

반환 : boolean

### @FunctionalInterface

함수형 인터페이스를 구현할 때 해당 어노테이션을 인터페이스 위에 붙였을 때 함수형 인터페이스가 불가하면 컴파일 에러를 발생시킨다.

ex) 추상메서드 2개

## 람다 활용 : 실행 어라운드 패턴

```java
public static String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(FILE))) {
        return p.process(br);
    }
}
```

## 함수형 인터페이스 사용

함수형 인터페이스의 추상 메서드 시그니처를 함수 디스크립터라고 한다.

람다 표현식을 사용하려면 함수 디스크립터를 기술하는 함수형 인터페이스  집합이 필요하다.

이를 위해 자바에서는 java.util.function 패키지로 여러 함수형 인터페이스를 제공한다.

- Predicate<T> T → boolean
- Consumer<T> → void
- Function<T, R> T → R
- Suplier<T> → T
- UnaryOperator<T> → T

### 기본형 특화

기본형의 경우 함수형 인터페이스 특성 상 제네릭을 받아서 디스크립터가 구성되는데, 이를 행하기 위해 박싱, 언박싱 과정이 불필요하게 들어가게된다.

때문에 이러한 불필요한 연산을 피할 수 있도록 돕는 IntPredicate, IntFunction 등 각 인터페이스에 맞게 지원한다.

## 형식 검사, 형식 추론, 제약

### 형식 검사

```java
List<Apple> heavierThan150g =
filter(inventory, (Apple, apple) -> apple.getWeight() > 150);

filter(List<Apple> list, Predicate<Apple> p){
		...		
		p.test();
		...
}

interface Predicate<T>{
		boolean test(T t);
}
```

람다가 사용되는 콘텍스트를 이용해서 람다의 형식을 추론할 수 있다. 

1. 람다가 사용된 콘텍스트가 무엇인가?
2. 대상 형식은 Predicate<Apple>
3. Predicate<Apple> 인터페이스의 추상 메서드는 무엇인가?
4. Apple을 파라미터로 받아 boolean을 반환한다.
5. 함수 디스크립터는 Apple → boolean이기 때문에 형식 검사가 성공적으로 완료된다.

### 특별한 void 호환 규칙

람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다

`void를 반환하는 함수 디스크립터는 다른 값이 반환되어도 사용이 가능함`

### 형식 추론

```java
Comparator<Apple> c = 
		(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

->

Comparator<Apple> c = 
		(a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

컴파일러가 람다의 표현식과 관련된 함수형 인터페이스를 추론한다.

### 지역 변수 사용

람다는 자기 바깥에 있는 접근 가능한 변수에 대해서도 사용이 가능하다

이와 같은 동작을 `람다 캡처링` 이라고 부른다.

쓰면 안되는 예시

```java
<컴파일 에러>
int n = 0;
Runnable r = () -> System.out.println(n);
n++;

->

int n = 0;
do {
    int finalN = n;
    n++;
    Runnable r = () -> System.out.println(finalN);
    r.run();
} while (n != 5);
```

## 메서드 참조

기존에 있던 메서드 정의를 재활용해서 람다처럼 코드뭉치를 전달할 수 있다.

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

->

inventory.sort(comparing(Apple::getWeight));
```

![image](https://github.com/JTStudys/Modern_Java_In_Action/assets/75903442/02739de4-c6d0-488b-91a5-ab3b788cdb5e)


### 그래서 왜 사용하는데?

기존 메서드 구현을 명시적으로 참조함으로써 가독성을 높일 수 있고, 재사용도 가능하기에 효율이 좋다.

**사용가능한 메서드 참조 형식**

- 정적 메서드 참조
    - static 유틸메서드 등
    - List::of
    - 정적 팩토리 메서드
- 다양한 형식의 인스턴스 메서드 참조
    - String::length
- 기존 객체의 인스턴스 메서드 참조
    - this::getWeight

기존의 표현식을 메서드 참조로 바꾸는 예시

```java
람다 : (args) -> ClassName.staticMethod(args)

메서드 참조 : ClassName::staticMethod
```

### 생성자 참조

```java
Apple apple1 = new Apple();

Apple apple1 = Apple::new;
```

위의 메서드 참조와 형식은 같다.

### 람다 표현식을 조합할 수 있는 유용한 메서드

몇몇 함수형 인터페이스에서 지원하는 유용한 유틸리티 메서드 소개

### Comparator

- comparing() → Comparator.comparing()
- comparing().reversed() → 역정렬
- thanComparing() → 두번째 비교자를 통한 추가비교

### Predicate

- negate() → 결과반전
- and() → 비교연산 추가
- or() → 비교연산 추가

### Function

- andThen() → 함수를 입력 후 다른 함수의 입력으로 전달
- compose() → andThen()의 반대순서로 함수의 입력전달
