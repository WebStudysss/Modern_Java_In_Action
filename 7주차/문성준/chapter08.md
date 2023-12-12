# 8. 컬렉션 API 개선

<aside>
📌 - 컬렉션 팩토리 사용하기
- 리스트 및 집합과 사용할 새로운 관용 패턴 배우기
- 맵과 사용할 새로운 관용 패턴 배우기

</aside>

---

### **8.1 컬렉션 팩토리**

자바에서는 적은 요소를 포함하는 리스트를 어떻게 만들까?

```java
// 다음 방법은 세 문자열을 저장하는데도 많은 코드가 필요하다.
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```

다음처럼 `Arrays.asList()` 팩토리 메서드를 이용해 코드를 줄일 수 있지만, 고정 크기의 리스트이기 때문에 요소를 갱신할 수는 잇지만 새 요소를 추가하거나 삭제하려면 Unsupported OperationException이 발생한다.

```java
// 
List<String> friends
	= Arrays.asList("Raphael", "Olivia ", "Thibaut");
//
List<String> friends = Arrays.asList("Raphael", "Olivia");
friends.set(0, "Rechard");
friends.add("Thibaut");
```

### **Unsupported OperationException 예외 발생**

내부적으로 고정된 크기의 변환할 수 있는 배열로 구현되었기 때문에 이와같은 일이 일어난다.

리스트를 인수로 받는 HashSet 생성자를 사용하거나, 스트림 API로 해결할 수 있다.

```java
// 리스트를 인수로 받는 HashSet 생성자
Set<String> friends
	= new HashSet<>(Arrays.asList("Raphael", "Olivia","Thibaut"));
// 스트림 API 사용
Set<String> friends 
	= Stream.of("Raphael", "Olivia","Thibaut")
					.collect(Collector.toSet());
```

하지만 두 방법 모두 매끄럽지 못하며 불필요한 객체 할당을 필요로 한다.

---

### **8.1.1 리스트 팩토리**

`List.of` 팩토리 메서드를 이용하면 간단하게 리스트를 만들 수 있다.

```java
List<String> friends = List.of("Raphael", "Olivia","Thibaut");
```

하지만 마찬가지로 변경할 수 없는 리스트이기 때문에 `add()`나 `set()`메서드를 사용할 수 없다.
이런 제약이 무조건 나쁜 것은 아니다. 컬렉션이 의도치 않게 변하는 것을 막을 수 있기 때문이다.
하지만 요소 자체가 변하는 것을 막을 수 있는 방법은 없다.
즉, 리스트는 변경 가능한 자료형이기 때문에, 리스트 자체에 대한 참조가 변경되지 않더라도 해당 리스트의 내용을 직접 변경할 수 있습니다. 이는 리스트가 가변하다는 의미입니다. 리스트에 요소를 추가하거나 제거하면 리스트가 변경됩니다.

<aside>
❓ Q. 어떤 상황에서 새로운 컬렉션 팩토리 메서드 대신 스트림 api를 사용해 리스트를 만드는지 궁금
A.

1. **데이터 변환 및 필터링**
    - 스트림 API는 강력한 변환 및 필터링 연산을 제공합니다. 예를 들어, 데이터의 일부만 선택하거나 특정 조건을 만족하는 요소만 필터링할 수 있습니다.
        
        ```java
        List<String> filteredList = myList.stream()
                                         .filter(s -> s.length() > 3)
                                         .collect(Collectors.toList());
        ```
        
2. **병렬 처리**
    - 스트림 API는 병렬 처리를 효과적으로 지원합니다. 대량의 데이터를 처리할 때, 멀티코어 프로세서에서 작업을 분산시켜 성능을 향상시킬 수 있습니다.
        
        ```java
        List<String> parallelList = myList.parallelStream()
                                         .map(String::toUpperCase)
                                         .collect(Collectors.toList());
        ```
        
3. **데이터 그룹화 및 분할**
    - 스트림 API를 사용하면 데이터를 그룹화하거나 분할하는 작업을 쉽게 수행할 수 있습니다.
        
        ```java
        Map<Integer, List<String>> groupedByLength 
        = myList.stream().collect(Collectors.groupingBy(String::length));
        ```
        
