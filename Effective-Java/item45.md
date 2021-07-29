# Item45 : 스트림은 주의해서 사용하라

스트림 API는 다량의 데이터 처리 작업을 돕고자 자바 8에서 추가되었다. 이 API가 제공하는 추상 개념 중 핵심은 다음과 같이 두가지이다.

1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
2. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

즉 스트림 파이프라인은 소스 스트림에서 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다. 각 중간 연산은 스트림을 어떠한 방식으로 변환한다.

중간 연산들은 모두 한 스트림을 다른 스트림으로 변환하는데, 변환된 스트림의 원소 타입은 변환 전 스트림의 원소 타입과 같을 수도 있고 다를 수도 있다. 

종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다. 즉 원소를  정렬해 컬렉션에 담거나, 특정 원소 하나를 선택하거나, 모든 원소를 출력하는 식이다.



### 스트림 파이프 라인

스트림 파이프라인은 지연 평가된다. 즉 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠이다. 

종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 no-op과 같으니, 종단 연산을 빼먹는 일이 절대 없도록 해야한다.

스트림 API는 메소드 연쇄를 지원하는 플루언드 API이며 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 기본적으로 스트림 파이프라인은 순차적으로 수행된다. 

파이프라인은 병렬로 실행하려면 파이프라인을 구성하는 스트림중에서 `parallel` 메소드를 사용하면 되나, 효과를 볼 수 있는 상황이 많지는 않다.



### 스트림 API

스트림 API는 다재다능하여 사실상 어떤 계산이라도 할 수 있지만, 적절히 사용해야 한다. 스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고  유지보수도 힘들어진다.

따라서 다음과 같은 상황에서 스트림을 사용하면 좋다. 다음 사전 파일에서 단어를 읽어 사용자가 지정한 문턱값보다 원소 수가 많은 아나그램 그룹을 출력하는 예제 인데 한번 보자.

* 같은 키를 사용하고 있는 아나그램 예제 (스트림 사용 x)

```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
    File dictionary = new File(args[0]);
    int minGroupSize = Integer.parseIont(args[1]);
    
    Map<String, Set<String>> groups = new HashMap<>();
    try (Scanner s = new Scanner(dictionary)) {
      while(s.hasNext()) {
        String word = s.next();
        // 맵 안에 키가 있으면 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환
        // 키가 없으면 건네진 함수 객체를 키에 적용하여 값을 계산해낸 다음 그 키와 값을 매핑해놓고, 계산된 값을 반환
        groups.computeIfAbsent(alphabetize(word), (unnused) -> new TreeSet<>()).add(word);
      }
    }
    
    for(Set<String> group : groups.value()) {
      if (group.size() >= minGroupSize) {
        System.out.println(group.size() + " : " + group);
      }
    }
  }
  
  private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
  }
}
```

다음 예는 스트림을 과하게 사용한 예이다.

* 스트림을 과하게 사용한 예

```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
    Path dictionary = Paths.get(args[0]);
    int minGroupSize = Integer.parseIont(args[1]);
    
    try (Stream<String> words = Files.lines(dictionary)) {
      words.collect(
      	groupingBy(word -> word.chars().sorted()
                  .collect(StringBuilder::new,
                          (sb, c) -> sb.append((char) c),
                          StringBuildder::append).toString()))
        .value().stream()
        .filter(group -> group.size() >= minGroupSize)
        .map(group -> group.size() + " : " + group)
        .forEach(System.out::println);
    }
  }
}
```

위 스트림을 과하게 사용한 결과 코드를 읽기가 매우 어렵다. 이처럼 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.

따라서 절충 지점을 찾으면 된다. 다음 예를 보자

* 스트림을  적절히 사용한 예제

```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
    Path dictionary = Paths.get(args[0]);
    int minGroupSize = Integer.parseIont(args[1]);
    
    try (Stream<String> words = Files.lines(dictionary)) {
      words.collect(groupingBy(word -> alphabetize(word))) 
        .value().stream()
        .filter(group -> group.size() >= minGroupSize)
        .forEach(group -> System.out.println(group.size() + " : " + group));
    }
  }
  
  private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
  }
}
```

이렇게 메소드와 스트림을 적절히 조합하여 사용하면 읽기도 쉽고 엄청난 시너지 효과를 나타낼 수 있다. 

여기서 주의할 점은 람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다. 그리고 도우미 메소드를 적절히 활용하는 일은 일반 반복코드에서보다는 스트림파이프라인에서 훨씬 중요하다.

alphabetize 메소드도 스트림을 사용해 다르게 구현할 수 있다. 하지만 그렇게 하면 명확성이 떨어지고 잘못 구현할 가능성이 커지며 심지어 느려질 수도 있다. 그러한 이유는 char용 스트림을 지원하지 않기 때문이다. 예를 들면 다음과 같다.

