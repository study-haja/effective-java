## ITEM 44: 표준 FUNCTIONAL INTERFACE 사용을 선호하라.

```LinkedHashMap``` 클래스에 있는 ```removeEldestEntry``` 함수를 오버라이딩 해서, 클래스를 캐시로 사용할 수 있다. 이 함수는 put 메소드가 호출될때마다 호출이 되고, true를 반환할 경우 가장 오래된 데이터를 삭제되도록 구현이 되있다.

``` java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
  return size() > 100;
}
```

위 방법 대신 람다를 사용해서 ``LinkedHashMap`` 의 static factory or constructor에 function object를 전달해서 구현하는 것이 낫다.

``` java
@FunctionalInterface interface
  EldestEntryRemovalFunction<K,V> {
  boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

위 방식은 동작하지만, 사용하면 않된다. 왜냐하면 이미 이 목적에 부합하는 인터페이스가 존재하기 때문이다.

```java.util.function``` 패키지는 이미 많은 functional interface를 포함하고 있다. 만약 이 패키지에 있는 표준 functional interface로 해결이 된다면, 굳이 새로운 인터페이스를 정의해서는 안된다.

다음의 기본 인터페이스들만 기억하면 된다. 아래는 object reference에 대해서 동작한다.

- Operator  : return 타입과 argument 타입이 같을 경우 사용
- Predicate : boolean 타입을 입력으로 받아서, boolean 타입을 리턴
- Function : argument, return 타입이 다른 경우
- Supplier : 0 argument를 받아서 값을 리턴
- Consumer : argument를 받아서 return 하지 않음.  

그리고 각각의 functional interface에는 primitive type을 지원하도록 설계되어있다. 그렇기 때문에 boxed primitive를 가진 기본 functional interface보다는 primitive functinal interface를 사용하는것이 좋다. (ITEM 61에서 보겠지만, 성능상 더 이점이 있다.)

## 전용 함수형 인터페이스를 정의하는 경우

표준 함수 인터페이스에서 필요한 기능을 제공해주지 못하는 경우에 직접 정의해야 한다. 예를 들면, 세개의 인자를 받는 Predicate 또는 exception을 던지는 경우가 그렇다.

또한 다음 조건에서 하나 이상을 만족하면, 표준 함수 인터페이스가 아닌 전용 함수형 인터페이스를 구현하는 것이 좋다.

- 자주 쓰이며, 이름 자체가 용도를 명확하게 설명해줌.
- 반드시 따라야 하는 규약이 있는 경우
- 유용한 디폴트 메소드를 제공할 수 있는 경우.

대표적인 예로 ToIntBiFunction<T,U> 를 사용할 수 있음에도 불구하고, Comparator<T>를 별도로 구현이 된것을 들수 있다.

## 전용 함수형 인터페이스를 구현할때 주의할 점

반드시 ```@FunctionalInterface``` 어노테이션을 붙여야 한다. 이 어노테이션을 붙임으로써 다음의 목적을 달성한다.

1. 해당 인터페이스가 lambda를 활성화하기 위해서 사용됨을 알려준다.
2. 인터페이스에 하나의 추상 메소드만이 존재하도록 보장한다. (컴파일러가 체크해준다.)

## 요약

```java.util.function.Function``` 에 있는 표준 함수형 인터페이스를 사용하되, 전용 함수형 인터페이스를 사용했을때 더 좋은 경우가 있는지 주의해서 살펴볼 필요가 있다.

