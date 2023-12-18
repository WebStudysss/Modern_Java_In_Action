# Chapter08 컬렉션 API 개션

자바 8, 9에 추가된 Collection API를 배우는 장

# 컬렉션 팩토리

작은 컬렉션 객체를 쉽게 만들 수 있는 몇 가지 방법을 제공 ⇒ 코드의 길이를 줄이기 위해

```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");

->

List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
```

- asList는 값의 사이즈를 고정시킨 Arrays의 `ArrayList구현체용 팩토리 메서드`
- 나머지 set, add 등은 구현되어있음

## 컬렉션별 팩토리 메서드

- **List.of**
    - null 불가능
    - set, add 미지원
        - 컬렉션이 의도치 않게 변하는 것을 막을 수 있음
    - 각 요소 안의 자체 데이터는 변경할 수 있음(get활용)
    - Collectors.toList도 동일 but 좀 더 간결한 `List.of`
- **Set.of**
    - of() 안의 데이터에 중복 요소가 있을 시 Illegal Exception 발생
- **Map**
    - Map.of
        - (K1, V1, K2, V2) 형식으로 초기화
    - Map.ofEntry
        - entry(k, v)를 통해 map 데이터 생성

# 리스트와 집합 처리

자바 8버전에 추가된 List와 Set 인터페이스 메서드 ⇒ 컬렉션을 바꾸는 동작을 에러 최소화

- **removeIf** : Predicate를 만족하는 요소 전체 탐색 후 제거
    
    ```java
    for(Transaction transaction : transactions){
          if(transaction.getTrader().getName().equals("dd")){
            transactions.remove(transaction);
          }
        }

    //Iterator의 계산 중간에 remove가 이뤄진다면
    //Iterator의 인덱스 참조가 변경됨 Error발생!
    ->
    
        transactions.removeIf(transaction ->
                transaction
                        .getTrader()
                        .getName()
                        .equals("dd")
        );
    ```
    
- **replaceAll** : UnaryOperator를 이용해 각 요소를 변경
    - 새로운 컬렉션을 생성하는 것이 아니라 교체하는 것
- **sort** : List를 정렬

# 맵 처리

Java 8부터 지원하는 Map 인터페이스 메서드

- **forEach**
    - 맵을 반복 확인하는 작업은 상당히 번거로운 작업 중 하나
    ⇒ 디폴트 메서드로 반복할 수 있게 지원
    - 맵에만 적용 되는 것은 아님
    ⇒ list, set 동일
    - ordered
    ⇒ 병렬처리 시 요소 순서일치하게 처리
- 정렬
    - Entry.comparingByValue
    - Entry.comparingByKey
    
    ```java
    Map<String, String> favouriteMovies
    
    ->
    
    favouriteMovies
    	.entrySet()
    	.stream()
    	.sorted(Entry.comparingByKey())
    	.forEachOrdered(System.out::println);
    ```
    
- getOrDefault(Key, defaultValue)
    - 키가 존재하지 않아도 NullPointException을 방지하기 위한 방식
- 계산
    - 맵에 키가 존재하는지 여부에 따라 동작할 수 있게 돕는 메서드
    - computeIfAbsent(Key) : key에 맞는 Value가 존재하는지 확인 후 존재하지 않으면 `키값을 기준`으로 값 처리 후 저장
    - computeIfPresent(Value) : value에 맞는 키가 존재하면 새 값을 계산하고 맵에 추가
    - compute(Key) : 값의 존재 유무와 상관없이 해당 값 기준 동작
- 삭제
    - remove(key, value) : 기존의 remove는 key기준
    ⇒ Key, value가 일치하는 것을 제거
- 교체
    - replace : 키가 존재하면 맵의 값을 교체
        - replace(key, value)
        - replace(key, oldValue, newValue)
    - replaceAll(BiFunction) : 각 항목의 값을 교체
- 합침
    - merge() : putAll()같은 과정을 진행할 때 중복되는 키가 있을 경우 어떻게 합쳐야할지 선택할 수 있는 BiFunction을 활용해 값 처리
    
    ```java
    moviesToCount.merge(movieName, 1L, (count, increment) -> count + 1L)
    ```
    

# 개선된 ConcurrentHashMap

동시성 친화적인 HashMap 버전이며, 내부 자료구조의 특정 부분만 잠궈 추가 갱신 작업을 허용한다.

- 리듀스와 검색
    - forEach, reduce, search메서드와, 각 메서드의 특성을 살려 K, (K,V), V, Map.entry 등 선택해서 구현하는 메서드가 있다.
- 계수
    - size → mappingCount ⇒ 병렬처리를 하는 이유는 큰 데이터를 처리할 때 사용할텐데, 기존의 size는 int형이기 때문에 더 큰 mappingCount를 통해 지원
- 집합뷰
    - KeySet이라는 맵을 바꾸면 집합도 바뀌고 집합을 바꾸면 맵도 영향을 받는다.
    - newKeySet으로 영향받지않는 Set을 만들 수 있음

# 마치며

- 팩토리메서드
- 컬렉션 팩토리가 반환한 객체는 사이즈 불변
- 각 컬렉션 별로 새로운 디폴트 메서드들을 지우너한다.
- ConcurrentHashMap은 스레드안전성을 제공하는 해시맵