* char 값들을 스트림으로 처리하는 코드

```java
"Hello world!".chars().forEach(System.out::print);
```

Hello world!를 출력하리라 기대했지만, 721011..... 이런식으로 출력한다. chars()가 반환하는 스트림의 원소는 char가 아닌 int값이기 때문이다. 따라서 정숫값을 출력하는 print 메소드가 호출된 것이다.

만약 이를 스트림으로 처리해주고 싶으면 다음과 같이 해야한다.

* char형으로 출력하는 코드

```java
"Hello world!".chars().forEach(x -> System.out.print((char) x));
```

하지만 char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.

따라서 스트림을 사용할때 가장 중요한 것은 기존 코드는 스트림을 사용하도록 리팩토링하되, 새 코드가 더 나아 보일 때만 반영하자. 



### 스트림은 언제 사용해야 하나

스트림에서 람다를 사용할 때 다음과 같은 제약이 있다.

1. 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을  수 있고, 지역변수를 수정하는 건 불가능하다.
2. 코드 블록에서는 return문을 사용해 메소드에서 빠져나가거나, break나 continue문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다. 또한 메소드 선언에 명시된 검사 예외를 던질 수 있다. 하지만 람다는 이중 어떤 것도 할 수 없다.

즉 계산 로직에서 이상의 일들을 수행해야 한다면 스트림과는 맞지 않는 것이다. 

반대로 다음과 같은 일에는 스트림을 사용하는 것이 좋다.

1. 원소들의 시퀀스를 일관되게 변환한다.
2. 원소들의 시퀀스를 필터링한다.
3. 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.( 더하기, 연결하기, 최솟값 구하기 등 )
4. 원소들의 시퀀스를 컬렉션에 모은다.( 공통된 속성을 기준으로 묶어가며 )
5. 원소들의 시퀀스에 특정 조건을 만족하는 원소를 찾는다.



한편, 스트림으로 처리하기 어려운 일도 있다. 대표적인 예로 한 데이터가 파이프라인의 여러 단계를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기 어려운 경우이다. 그 이유는 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다. 그렇다고 아예 사용을 못하는건 아니다. 다음 예를 보자

* 처음 20개의 메르센 소수를 출력하는 프로그램

```java
static Stram<BigInteger> primes() { // 2^p-1 수에서 p가 소수이면 해당 메르센 수도 소수일 수 있는데 이때 그 수를 메르센 수
	return Stream.iterate(TWO, Biginteger::nextProbablePrime);
}

public static void main(String[] args) {
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE)) // 소수들을 사용해 메르센 수를 계산
    .filter(mersenne -> mersenne.isProbablePrime(50)) // 결과값이 소수인 경우만 남김 ( 50 개 )
    .limit(20) // 결과를 20개로 제한
    .forEach(System.out::println); // 출력
} 
```

이제 우리가 각 메르센 소수의 앞에 지수를 출력하기 원한다고 해보면, 이 값은 초기 스트림에만 나타나므로 결과를 출력하는 종단 연산에서는 접근할 수 없다. 하지만 첫번째 중간 연산에서 수행한 매핑을 거꾸로 수행해 메르센 수의 지수를 쉽게 계산 해낼 수 있다. 지수는 단순한  숫자를 이진수로 표현한 다음 몇 비트인지를 세면 나오므로, 종단 연산을 다음처럼 작성하면 원하는 결과를 얻을 수 있다.

* 수정 된 코드

```java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```



또한 스트림과 반복 중 어느쪽을 써야 할지 바로 알기 어려운 작업도 많다. 다음 예를 보자.

* 데카르트 곱 계산 - 반복 방식

```java
private static List<Card> newDeck() {
	List<Card> result = new ArrayList<>();
	for (Suit suit : Suit.values()){
		for (Rank rank : Rank.values) {
			result.add(new Card(suit, rank));
		}
	}
	return result;
}
```

* 데카르트 곱 계산 - 스트림 방식

```java
private static List<Card> newDeck() {
	return Stream.of(Suit.values())
    .flatMap(suit -> // flapMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합침
             Stream.of(Rank.values())
             			.map(rank -> new Card(suit, rank)))
    .collect(toList());
}
```

위의 예제들 처럼 답은 없다. 즉 개인 취향과 프로그래밍 환경의 문제다. 그러니 스트림에 대해 확신이 들지 않는다면 첫 번째 방식을 사용하는 것이 좋고 안전할 것이다. 그러나 스트림 방식이 더 나아보이고 동료들도 스트림 코드를 이해할 수 있고 선호한다면 스트림 방식을 사용하자.

