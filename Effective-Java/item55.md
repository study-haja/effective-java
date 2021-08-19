# Item55: 옵셔널 반환은 신중히 하라

자바 8 이전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지가 다음과 같이 두 가지 있었는데 각각 문제가 있었다.

1. 예외를 던진다. -> 진짜 예외에서만 사용해야 한다.
2. null을 반환한다. -> null이 절대 반환되지 않는다고 확신하지 않는 한 별도의 null 처리 코드를 추가해야 한다.

하지만 자바8에 Optional이 생기고나서 위의 문제들을 어느정도 해결해 줄 수 있었다.



###  Optional

Optional<T> 는 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다는 뜻이다. 즉 옵셔널은 원소를 최대 1개 가질 수 있는 불변 컬렉션인다.

보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 Optiona<T>를 반환하도록 선언하면 좋다. 그러면 유효한 반환값이 없을 때는 빈 결과를 반환하는 메서드가 만들어진다. 

옵셔널을 반환하는 메서드는 예외를 던지는 메소드보다 유연하고 사용하기 쉬우며, null을 반환하는 메소드보다 오류 가능성이 작다. 다음 예를 보자.

* 컬렉션에서 최댓값을 구하는 예제

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
	if (c.isEmpty())
    throw new IllegalArgumentException("빈 컬렉션");
  
  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  
  return result;
}
```

위 예제는 빈 컬렉션을 건네면 IllegalArgumentException을 던진다. 하지만 Optional을 반환하면 다음과 같다.

* 컬렉션에서 최댓값을 구하고 Optional로 반환하는 예제

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
	if (c.isEmpty())
    throw Optional.empty(); // 빈 옵셔널 
  
  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  
  return Optional.of(result); // null이 들어가면 NullPointerException 발생 -> 절대 null 반환x
}
```

스트림의 종단 연산 중 상당수가 옵셔널을 반환한다. 위의 max 예제를 스트림 버전으로 작성한다면 다음과 같다.

* 컬렉션에서 최댓값을 구하고 Optional로 반환 스트림 버전 예제

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
  return c.stream().max(Comparator.naturalOrder());
}
```

그렇다면 null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야 하는 기준은 무엇일까?

>  옵셔널은 검사 예외와 취지가 비슷하다.

즉 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려준다. 만약 비검사 예외를 던지거나 null을 반환한다면 API 사용자가 그 사실을 인지하지 못해 끔찍한 결과를 초래할 수 있다. 하지만 검사 예외를 던지면 클라이언트에서는 반드시 이에 대처하는 코드를 작성해 넣아야 한다.

비슷하게, 메소드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을때 취할 행동을 선택해야 한다. 그중 하나는 기본값을 설정하는 방법이다.

* Optional 기본값 설정

```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

또는 상황에 맞는 예외를 던질 수 있다. 예외 팩토리를 사용해 실제로 발생하지 않는한 예외 생성 비용은 들지 않는다.

*  Optinal 예외 설정

```java
Toy myToy = max(toy).orElseThrow(TemperTantrumException::new);
```

옵셔널에 항상 값이 채워져 있다고 확신한다면 그냥 곧바로 값을 꺼내 사용할 수 도 있다. 다만 잘못 판단한 것이라면 NoSuchElementException이 발생 할 것이다.

* 항상 값이 채워져 있다고 가정한 코드

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

이따금 기본값을 설정하는 비용이 아주 커서 부담이 될 때가 있다. 그럴 때는 Supplier<T>를인수로 받는 orElseGet을 사용하면, 값이 처음 필요할 때 Supplier<T>를 사용해 생성하므로 초기 설정 비용을 낮출 수 있다. 더 나아가 filter, map, flatMap,ifPresent등 특별한 쓰임에 대비한 메소드도 있기 때문에 적절하게 사용하면 된다.

여기서 ifPresent를 보면 옵셔널이 비어있으면 false, 채워져있으면 true를 반환한다. 이 메소드로 원하는 작업을 모두 수행할 수 있지만, 앞에 언급했던 메소드들로 대체할 수 있다. 다음 예를 보자.

*  isPresent()로 구현한 코드

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " + (parentProcess.isPresent() ? 
                                 String.valueOf(parentProcess.get().pid())  : "N/A"))
```

* map으로 구현한 코드

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " +
                  ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

스트림을 사용한다면 옵셔널들을 Stream<Optional<t>>로 받아서, 그중 채워진 옵셔널들에게 값을 뽑아 Stream<T>에 건네 담아 처리하는 경우가 드물지 않다. 자바 8에서는 다음과 같이 구현할 수 있다.

```java
streamOfOptionals
  .filter(Optional::isPresent)
  .map(Optional::get)
```

자바 9에서는 Optional에 stream() 메소드가 추가되었다. 이 메소드는 Optional을 Stream으로 변환해주는 어댑터이다. 옵셔널에 값이 있으면 가 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로 변환한다. 이를 stream의 flatMap 메소드와 조합하면 앞의 코드를 다음처럼 바꿀 수 있다.

```java
streamOfOptionals
  .flatMap(Optional::stream)
```

반환값으로 옵셔널을 사용한다고 해서 무조건 득이 되는 건 아니다. **컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.** 즉 빈 Optional<List<T>>를 반환하기보다는 빈 List<T>를 반환하는 것이 좋다. (null 보단 빈 컬렉션을 사용하는 것이 좋다.)



### 결론

그렇다면 어떤 경우에 메서드 반환타입을 T 대신 Optional<T>로 선언해야 할까? 바로 다음과 같다.

1. 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T>를 반환한다.

그런데 이렇게 하더라도 Optional<T>를 반환하는 데는 대가가 따른다. Optional도 엄연히 새로 할당하고 초기화해야 하는 객체이고, 그 안에서 값을 꺼내려면 메소드를 호출해야 하는 단계를 거치니 성능이 중요한 상황에서는 옵셔널이 맞지 않을 수 있다.

박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수 밖에 없다. 값을 두겹이나 감싸기 때문이다. 따라서 자바 API는 OptionalInt, OptionalLong, OptionalDouble 과 같은 옵셔널 클래스를 만들어놨다. 이렇게 대체제가 있으니 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자. 

그리고 옵셔널을 맵의 키나 값으로 사용하면 절대 안된다. 쓸데없이 복잡성만 높이고 오류 가능성만 키운다. 

또 옵셔널을 인스턴스 필드에 저장해두는 방식은 때에 따라 설정하는 것이 좋다.