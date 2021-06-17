# ITEM 30 : 제너릭 함수를 선호하라.

위 코드는 컴파일은 되지만 타입 안전성을 보장해주지 못한다.
``` java
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

 다음은 제너릭 함수로 수정한 것이다.

``` java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}

public static void main(String[] args) {
  Set<String> guys = Set.of("Tom","Dick","Harry");
  Set<String> stooges = Set.of("Larry","Moe","Curly");
  Set<String> aflCio = union(guys,stooges);
  System.out.println(aflCio);
}
```

위 함수는 타입 안전성을 보장해주며 사용하기도 간편하다.

## 제너릭 싱글턴 팩토리 패턴

immutable 하지만 많은 다른 타입들로 적용될 수 있는 객체를 생성할 필요가 있을 때가 있다. 제너릭은 erasure에 의해서 구현되기 때문에, 모든 요구 파라미터에 대해서 단일 객체를 사용할 수 있다. 하지만 static factory 함수를 작성해서, 각각의 요구되는 타입 파라미터에 대해서 객체를 리턴하기 위한 static 함수를 정의해야 한다. 아래 코드가 예시이다.

``` java
private static final ReverseComparator rcInstance = new ReverseComparator();
public static <T> Comparator<T> reverseOrder() {
  return (Comparator<T>) rcInstance;
}

public static final Set EMPTY_SET = new EmptySet();
public static final <T> Set<T> emptySet() {
  return (EmptySet<T>) EMPTY_SET;
}
```

이와 같은 패턴을 **제너릭 싱글턴 팩토리** 라고 한다.

다음은 제너릭 싱글턴 팩토리 패턴을 사용해서, 제너릭 Identity Function(파라미터로 객체를 전달받고, 수정없이 리턴하는 함수)을 구현한 예이다. 아래에서 보면 각각의 타입 T에 대해서, ```identityFunction``` 을 정의하지 않고 제너릭 함수 하나만 정의한 것을 볼수 있다.

``` java
public class GenericSingletonTest {

    private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;
    private static UnaryOperator<Object> VERBOSE_IDENTITY_FN = new UnaryOperator<Object>() 		{
        @Override
        public Object apply(Object o) {
            return o;
        }
    };

    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
    }

    public static void main(String[] args) {
        String[] strings = {"jute","hemp","nylon"};
        UnaryOperator<String> sameString = identityFunction();
        for (String s  : strings) System.out.println(sameString.apply(s));
        Number[] numbers = {1,2.0,3L};
        UnaryOperator<Number> sameNumber = identityFunction();
        for (Number n : numbers) System.out.println(sameNumber.apply(n));
    }
}
```

```(UnaryOperator<T>) IDENTITY_FN ``` 에서 warning이 발생하는데, 이는 ```UnaryOperator<Object>``` 타입 객체를 모든 T에 대해서 ```UnaryOperator<T>``` 로 캐스팅 할수는 없기 때문이다. 하지만 apply 메소드에서 객체를 입력받아 그대로 리턴해주기 때문에 우리는 항상 성공함을 보장할 수 있다.

## 요약

제너릭 함수는 제너릭 타입과 마찬가지로 다음과 같은 장점이 있다.

- 클라이언트가 인풋 파라미터에 대해서 명시적으로 캐스팅하고 리턴할 필요가 없어지므로, 타입 안전성이 보장되고 사용하기 쉽다.

함수를 작성할때는 항상 타입 캐스팅 없이 사용할 수 있도록 해야하고, 보통 제너릭으로 선언해야 한다. 일반적으로 타입 캐스팅을 해야하는 함수들은 제너릭 함수로 변환해야 한다. 
