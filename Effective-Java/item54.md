# Item 54 : NULL 대신 빈 컬렉션 이나 빈 배열을 반환하라

다음 처럼 리스트가 비어있을때 null을 반환하는 코드가 있다고 하자.

``` java
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheeses() { 
  return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock); 
}
```

이렇게 할 경우, 저 함수를 사용하는 클라이언트는 다음과 같은 null 체크 로직을 수행해야 한다.

``` java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
System.out.println("Jolly good, just the thing.");

```

이러한 코드는 실수할 가능성이 존재한다.

빈 컨테이너 객체를 할당하는 비용이 발생할것 같지만 2가지 이유로 걱정할 필요가 없다.

1. 이러한 수준의 최적화는 성능에 직접적으로 영향이 있는 경우가 아니면 고려할 필요가 없다.
2. 성능이 걱정될 경우, 메모리 할당없이 빈 컬렉션이나 배열을 반환할수 있다.

## 빈 컬렉션 반환하기

먼저 다음처럼 빈 컬렉션을 반환할 수 있다.

``` java
public List<Cheese> getCheeses() {
return new ArrayList<>(cheesesInStock);
}
```

여기서 비어있는 컬렉션에 메모리 할당을 줄이는 방법은 ```Collections.emptyList``` 를 사용하는 것이다.

``` java
public List<Cheese> getCheeses() {
return cheesesInStock.isEmpty() ? Collections.emptyList()
: new ArrayList<>(cheesesInStock); }
```

비슷한 예로, ```Collections.emptySet``` , ```Collections.emptyMap``` 가 있다.

## 비어있는 배열 반환하기

이 경우도, 절대 비어있는 배열 대신에 null을 반환해서는 안된다.

``` java
public Cheese[] getCheeses() {
return cheesesInStock.toArray(new Cheese[0]);
}
```

여기서도 비어있는 배열에 메모리할당하는 것이 부담되면 다음처럼 할수 있다.

``` java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
public Cheese[] getCheeses() {
return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

>toArray 함수의 경우, 컬렉션에 있는 원소 갯수보다 인자로 전달된 배열의 길이가 더 길면, 인자로 전달된 배열을 리턴한다.
>
>만약 컬렉션에 있는 원소의 갯수가 더 크면, 컬렉션의 크기 만큼 배열을 새로 할당하여 리턴한다.

## 요약

절대 빈 배열이나 컬렉션 대신 null을 리턴하지 마라. API 사용이 더 어렵고, 에러 발생 가능성이 증가한다. 또한 성능 증가 이점도 없다.
