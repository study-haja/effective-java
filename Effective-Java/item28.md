# ITEM 28 : 배열보다 리스트를 선호하라.

배열은 제너릭 타입과 비교해서 두가지 차이가 있다.

첫번째로 배열은 covariant 하다. 즉, 만약 ```Sub``` 가 ```Super``` 의 서브 타입이라면, ```Sub[]``` 또한 ```Super[]``` 의 서브 타입이 된다는 뜻이다. 반면 제너릭은 invariant 하다 : 두개의 서로다른 타입 ```Type1```, ```Type2``` 가 있을때, ```List<Type1>``` 은 ```List<Type2>``` 의 서브 타입과 슈퍼 타입 모두 될 수 없다.

두번째는 배열은 reified(구체화된) 하다. 이는 배열은 런타임에 그들의 요소 타입을 알아내고 강요할 수 있다는 의미이다.

``` java
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // ArrayStoreException
```

반면 제너릭은 erasure에 의해서 구현된다. 이말은 제너릭이 오직 컴파일 타임에 타입 조건을 강요할 수 있고 런타임에는 요소 타입 정보를 삭제함을 의미한다. 여기서 erasure는 제너릭 코드가 제너릭이 도입되기 전의 코드(legacy)와 호환되도록 해준다. 

위 두가지 차이 때문에 배열과 제너릭은 잘 호환 되지 않는다. 

## 자바에서 제너릭 배열을 허용하지 않는 이유

자바에서 제너릭 배열은 허용되지 않는다. 이유는 typesafe 하지 않기 때문이다. 만약 허용이 된다면, 컴파일러에 의해 생성된 타입 캐스팅이 런타임에 실패하게 된다. 이는 제너릭 타입 시스템이 제공하는 타입 안전성을 보장해주지 않는다.

``` java
List<String>[] stringLists = new List<String>[1]; // (1) List[] stringLists = new List[1];
List<Integer> intList = List.of(42); // (2) true
Object[] objects = stringLists; // (3) 
objects[0] = intList; // (4)
String s = stringLists[0].get(0); // (5) String s = (String) stringList[0].get(0)
```

먼저 1번 라인이 가능하다고 해보자. 2번 라인은 원래 가능하다. 3번 라인은 배열은 covariant 하기 때문에, 가능하다. 4번 라인은 제너릭의 erasure에 의해서  ```List<String>[]``` 는 ```List[]``` 로 변환되고 ```List<Integer>``` 는 ```List``` 이 되므로 가능하다. 여기서 ```List<Integer``` 를 ```List<String>``` 배열에 저장하게 되면서 문제가 발생한다. 5번 라인에서는 ```Integer``` 를 ```String```으로 캐스팅하면서 런타임 오류가 발생한다. 이러한 상황을 방지 하기 위해서, 1번 라인은 컴파일 타임 에러를 발생시켜야 한다.

```E```, ```List<E>```, ```List<String>``` 는 ```non-reifiable type```이라고 부르는데,  직관적으로 말하면 runtime 시 오브젝트의 정보가 compile시 오브젝트의 정보 보다 더 적다는 것을 의미한다. erasure 때문에, 유일한 reifiable parameterized type은 unbounded wildcard type (예 : ```List<?>```, ```Map<?,?>```) 이다. 드물게 사용되긴 하지만 unbounded wildcard type 배열을 만드는 것은 허용된다.

``` java
Set<?>[] setArr = new HashSet<?>[100];
```

### 제너릭 타입의 가변인자

제너릭 타입의 가변인자를 사용하는 다음의 경우를 보자.

``` java
@SafeVarargs
static void test(List<String>... lists) {
  // ...
}

public static void main(String[] args) {
  List<String> l1 = new ArrayList<>();
  List<String> l2 = new ArrayList<>();
  test(l1,l2);
}
```

가변인자 함수를 호출할때마다, 가변인자 파라미터들을 저장하기 위해서 배열이 생성된다. 만약 배열의 요소 타입이 reifiable하지 않다면, 다음과 같은 warning이 발생한다.

>Note: /Users/jaegu/Desktop/EffectiveJava/src/main/java/item26/GenericTest.java uses unchecked or unsafe operations.
>Note: Recompile with -Xlint:unchecked for details.

이럴 경우 @SafeVarargs 를 추가하면 warning을 방지할 수 있다.

## 제너릭 리스트를 사용했을 때의 장점

만약 제너릭 배열 생성 오류나 array type으로 캐스팅 하는 것에서 unchecked cast warning이 발생하면, 가장 좋은 해결방법은 ```List<E>``` 을 사용하는 것이다. 약간의 간결함과 퍼포먼스를 희생하지만 타입 안전성과 상호운용성을 얻을 수 있다.

다음은 제너릭 리스트를 사용하는 코드로 리팩토링하는 과정이다.

``` java
public class Chooser {
  private final Object[] choiceArray;
  
  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }
  
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```

위 코드에서 choose 함수는 Object 타입의 객체를 리턴하므로, 사용하기 위해서 수동으로 타입 캐스팅을 해야하므로 번거롭고 틀린 타입으로 캐스팅시 런타임 에러가 발생할 수 있다.

위 코드를 제너릭을 사용해서 다음과 같이 수정할 수 있다.

``` java
public class Chooser<T> {
  private final T[] choiceArray;
  
  public Chooser(Collection<T> choices) {
    choiceArray = (T[]) choices.toArray();
  }
}
```

위 처럼 할 경우, 동작은 하지만 컴파일러가 타입 안전성을 보장해주지 못한다는 warning을 출력하게 된다. 

unchecked cast warning 을 제거하기 위해서 배열 대신 리스트를 사용하면 된다.

``` java
public class Chooser<T> {
  private final List<T> choiceList;
  
  public Chooser(Collection<T> choices) {
    choiceList = new ArrayList<>(choices);
  }
  
  public T choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceList.get(rnd.nextInt(choiceList.size()));
  }
}
```

위 버전은 더 장황하고 성능상 조금 느릴지도 모른다, 하지만 런타임에 ```ClassCastException``` 을 발생시키지는 않을 것이다. 

## 요약

배열과 제너릭은 매우 다른 타입 규칙을 가지고 있다. 배열은 covariant하고 reified(런타임에 Type Erasure에 의해서 타입이 지워지지 않는다) 하다. 결과적으로 배열은 런타임에 타입 안전성을 제공하지만 컴파일 시간의 타입 안전성은 보장하지 않는다. 반대로 제너릭은 컴파일 시간의 타입 안전성은 보장하지만 런타임 시간에는 보장하지 않는다. 배열과 제너릭은 잘 호환되지 않는다. 따라서 두가지를 섞어 쓰다가 컴파일 에러나 warning을 보게 된다면, 가장 먼저 해야하는 것은 배열을 리스트로 바꾸는 것이다.