4. **매핑 및 리듀싱**
    - 스트림 API는 매핑 및 리듀싱 작업을 지원합니다. 데이터를 특정 속성으로 매핑하거나, 모든 요소를 하나로 리듀스할 수 있습니다.
        
        ```java
        int totalLength = myList.stream()
                                .map(String::length)
                                .reduce(0, Integer::sum);
        ```
        
5. **지연 평가**
    - 스트림 연산은 지연 평가를 지원하며, 필요한 경우에만 계산이 이루어집니다. 이는 불필요한 계산을 피하고 성능을 최적화하는 데 도움이 됩니다.
        
        ```java
        Stream<String> stream 
        = myList.stream().filter(s -> s.length() > 3);
        ```
        
</aside>

### **8.1.2 집합 팩토리**

List.of와 비슷한 방법으로 바꿀 수 없는 집합을 만들 수 있다.

`Set<String> friends = Set.of("Raphael", "Olivia","Thibaut");`

중복된 요소를 제공해 집합을 만들려고 한다면, 요소가 중복되어 있다는 `IllegalArgumentException`에러가 발생한다. 집합은 오직 고유의 요소만 포함할 수 있기 때문이다.

### **8.1.3 맵 팩토리**

Map.of 팩토리 메서드에 키와 값을 번갈아 제공하는 방법으로 맵을 만들 수 있다.

```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
```

10개 이상의 키와 값 쌍을 가진 맵을 만들때는 Map.Entry<K, V> 객체를 인수로 받는 Map.ofEntries 팩토리 메서드를 이용하는 것이 좋다. 이 메서드는 키와 값을 깜쌀 추가 객체 할당을 필요로 하기 때문이다.

```java
import static java.util.Map.entity;
Map<String, Integer> ageOfFrieds = Map.ofEntries(
entry("Raphael", 30),
entry("Olivia", 25),
entry("Thibaut", 26));
```

```java
List<String>ac = List.of("kn","js");
ac.set(0,"B");
System.out.println(ac);
// 위 코드는 List.of로 만든 컬렉션-> 불변이라 ERR
// 수정
import java.util.*;
class Main {
    public static void main(String[] args) {
        List<String> ac = new ArrayList<>(List.of("kn", "js"));
        ac.set(0, "B");
        System.out.println(ac);
    }
}
// 리스트의 요소를 변경하고 싶었기 때문에, 가변한 리스트를 사용하기 위해 ArrayList 사용해서
// 리스트를 생성하고 요소를 변경했음.
```

---

## **8.2 리스트와 집합 처리**

자바 8에서는 List, Set 인터페이스에 다음과 같은 메서드를 추가했다.

- `removeIf` : 프레디케이트를 만족하는 요소를 제거한다. (List, set)
- `replaceAll` : UnaryOperator 함수를 이용해 요소를 바꾼다. 리스트에서 사용할 수 있다.( List)
- `sort` : 리스트를 정렬한다. (Interface에서 제공하는 기능)

이 메서드는 새로운 결과를 만드는 것이 아니라 호출한 기존 컬렉션 자체를 바꾼다.

### **8.2.1 removeIf 메서드**

다음은 숫자로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드다.

```java
for(Transaction transaction : transactions) {
if(Character.isDigit(transaction.getReferanceCode().charAt(0))) {
transaction.remove(transaction);}}
```

코드를 실행해보면 `ConcurrentModificationException`을 일으킨다.

forEach 루프는 Iterator 객체를 사용하므로 위 코드는 다음과 같이 해석된다.

```java
for(Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();){
Transaction transaction = iterator.next();
if(Character.isDigit(transaction.getReferanceCode().charAt(0))) {
transactions.remove(transaction);}}// 반복하면서 별도의 두 객체를 통해 컬렉션을 바꾸고 있는 문제
```

- `Iterator` 객체를 통해 소스를 질의하고 `next()`, `hasNext()`를 통해 소스를 질의한다.
- `Collection` 객체 자체에 `remove()`를 호출해 요소를 삭제하고있다. 
따라서 반복자의 상태와 컬렉션의 상태가 동기화되지 않는다.

