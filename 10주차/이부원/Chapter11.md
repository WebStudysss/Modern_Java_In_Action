## NULL값이 발생하는 이유

11장의 초반은 NPE(NullPointException)에 대한 설명으로 시작한다.
NPE는 자바뿐만 아니라 모든 프로그래밍 언어에서 가장 많이 나오는 오류라고 생각한다. NPE가 발생하는 이유는 많이 있으나 대표적으로 객체가 제대로 생성되지 않았을 경우 NPE오류가 나타난다.

```java
public class Exam {
    public static void main(String[] args) {
        Animal animal = new Animal();
        if (animal.getAnimal().equals("강아지")){
            System.out.println("강아지입니다.");
        }

    }
}

class Animal{
    private String dog;

    public Animal() {
    }

    public String getAnimal(){
        return dog;
    }
}
```

위의 코드를 실행했을 경우 `NullPointerException`이 나오는 것을 확인할 수 있다.
이것을 예방하기 위해서는 null체크를 해주면 해결할 수 있다.

```java
        Animal animal = new Animal();
        if (animal.getAnimal() == null){
            System.out.println("동물이 존재하지 않아요.");
        } else {
            if (animal.getAnimal().equals("강아지")){
                System.out.println("강아지입니다.");
            }
        }
```
위와 같은 코드처럼 null값인지 확인해주기만 하면된다. 하지만 매번 이렇게 null값을 체크하는 것은 코드의 가독성을 해칠 수 있으며 만약 return 해야하는 값이 존재한다면 코드가 언제 return 되는지 알아볼 수 없게 된다. 즉, 출구(return 되는 위치)가 어디인지 모르는 것이다.
또한 코드가 복잡해진다면 null 값 체크를 누락할 수 있기도 하기때문에 더 좋은 방법이 필요하다.

## Optional 클래스

자바8에서 null값을 체크하기 위하여 `java.util.Optional<T>`라는 새로운 클래스를 제공한다.
값이 있으면 Optional 클래스는 값을 감싸고, 반면 값이 없으면 Optional.empty 메서드로 Optional을 반환한다. Optional.empty는 Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드이다.

```java
public class Exam {
    public static void main(String[] args) {
        Animal animal = new Animal();
        if (!animal.getAnimal().isPresent()){
            System.out.println("동물이 존재하지 않아요.");
        } else {
            if (animal.getAnimal().equals("강아지")){
                System.out.println("강아지입니다.");
            }
        }
    }
}

class Animal{
    private String dog;

    public Animal() {
    }

    public Optional<String> getAnimal(){
        return Optional.ofNullable(dog);
    }
}
```

Optional을 이용하여 `!animal.getAnimal().isPresent()` 형식으로 수정이 가능하다. 값이 존재하지 않는다면 if문에 들어가게 된다. Optional을 활용하면 null 값 체크가 횔씬 간편해지고 가독성도 좋아진다. 하지만 모든 null 참조를 Optional로 대치하는 것은 바람직하지 않다. Optional은 그저 역할을 더 이해하기 쉬운 API를 설계하도록 돕는 역할인 것이다.

## Optional 적용 패턴

- `Optional.empty()`

null을 담고 있는, 한 마디로 비어있는 Optional 객체를 얻어옵니다. 이 비어있는 객체는 Optional 내부적으로 미리 생성해놓은 싱글턴 인스턴스입니다.'

```java
Optional<Member> maybeMember = Optional.empty();
```

- `Optional.of(value)`

null이 아닌 객체를 담고 있는 Optional 객체를 생성합니다. null이 넘어올 경우, NPE를 던지기 때문에 주의해서 사용해야 합니다.

```java
Optional<Member> maybeMember = Optional.of(aMember);
```

- `Optional.ofNullable(value)`

null인지 아닌지 확신할 수 없는 객체를 담고 있는 Optional 객체를 생성합니다. Optional.empty()와 Optional.ofNullable(value)를 합쳐놓은 메소드라고 생각하시면 됩니다. null이 넘어올 경우, NPE를 던지지 않고 Optional.empty()와 동일하게 비어 있는 Optional 객체를 얻어옵니다.

```java
Optional<Member> maybeMember = Optional.ofNullable(aMember);
Optional<Member> maybeNotMember = Optional.ofNullable(null);
```

