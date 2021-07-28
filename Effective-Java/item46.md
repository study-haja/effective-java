#  ITEM 46 : STREAM에서 SIDE EFFECT가 없는 함수를 선호하라.

스트림 패러다임은 계산을 일녀의 transformation으로 구조화하는 것인데, 여기서 transformation은 이전 단계의 결과에 대해 항상 pure function 성질을 유지해야 한다. 

>pure function 이란 결과가 항상 input에만 의존하는 함수를 말한다.
>
>이를 달성하기 위해서는, stream 작업도중에 중간 결과나 죄종 결과들이 side effect가 없어야 한다. 
>
>side effect는 외부 상태를 변경하거나 예상치 못한 에러가 발생하는 상황을 말한다.

## forEach 작업은 항상 stream의 상태를 확인하기 위해서만 사용할것

다음 코드는 각각의 단어에 대한 빈도수 테이블을 만드는 코드이다.

``` java
Map<String,Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
  words.forEach(word -> {
    freq.merge(word.toLowerCase(),1L,Long::sum);
  })
}
```

위 코드의 문제점은 전혀 스트림 코드가 아니라는 점이다. 스트림 코드 가면을 쓴 iterative code이다. 문제의 원인은 모든 작업을 forEach 작업에서 수행하고 있다는 점이다.

forEach 작업에서 단지 stream의 결과를 나타내는 것 이상의 일을 하고 있다면 코드에 냄새가 난다고 볼수 있다.

이를 개선한 버전은 아래와같다.

``` java
Map<String,Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
  freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

아까전 보다 더 짧아지고 깨끗해졌음을 알수 있다. 

## Collectors API

Collectors API는 39개의 함수를 갖고 있고, 그중에서는 파라미터가 4가지가 되는 것도 있어서 매우 어렵게 느껴진다. 하지만 초심자 입장에서는 collector를 단지 reduction 전략을 감싸고 있는 객체라고 봐도 된다.

3가지 collector가 존재한다.

- toList()
- toSet()
- toCollection(collectionFactory) : 프로그래머가 명시한 collection type

다음은 빈도 테이블에서 top 10개의 단어를 골라내는 예제이다.

``` java
List<String> topTen = freq.keySet().stream()
  	.sorted(comparing(freq::get).reversed())
  	.limit(10)
  	.collect(toList());
```

```comparing``` 함수는 comparator construction method이고  ``` freq::get ``` 함수에서는 빈도수를 리턴한다. 

### Map Collector

1. 가장 간단한  ``` toMap(keyMapper, valueMapper) ``` 형태

아래는 Item 34에서 본 enum string 값을 key로 하고 enum을 value 하는 map 생성 예제이다.

``` java
private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(toMap(Object::toString, e->e))
```

2. 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성하는 예시

아래는 album 스트림에서 artist를 key로하고, artist의 판매량이 가장 높은 앨범을 value로 하는 예제이다.

``` java
Map<Artist, Album> topHits = albums.collect(
	toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```

3. 마지막에 쓴 값을 취하는 collector

같은 key를 가진 원소가 여러개 있을경우, 그중에 최근 값만을 취하고 싶을 수 있다. 그럴때 toMap을 다음과 같이 사용한다.

``` java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

### Grouping By

원소들을 카테코리별로 분류하고 싶을때 사용할수 있다. 

다음은 Item45에서 봤던 anagram(순서는 다르고 같은 문자들로 이루어진 단어들)을 구하는 예이다.

``` java
words.collect(groupingBy(word->alphabetize(word)))
```

### Joining

문자열과 같은  ```CharSequence``` 스트림에서만 동작한다. 원소들을 concatenate 하는 컬렉터를 리턴한다. 오직 delimiter 라는 파리미터만 취하고, 스트림 원소들에 delimiter를 삽입하여 join한다. 

## 요약

스트림 파이프라인을 프로그래밍하는것의 본질은 side-effect 없는 함수 객체들이다.

stream의 종료작업에서 forEach를 할경우, 계산의 결과를 보고하는 용도로만 사용되어야 한다. 절대 계산을 하는 용도로 사용되어서는 안된다. stream을 적절히 사용하기 위해서는  collector에 대해서 알아야하는데, 가장 중요한 collector 5가지는 다음과 같다.  ```toLIst, toSet, toMap, groupingBy, joining```
