# Item 58 : 전통적인 FOR LOOP 보다 FOR EACH LOOP를 선호하라.

``` java
for (Element e : elements) {
  ...// Do something with e
}
```

- 위는 ```for each element e in elements``` 라고 읽힌다. 
- foreach 문은 전통적인 for loop와 비교했을때 성능차이가 없다.

## FOR EACH의 장점

``` java
// Can you spot the bug?
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
NINE, TEN, JACK, QUEEN, KING } ...

static Collection<Suit> suits = Arrays.asList(Suit.values()); 
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
	for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) 
    deck.add(new Card(i.next(), j.next()));
```

위 코드에는 버그가 있다. 바로  바깥 for문에 있는 Iterator i가 너무 많이 호출된다는 점이다. 결국 ```NoSuchElementException``` 이 발생할 가능성이 있고, exception이 나지 않더라도 비정상적인 결과가 나타나게 된다.

다음 처럼 Suit 값을 저장해서 해결할 수 있다.

``` java
// Fixed, but ugly - you can do better!
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) { 
  Suit suit = i.next();
	for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) 
    deck.add(new Card(suit, j.next()));
}
```

하지만 중첩 for-each문을 사용해서 더 간결하게 문제를 해결할 수 있다.

``` java
for (Suit suit : suits)
	for (Rank rank : ranks) 
    deck.add(new Card(suit, rank));
```

## for-each를 쓸수 없는 경우

- 파괴적 필터링 - Collection을 순회하면서 ```remove``` 함수를 호출해야 하는 경우
- 변환 - 리스트 또는 배열을 순회하면서 일부 값들을 치환해야 하는 경우, 인덱스나 iterator를 사용할 수 밖에 없다.
- 병렬 순회 - 여러 컬레션들을 병렬로 순회해야 할 경우

## Iterable interface를 구현하는 객체들은 모두 for-each 문으로 순회가능하다.

``` java
public interface Iterable<E> {
// Returns an iterator over the elements in this iterable Iterator<E> iterator();
}
```

## 요약

for-each loop는 명료성, 유연함, 버그 방지, no performance penalty(성능저하 없음) 측면에서 확실한 장점이 있으므로, 가능하면 for loop 대신에 for-each 문을 사용하라.