### flatMap으로 Optional 객체 연결

```java
public class Person {
    private Optional<Car> car;

    public Optional<Car> getCar() {
        return car;
    }
}

public class Car {
    private Optional<Insurance> insurance;

    public Optional<Insurance> getInsurance() {
        return insurance;
    }
}

public class Insurance {
  private String name;

  public String getName() {
      return name;
  }
}
```

map을 이용해서 위의 클래스들을 Optional로 연결하여 사용할 수 있다.

```
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson.map(Person::getCar)
		.map(Car::getInsurance)
        .map(Insurance::getName);
```

하지만 위의 코드는 컴파일 되지 않는다. 이유는 중첩된 Optional 객체는 map 연산을 사용할 경우 Optional 객체를 반환하기때문에 `Optional<Optional<Car>> car`와 같은 형태가 리턴된다. 이 문제를 해결하기 위해서는 flatMap을 이용하여 평준화 해줘야한다.

```java
Optional<Person> optionalPerson = Optional.of(person);
String name = optionalPerson.flatMap(Person::getCar)
                .flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("UNKNOWN");
```

flatMap을 이용하여 Optional을 재구성할 수 있었으며, 별도의 null값 확인하는 분기문을 추가하지 않아도 코드를 쉽게 이해할 수 있게되었다.


### Optional 스트림 조작

자바 9에서 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다.

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
    return persons.stream()
            .map(Person::getCar) // Optional<Car> 스트림으로 변환
            .map(optCar -> optCar.flatMap(Car::getInsurance)) // Optional<Car> -> Optional<Insurance> 로 변환
            .map(optInsurance -> optInsurance.map(Insurance::getName)) // Optional<Insurance> -> Optional<String> 으로 매핑
            .flatMap(Optional::stream) 
            .collect(Collectors.toSet());
```

위의 결과로 중간에 값이 없어 null값이 들어갈 수 있다. 마지막으로 결과를 얻기 위해서는 filter를 이용하여 결과를 얻을 수 있다.

```java
Stream<Optional<String>> optionalStream = persons.stream()
                                                .map(Person::getCar)
                                                .map(optCar -> optCar.flatMap(Car::getInsurance))
                                                .map(optInsurance -> optInsurance.map(Insurance::getName));
Set<String> result = optionalStream.filter(Optional::isPresent)
                                    .map(Optional::get)
                                    .collect(Collectors.toSet());
```

### 디폴트 액션과 Optional 언랩

- get()

get()을 사용하는 경우 래핑된 값이 있으면 값을 리턴하지만 값이 없으면 NoSuchElementException을 반환 한다. 따라서 Optional에 값이 반드시 있다고 가정할 수 있는 상황이 아니라면 get 메서드를 사용하지 않는 것이 바람직하다.

- orElse()

rElse를 사용하면 Optional이 값을 포함하지 않을 경우 기본값을 제공할 수 있다.

- orElseGet()

orElse 메서드에 대응하는 lazy 버전이다. Optional에 값이 없을때만 Supplier가 실행된다.

- orElseThrow()

Optional이 비어 있을때 예외를 발생시킨다는 점에서 get메서드와 비스하다.

- isPresent()

ifPresent를 이용하면 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다.

- isPresentOrElse()

Java 9 버전에 추가된 이 메서드는 Optional이 비어있을때 실행할 수 있는 Runnable을 인수로 받는다는 점만 ifPresent와 다르다.

### 두 Optional 합치기

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    if (person.isPresent() && car.isPresent()) {
        return Optional.of(new Insurance());
    }
    return Optional.empty();
}
```

이 메서드를 통해 person과 car의 시그니처만으로 둘 다 아무 값도 반환하지 않을 수 있다는 정보를 명시적으로 보여줬다.

### 필터로 특정값 거르기

- 객체로 특정값을 필터링할때는 null값을 체크한다.

```java
if (insurance != null && "XXX".equals(insurance.getName())) {
  System.out.println("ok");
}
```

위의 코드를 Optional의 filter를 이용하여 더 가독성 있게 구현할 수 있다.

```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance -> "CambridgeInsuracne".equals(insurance.getName()))
            .ifPresent(x -> System.out.println("ok"));
```