Iterator 객체를 명시적으로 사용하고 그 객체의 remove() 메서드를 호출해줘야 정상적으로 동작한다.

```java
for(Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
Transaction transaction = iterator.next();
if(Character.isDigit(transaction.getReferanceCode().charAt(0))) {
iterator.remove();}}
```

위 코드 패턴은 자바8의 `removeIf` 메서드로 바꿀 수 있다. `removeIf`는 삭제할 요소를 가리키는 프레디케이트를 인수로 받는다.

```java
transactions.removeIf(transaction ->
								Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

### **8.2.2 replaceAll 메서드**

리스트의 각 요소를 새로운 요소로 바꾸려고 할 때 스트림 API를 사용하면 다음과 같다.

```java
referenceCode.stream().map(code -> Character.toUpperCase(code.charAt(0)) + 
	code.subString(1))
			.collect(Collectors.toList())
			.forEach(System.out::println);
```

하지만 이 코드는 새 문자열 컬렉션을 만든다. 기존 컬렉션을 바꾸려면 `ListIterator` 객체를 이용해야한다.

```java
for(ListIterator<String> iterator = referenceCodes.listIterator();
iterator.hasNext(); ) {
String code = iterator.next();
iterator.set(Character.toUpperCase(code.charAt(0)) + code.subString(1));}
```

코드가 복잡해졌다. 그리고 이처럼 컬렉션 객체와 `Iterator` 객체를 혼용하면 반복과 변경이 동시에 이루어져 쉽게 문제를 일으킨다. 이는 자바 8의 replaceAll 메서드를 사용하면 간단하게 구현할 수 있다.

```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + 
code.subString(1));
```

---

# **8.3 맵 처리**

### **8.3.1 forEach 메서드**

맵에서 키와 값을 반복하면서 확인하는 작업은 번거롭다.

```java
// 실제 Map.Entry<K,V>의 반복자를 이용해 맵의 항목 집합을 반복할 수 있다.
for(Map.Entity<String, Integer> entry : ageOfFriends.entitySet()) {
String friends = entry.getKey();
Integer age = entry.getValue();
System.out.println(friend + " is " + age + " years old");}
```

자바8의 `Map` 인터페이스는 `BiConsumer`(키와 값을 인수로 받음)를 인수로 받는 `forEach` 메서드를 지원함.

```java
ageOfFrends.forEach((friends, age) 
-> System.out.println(friend + " is " + age + " years old"));
```

### **8.3.2 정렬 메서드**

새로 추가된 유틸리티를 이용해 맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다.

- `Entry.comparingByKey`
- `Entry.comparingByValue`

```java
Map<String, String> favouriteMovies 
	= Map.ofEntries(entry("Raphael", "Star Wars"),
	entry("Cristina", "Matrix"),
	entry("Olivia", "James Bond"));
favouriteMovies
	.entrySet()
	.stream()
	.sorted(Entry.ComparingByKey())
	.forEachOrdered(System.out::println); // 사람의 이름을 알파벳 순으로 스트림 요소를 처리

/* 출력
cristina=matrix
olivia=james Bond
Raphael=Star Wars
*/
```

### **8.3.3 getOutDefault 메서드**

기존에는 찾으려는 키가 존재하지 않는다면 null이 반환되며, 
만약 nullPointerException을 방지하려면 null인지 확인해야 한다. → 기존의 키 존재를 확인한다는 소리.
→ 기본값을 반환하는 방식으로 이 문제를 해결할 수 있다. `getOrDefault` 메서드를 이용하면 쉽게 이 문제를 해결할 수 있다.

이 메서드는 첫 번째 인수로 키를, 두 번째 인수로 기본값을 받으며 맵에 키가 존재하지 않으면 두 번째 인수로 받은 기본값을 반환한다. 즉, 요청결과를 확인하지 않아도 `NullPointerException` 문제를 해결할 수 있다.

```java
Map<String, String> favouriteMovies 
	= Map.ofEntries(entry("Raphael", "Star Wars"),
	entry("Olivia", "James Bond"));

