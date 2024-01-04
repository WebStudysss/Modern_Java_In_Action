# 11. null 대신 Optional 클래스

<aside>
📖 이 장의 내용
- null 참조의 문제점과 null을 멀리해야 하는 이유
- null 대신 Optional : null로부터 안전한 도메인 모델 재구현하기
- Optional 활용 : null 확인 코드 제거하기
- Optional에 저장된 값을 확인하는 방법
- 값이 없을 수도 있는 상황을 고려하는 프로그래밍

</aside>

---

## **11.1 값이 없는 상황을 어떻게 처리할 수 있을까?**

```java
public String getCarInsuranceName(Person person) {
	return person.getCar().getInsurance().getName();
}
```

위 코드에서 `person`이 `null`이거나 `getCar()`, `getInsurance()`가 `null`을 반환한다면 어떻게 될까?

### **11.1.1 보수적인 자세로 `NullPointerException` 줄이기**

```java
public String getCarInsuranceName(Person person) {
	if(person != null) {
		Car car = person.getCar();
		if(car != null) {
			Insurance insurance = car.getInsurance();
			if(insurance != null) {
				return insurance.getName();
				}
			}
	}
	return "Unknown";
}
```

위 코드는 변수를 참조할때마다 `null`을 확인한다. 
따라서 중첩된 `if`가 추가되면서 코드 들여쓰기 수준이 증가한다. (메모리, 시간복잡도 낭비중임)

이와같은 반복 패턴 코드를 '**깊은 의심(deep doubt)**'라 부른다. 이를 반복하다보면 코드의 구조가 엉망이 되고 가독성도 떨어진다.

```java
public String getCarInsuranceName (Person person){
            if (person == null) {
                return "Unknown";
            }
            Car car = person.getCar();
            if (car == null) {
                return "Unknown";
            }
            Insurance insurance = car.getInsurance();
            if (insurance == null) {
                return "Unknown";
            }
            return insurance.getName();
        }
```

위 코드처럼 중첩 if 블록을 없앨 수 있지만, 너무 많은 출구(`return`)이 생겼기 때문에 유지보수가 어려워지고 실수가 발생하기 쉽다.

---

### **11.1.2 `null` 때문에 발생하는 문제들**

- 에러의 근원이다 : `NullPointerException`은 자바에서 가장 흔한 에러
- 코드를 어지럽힌다 : 중첩된 `null` 확인 코드로 가독성이 저하
- 아무 의미가 없다 : `null`은 아무 의미도 표현하지 않음
- 자바 철학에 위배 : 자바는 개발자로부터 모든 포인터를 숨겼지만, `null`은 예외
- 형식 시스템에 구멍을 만든다 : `null`은 무형식이며 정보를 포함하고있지 않으므로 모든 참조 형식에 `null`을 할당할 수 있다. 이렇게 할당된 `null`이 전파되면 애초에 `null`이 어떤 의미로 사용되었는지 알수 없다.

---

### **11.1.3 다른 언어는 null 대신 무얼 사용하나?**

그루비 같은 언어에서는 안전 내비게이션 연산자(`?.`)를 도입해서 `null` 문제를 해결했다.

```java
def carInsuranceName = person?.car?.insurance?.name
```

`null`이 할당될 수 있는 값에 안전 네비게이션 연산자를 이용하면 `null` 참조 예외 걱정 없이 객체에 접근할 수 있다. 이때 `null`인 참조가 있으면 결과로 `null`이 반환된다.

하스켈, 스칼라 등의 함수형 언어는 다른 관점에서 `null` 문제에 접근한다. 하스켈의 `Maybe`와 스칼라의 `Optional`은 주어진 형식의 값을 갖거나 아무 값도 갖지 않을 수 있는 구조를 가진다.
따라서 `null` 참조 개념은 자연스럽게 사라지게 되는 것이다.

---

### **11.2 Optional 클래스 소개**

`Optional`은 선택형값을 캡슐화하는 클래스다. 
값이 있으면 `Optional` 클래스는 값을 감싸고, 값이 없으면 `Optional.empty` 메서드로 `Optional`을 반환.
또한, `Optional.empty`는 `Optional`의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드다.

`null`을 참조하면 `NullPointerException`이 발생하지만 `Optional.empty()`는 `Optional` 객체이므로 이를 다양한 방식으로 활용할 수 있다.

`Car` 변수를 그냥 사용했을때는 `null` 참조가 할당되었을때 올바른 상황인지 아니지 알수 없다. 
하지만 `Optional`을 사용하면 `Optional<Car>`로 바뀌고, 이는 값이 없을 수 있음을 명시적으로 보여준다.
또한, `Optional`을 이용하면 값이 없는 상황이 우리 데이터에 문제가 있는 것인지 아니면 알고리즘의 버그인지 명확하게 구분할 수 있다.  모든 `null` 참조를 `Optional`로 대치하는 것은 바람직하지 않고 `Optional`의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것이다.

---

