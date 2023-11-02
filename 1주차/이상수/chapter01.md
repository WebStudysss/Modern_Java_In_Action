# chapter01 자바 8, 9, 10, 11 무슨 일이 일어나고 있는가?

## 1.1 자바 8 버전 이후의 자바

### 자바8의 코드 간략화

```java
<Java8 이전>
Collections.sort(inventory, new Comparator<Apple>(){
		public int compare(Apple a1, Apple a2) {
				return a1.getWeight().compareTo(a2.getWeight());	
		}
});

->

<Java8>
inventory.sort(comparing(Apple::getWeight));
```

위의 코드 예시와 같이 자바8에서는 비교 코드를 더욱 간략하게 표현이 가능해졌다.

자바언어의 가장 큰 변화를 가져왔던 자바 8이 들어오면서 바뀐 변화에 대해서 살펴볼 것이다.

- **멀티 코어를 활용한 연산 처리를 활용하기 쉬워졌다.**
    
    자바8 이전 버전에서는 싱글 코어가 아닌 방법을 활용하려면 스레드를 활용해서 병렬 실행 환경을 관리하는 형태였는데, 이는 구현부터 관리까지 비용이 많이 들었다. 이를 쉽게 해결하기 위한 방식을 소개해준다.
    
- **우리가 배울 자바8은 큰 틀로 보았을 때 세가지 기능이 추가되었다.**
    - 스트림 API(Stream API)
    - 메서드에 코드를 전달하는 기법
    - 인터페이스의 디폴트 메서드

또 기존 버전과의 큰 변화점을 설명하자면

**첫째**, 병렬 연산이 쉬워졌다.

병렬연산을 지원하는 Stream API가 있는데 이를 활용하면 기존의 멀티코어 환경에서 사용하는 synchronized 키워드를 사용하지 않아도 된다.

또 Stream API에서 메서드에 코드를 전달하는 기법과 인터페이스의 디폴트 메서드를 전달해 줄 수 있을 것이다.

**둘째**, 코드가 간결해졌다.

메서드에 코드를 전달하는 기법이 생기면서(메서드 참조, 람다)

앞의 예시에서 보았던 짧은 코드가 된 것이다.

## 1.2  왜 아직도 자바는 변화하는가

많은 언어들이 등장하고 사라지는 와중에 자바 언어는 처음부터 객체지향의 특성을 살리고, 스레드와 락을 이용한 동시성도 지원을 했었다. 이로 인해 자바는

대중적인 언어로 발전했고, 빅데이터 트랜드가 생긴 이후로는 

빅 데이터 처리를 할 때 효율적인 방법인 병렬 프로세싱이 중요해졌는데, 자바는 병렬에는 약했었다. 이와 같이 변화하는 트랜드에 맞춰서 개발 언어도 발전한 것이다.

### 자바 8버전의 프로그래밍 개념

**첫번째 개념** - 스트림 처리

이는 한번에 하나의 항목을 처리하던 기존 방식이 아닌, 스트림 파이프라인을 통해 값을 처리할 때 여러 cpu에 할당하여 병렬 처리를 돕기도 하고, 스트림을 통해 한 데이터객체에 대한 동작처리를 다양하게 처리할 수 있다.

**두번째 개념** - 동작 파라미터화

객체의 동작을 행할 때 동작 안에 다른 동작을 넣는 방법이라고 생각하면 된다. 객체 안의 메서드에서 다른 메서드를 같이 실행하는 것이다.

8버전 전에는 이러한 방식을 구현하려면 동작을 구현한 객체를 생성해서 해당 객체를 파라미터로 넣었어야 했다.

**세번째 개념** - 병렬성

성능에 악영향을 끼치는 synchronized 키워드를 사용하지 않고도 병렬처리가 가능한데, 이 이유는 stream은 공유되지 않는 가변 데이터를 통해 처리가 이루어지기 때문에 해당 스트림이 진행되는 동안은 값이 변하지 않고 처리 마지막에 값이 변한다.

## 1.3 자바 함수

프로그래밍 언어에서는 값을 바꾸며, 전달하는 것이 핵심 가치인데, 이 핵심 가치를 이룰 수 있는 것에 `일급` 이라는 말이 붙는다.

이전의 버전에서는 메서드나 클래스 등이 일급 객체가 될 수 없었는데, 클래스나 메서드 등 자체의 값을 전달할 수 있는 값이 되도록 `일급`화 시키는게 가능해졌다.

### 메서드와 람다를 `일급`으로

```java
<Java8 이전>
File[] hiddenFiles = new File(".").listFiles(new FileFilter(){
		public boolean accept(File file){
				return file.isHidden();
		}

}

->

<Java8>
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```

이러한 변경에서 사용한 기법을 메서드참조라고 하는데 File객체에 있는 isHidden()메서드가 구현되어있다면 위와 같이 불러올 수 있는 것이다.

이전의 자바 버전에서는 ::형태를 사용하지 못했기 때문에 이급 메서드였다.

하지만 버전이 바뀌고 8버전부터는 메서드참조가 가능하기 때문에 일급 메서드로 바뀌었다.

장점

- 코드의 간결성
- 직관적

### 람다(익명 함수)

람다도 값으로 취급할 수 있는데

```java
(int x) -> x + 1
```

의 값이 있다면 ‘x라는 인수로 호출하면 x+1를 반환하라’라는 명령을 가진 함수를 만들 수 있는 것이다.

이런 람다 문법으로 구현된 프로그램을 함수형 프로그래밍, 즉 ‘함수를 일급값으로 넘겨주는 프로그램을 구현한다.’라고 한다.