System.out.println(favorieMovies.getOrDefault("Olivia", "Matrix")); //James Bond 출력
System.out.println(favorieMovies.getOrDefault("Thibaut", "Matrix")); //Matrix 출력
/*
키가 존재하더라도 null인 상황에선 getOrDefault 가 null을 반환할 수 있다.
→ 즉, 키가 존재하느냐의 여부에 따라서 두 번째 인수가 반환될지 결정된다는 말이다.
*/
```

### **8.3.4 계산 패턴**

맵에 키가 존재하는지 여부에 따라 동작을 수행하고자 할 때에 사용할 수 있는 연산이 있다.

- `computeIfAbsent` : 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 값을 계산하고 맵에 추가한다.
- `computeIfPresent` : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
- `comput` : 제공된 키로 새 값을 계산하고 맵에 저장한다.

### **8.3.5 삭제 패턴**

키가 특정한 값일 경우 항목을 제거하도록 `remove` 메서드를 사용할 수 있다.

```java
String key = "Raphael";
String value = "Jack Reacher 2";
favoriteMovies.remove(key, value);
```

### **8.3.6 교체 패턴**

맵의 항목을 바꾸는데 사용할 수 있는 두 개의 메서드가 추가되었다.

- replaceAll : BiFunction을 적용한 결과로 각 항목의 값을 교체한다.
- Replce : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다.

```java
//repaceAll을 적용할 것이므로 바꿀 수 있는 맵을 사용해야 한다.
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars"),
favouriteMovies.put("Olivia", "James Bond");
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

### **8.3.7 합침**

맵을 합칠때는 `putAll`을 사용할 수 있다.

```java
Map<String, String> family = Map.ofEntries(
	entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friens = Map.ofEntries(
	entry("Raphael", "Star Wars"));
Map<String, String> everyone = newHashMap<>(family);
everyone.putAll(friends);
```

중복된 키가 있다면 merge 메서드를 이용할 수 있다. 이 메서드는 중복된 키를 어떻게 합칠지 결정하는 BiFunction을 인수로 받는다.

```java
Map<String, String> everyone = newHashMap<>(family);
friends.forEach((k, v) ->
everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
```

---

## **8.4 개선된 ConcurrentHashMap**

ConcurrentHashMap는 병렬 친화적인 HashMap이다. 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다.

### **8.4.1 리듀스와 검색**

ConcurrentHashMap은 스트림과 비슷한 세 가지 새로운 연산을 지원한다.

- `forEach` : 각 (키,값) 쌍에 주어진 액션을 실행
- `reduce` : 모든 (키,값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- `search` : 널이 아닌 값을 반환할 때까지 각 (키,값) 쌍에 함수를 적용

다음처럼 키, 값, Map.entry 인수를 이용해 네 가지 연산 형태를 지원한다.

- 키, 값으로 연산 (forEach, reduce, search)
- 키로 연산 (forEachKey, reduceKeys, searchKeys)
- 값으로 연산 (forEachValue, reduceValues, searchValues)
- Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)

이들 연산은 ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행하므로, 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다.

또한 병렬성 기준값(threshhold)를 지정해야 하는데, 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다. 기준값이 1일때는 공통 스레드 풀을 이용해 병렬성을 극대화하며, 기준값이 Long.MAX_VALUE일때는 한 개의 스레드로 연산을 실행한다.

이 예제에서는 reduceValues 메서드를 이용해 맵의 최댓값을 찾는다.

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
Optional<Integer> maxValue =
	Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

### **8.4.2 계수**

ConcurrentHashMap 클래스는 맵의 매핑 개수를 반환하는 mappingCount 메서드를 제공한다. size 대신 mappingCount 를 사용해야 매핑의 개수가 int의 범위를 넘어서는 이후의 상황을 대처할 수 있다.

### **8.4.3 집합뷰**

ConcurrentHashMap을 집합 뷰로 반환하는 keySet 메서드를 제공한다. 맵을 바꾸면 집합도 바뀌고 집합을 바꾸면 맵도 영향을 받는다. newkeySet 메서드를 이용하면 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있다.