### **11.3 Optional 패턴 적용**

**11.3.1 Optional 객체 만들기**

- **빈 Optional**

```java
// Optional.empty로 빈 Optional 객체를 만들 수 있다.
Optional<Car> optCar = Optional.empty();
```

- **null이 아닌 값으로 Optional 만들기**
    - 정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional을 만들 수 있다.
        
        ```java
        Optional<Car> optCar = Optional.of(car); // car가 null이라면 
        // 즉시 NullPointerException이 발생한다.
        // (Optional을 사용하지 않았다면 car 프로퍼티에 접근하려 할 때 에러가 발생했을 것이다)
        ```
        
- **null값으로 Optional 만들기**
    - 정적 팩토리 메서드 Optional.ofNullable로 null 값을 저장할 수 있는 Optional을 만들 수 있다.
        
        ```java
        Optional<Car> optCar = Optional.ofNullable(car); // car가 null이면
        // 빈 Optional 객체가 반환
        ```
        

---

### **11.3.2 맵으로 Optional의 값을 추출하고 변환하기**

```java
String name = null;
	if(insurance != null) {
	name = insurance.getName();
}
```

Optional은 map 메서드를 지원한다. 따라서, 위의 코드를 다음과 같이 `Optional`로 사용할 수 있다.

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optioanl<String> name = optInsurance.map(Insurance::getName);
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/90b9ab5b-7fca-4778-a6e1-063a32b661fc/Untitled.png)

`Optional`의 `map` 메서드는 스트림의 `map` 메서드와 개념적으로 비슷하다.
여기서 `Optional` 객체를 최대 요소의 개수가 한 개 이하인 데이터 컬렉션으로 생각할 수 있다.
`Optional`이 값을 포함하면 `map`의 인수로 제공된 함수가 값을 바꾸고, 비어있으면 아무 일도 일어나지 않는다.

---

### **11.3.3 flatMap으로 Optional 객체 연결**

```java
public class Person {
	private Optional<Car> car;

	public Optional<Car> getCar() {
		return car;
	}
}
...

Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson
	.map(Person::getCar)
	.map(car::getInsurance)
	.map(Insurance::getName);
```

위 코드에서 `optPerson`의 형식은 `Optional<Person>`이므로 `map` 메서드를 호출할 수 있다. 
하지만 `getCar`는 `Optional<Car>` 형식의 객체를 반환한다. 
즉, `map` 연산의 결과는 `Optional<Optional<Car>>` 형식의 객체가 되어 컴파일되지 않는다.

위 문제를 `flatMap`을 사용해 해결할 수 있다. 
`flatMap`은 인수로 받은 함수를 적용해서 생성된 각각의 스트림에서 콘텐츠만 평준화하여 남긴다. 다시말해 함수를 적용해서 생성된 모든 스트림이 하나의 스트림으로 병합되어 평준화된다는 것을 의미한다.

---

### **Optional로 자동차의 보험회사 이름 찾기**

```java
public String getCarInsuranceName(Optional<Person> person) {
	return = person.flatMap(Person::getCar)
		.flatMap(car::getInsurance)
		.map(Insurance::getName)
		.orElse("Unknown");
}
```

`null`을 확인하느라 조건 분기문을 추가해서 코드를 복잡하게 만들지 않으면서도 쉽게 이해할 수 있는 코드를 완성했다.

`Optional`을 인수로 받거나 `Optioanl`을 반환하는 메서드를 정의한다면 결과적으로 이 메서드가 빈 값을 받거나 빈 결과를 반환할 수 있음을 잘 문서화해서 제공하는 것과 같다.

---

### **도메인 모델에서 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유**

설계 단계에서 `Optional`은 선택형 반환값을 지원하는 것일 뿐, 
필드 형식으로 사용될 것을 가정하지 않았기 때문에 `Serializable` 인터페이스를 구현하지 않는다. 
따라서 직렬화 모델이 필요하다면 다음 예제와 같이 `Optional`로 값을 반환받을 수 있는 메서드를 추가하는 것을 권장한다.

```java
public class Person {
	private Car car;

	public Optional<Car> getCarAsOptional() {
		return Optional.ofNullable(car);
	}
}
```

---