### 코드 넘겨주기

전체 범위에서 특정 항목을 선택해서 반환하는 동작을 **필터**라고 하는데 이런 필터를 자바 8 이전에는 상당히 긴 코드로 구현되었을 것이다.

필터를 새로 생성하면 또 새로 메서드를 구현해야할 것이다. 비슷한 코드니 Copy&Paste를 할텐데 이렇게 구현을 지속하게되면 코드를 수정해야할 때 구현한 모든 메서드 내부를 수정하거나 할텐데 이는 상당히 비효율적이다.

자바 8버전 에서는 반복을 제거하면서 가독성까지 챙길 수 있는데 예시를 보면서 이해해보자

```java
<Java8 이전>
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if ("green".equals(apple.getColor())) {
        result.add(apple);
      }
    }
    return result;
  }

  public static List<Apple> filterHeavyApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getWeight() > 150) {
        result.add(apple);
      }
    }
    return result;
  }

->

<Java8 이후>
public static boolean isGreenApple(Apple apple) {
    return "green".equals(apple.getColor());
  }

  public static boolean isHeavyApple(Apple apple) {
    return apple.getWeight() > 150;
  }

  public static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (p.test(apple)) {
        result.add(apple);
      }
    }
    return result;
  }
```

이 코드에서 다른 점은 

```java
if ("green".equals(apple.getColor())) {
        result.add(apple);
      }

if (apple.getWeight() > 150) {
        result.add(apple);
      }
->

if (p.test(apple)) {
        result.add(apple);
      }
```

이 두곳이 주요한데, Predicate(함수형 인터페이스) p를 상속받은 filterApple이 되었고 해당 p에 있는 test를 실행할 뿐이다.

이를 호출할 때는 filterApples(inventory, Apple::isGreenApple);

형식으로 호출하는데, 여기서 알게 된 사실은 자바8에서는 이와 같이 메서드동작 자체를 전달할 수 있다는 것을 알 수 있게 되었다.

가장 대표적인 함수형 인터페이스 4가지

<aside>
💡 자바에서 지원하는 함수형 인터페이스로는 대표적으로 크게 네가지가 있다.

- Supplier<T> - 공급자 : 매개변수는 없고 반환값만 있다
    - get()
- Consumer<T> - 사용자 : 매개변수는 있고, 반환값이 없다
    - accept()
- Function<T, R> - 함수 : 일반적인 함수형태, 하나의 매개변수를 받아 결과를 반환
    - apply()
- Predicate<T> - 매개변수 하나를 받아서 boolean타입으로 반환한다
    - test()

T는 제네릭타입을 뜻하고, R은 리턴타입을 뜻한다.

추가로 Bi가 붙은 형태가 있는데 다른 방법은 똑같고 매개변수가 두개 들어간다는 의미이다.

</aside>

### 메서드 전달에서 람다로

아까 위에서 구현했던 함수형 인터페이스의 자리에 메서드가 아니라 람다로 익명 메서드를 사용해도 동일하게 동작할 수 있다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (p.test(apple)) {
        result.add(apple);
      }
    }
    return result;
  }
```

<aside>
💡 filterApples(inventory, (Apple a) → GREEN.equals(a.getColor()));

filterApples(inventory, () → GREEN.equals(Apple::getColor));

filterApples(inventory, () → Apple::Weight() < 80);

…

</aside>

람다의 코드가 너무 길다면 메서드참조를 하는 것이 코드의 명확성을 위해 더 좋을 것이다.

## 1.4 스트림

거의 모든 자바 애플리케이션을 컬렉션을 만들고 활용한다. 이 컬렉션을 통해서 값을 추출하고 필터하고, 등등 여러 작업을 시행하는데 이 동작을 코드로 작성하려면 자바8 이전에는 많은 코드 구현이 필요했다.

자바8부터는 스트림API가 도입되면서 해당 코드들을 전부 돌면서 데이터를 확인해야하는 for문 제거가 가능하고, 또 라이브러리 내부에서 모든 데이터가 처리되기 때문에 기존의 방식보다 더욱 간편하고 흐름을 읽기가 쉬운 코드로 변경될 것이다.

또 api자체에서 지원하는 병렬처리도 손쉽게 가져올 수 있을 것이다.

### 멀티스레딩은 어렵다

자바8이전의 멀티스레딩 환경은 구현부터 어려움이 있었는데, 스트림API에서 지원하는 병렬처리로 손쉽게 멀티스레딩 환경을 구축할 수 있게 되었다.

## 1.5 디폴트 메서드와 자바 모듈

자바8 이전의 List 인터페이스는 List의 스트림을 처음에는 지원하지 않았기 때문에 .sort라는 코드가 없었는데, 이를 자바8에서는 어떻게 구현을 했을까 한다면 

```java
default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
				a.sort();
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/063bb2d1-ee4b-4cdf-a8cb-3612352a6b48/3cb7b415-d52a-4c7b-b669-9289b9a4e4f3/Untitled.png)

인터페이스에 sort메서드를 추가하는 방법도 있겠지만, 이를 넣는다면 List인터페이스를 상속하는 모든 코드에 sort의 세부 구현을 우리가 해야한다.

이를 방지하기 위해서 자바8 개발자는 default메서드를 인터페이스에도 넣을 수 있게 변경이 되었고, 이 인터페이스 변경과 함께 a.sort()같은 형태로도 구현이 가능하게 되었다.

---

- ***WIMP***는 windows, icons, menus, pointer