### **11.3.4 Optional 스트림 조작**

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
	return persons.stream()
		.map(Person::getCar) //Stream<Optional<Car>>
		.map(optCar -> optCar.flatMap(Car::getInsurance)) //Stream<Optional<Insurance>>
		.map(optIns -> OptIns.map(Insurance::getName)) //Stream<Optional<String>>
		.flatMap(Optional::stream) //Stream<String>
		.collect(toSet()); //Set<String>
}
```

스트림 요소를 조작하려면 변환, 필터 등의 일련의 긴 체인이 필요한데, `Optional`로 값이 감싸있으면 이 과정이 조금 더 복잡해진다.

---

### **11.3.5 디폴트 액션과 `Optional` 언랩**

Optional 클래스는 Optional 인스턴스에 포함된 값을 읽는 다양한 방법을 제공한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/075997f4-35b2-4955-b621-060d6a108880/e2f60275-0dea-4f4c-99c8-0acc669a8d3a/Untitled.png)

- `get()`
값을 읽는 가장 간단하면서 가장 안전하지 않은 메서드이다. 래핑된 값이 있으면 해당 값을 반환하고 없으면 NoSuchElementException을 반환한다. 그냥 쓰지 않는 것이 바람직하다.
- `orElse(T other)` 
메서드를 이용하면 Optional이 값을 포함하지 않을 때 기본 값을 제공할 수 있다.
그런데 여기서 값이 있어도 orElse가 실행되므로 orElseGet을 사용하는것이 낫다.
- `orElseGet(Supplier<? extends T> other)`
`orElse` 메서드에 대응하는 게으른 버전의 메서드다. `Optional`에 값이 없을때만 `Supplier`가 실행되며, 디폴트 메서드를 만드는데 시간이 걸리거나 `Optional`이 비어있을 때만 기본값을 생성하고자 할 때 사용한다.
- `orElseThrow(Supplier<? extends X> exceptionSupplier)`
`Optional`이 비어있을때 지정한 예외를 발생시킨다. 즉, 발생 시킬 예외의 종류를 정의할 수 있다.
- `ifPresent(Consumer<? super T> consumer)`
값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다.
- `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`
`Optional`이 비어있을 때만 실행할 수 있는 `Runnable`을 인수로 받는다.

---

### **11.3.6 두 Optional 합치기**

```java
public Insurance findCheapestInsurance(Person person, Car car) {
	...
	return cheapestCompany;
}

public Optional<Insurance> nullSafeFindCheapestInsurance(
Optional<Person> person, Optional<Car> car) {
	if (person.isPresent() && car.isPresent()) {
		return Optional.of(findCheapestInsurance(person.get(), car.get()));
		} else {
			return Optional.empty();
		}
}
```

위 코드는 두 `Optional`을 인수로 받아서 둘 중 하나라도 비어있으면 빈 `Optional<Insurance>`를 반환한다.

`person`과 `car`의 시그니처만으로 둘 다 아무 값도 반환하지 않을 수 있다는 정보를 명시적을 주지만, 구현 코드는 null 확인 코드와 크게 다르지 않다.

```java
public Optional<Insutrance> nullSafeFindCheapestInsurance(
Optional<Person> person, Optional<Car> car) {
return person.flatMap( p -> car.map( c -> findCheapestInsurance(p, c)));
}
```

위 코드처럼 한 줄의 코드로 메서드를 재구현할 수 있다. 첫 번째 Optional에 flatMap을 호출했으므로 첫 번째 Optional이 비어있다면 인수로 전달된 표현식이 실행되지 않고 빈 Optional을 반환한다.

두 번째 Optional도 마찬가지이고, 두 값이 모두 존재한다면 map 메서드로 전달한 람다 표현식이 안전하게 findCheapestInsurance 메서드를 호출한다.

---

### **11.3.7 필터로 특정값 거르기**

```java
Insurance insurance = ...;
	if(insurance != null && "CambridgeInsurance".equals(insurance.getName())){
	System.out.println("ok");
}
```

위 코드처럼 객체의 메서드를 호출해서 프로퍼티를 확인해야 하는 경우, 다음과 같이 재구현할 수 있다.

```java
Optional<Insurance> optInsurance = ...;
	optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName()))
		.ifPresent(x -> System.out.println("ok"));
```

---

## **11.4 Optional을 사용한 실용 예제**

### **11.4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기**

```java
Object value=map.get("key");

// 위 코드와 같이 null이 반환될 수 있는 코드를 Optional로 감싸서 개선할 수 있다.

Optional<Object> value= Optional.ofNullable(map.get("key"));
```

---

### **11.4.2 예외와 Optional 클래스**

`null`을 확인할 때는 `if`문을 사용했지만 예외를 발생하시키는 메서드는 `try/catch` 블록을 사용해야 한다.

```java
public static Optional<Integer> stringToInt(String s) {	
	try {
			return Optional.of(Integer.parseInt(s));
	} catch (NumberFormatException e) {
		return Optional.empty();
	}
}
```

위 코드와 같이 정수로 변환할 수 없는 문자열은 빈 `Optional`로 `return`하도록 구현할 수 있다.

---

**11.4.3 기본형 Optional을 사용하지 말아야 하는 이유**

`Optional`도 기본형으로 특화된 `OptionalInt`, `OptionalLong`, `OptionalDouble` 등의 클래스를 제공함

하지만 `Optional`의 최대 요소 수는 한개이므로 기본형 특화 `Optional`로 성능을 향상시킬 수 없다.

또한 기본형 특화 `Optional`은 `map`, `flatMap`, `filter`등을 제공하지 않고, 생성한 결과를 다른 일반 `Optional`과 혼용할 수 없다
