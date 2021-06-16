# ITEM 26 : RAW 타입을 쓰지 마라.

하나이상의 타입 파라미터를 가지고 있는 클래스 또는 인터페이스를 **제너릭 클래스 또는 인터페이스** 라고 한다.

모든 제너릭 타입은 타입 파라미터가 없는 제너릭 타입의 이름인 **raw type** 을 정의한다. 예를들어, ```List<E>``` 에 대응하는 raw type은 ```List``` 이다. Raw type은 제너릭 타입 정보가 삭제 된것처럼 동작한다. 주로 제너릭이 생기기 이전의 코드와 호환성을 위해서 존재한다. 자바에 제너릭이 추가되기 이전에는, 전형적인 컬렉션 선언방법이었다. 

``` java
private final Collection stamps = ...;
```

만약 위 처럼 선언한뒤에 ```stamps.add(new Coin(...)); ``` 를 호출하게 되면, 컴파일.런타임 에러가 나지않는다. 하지만 결국 다음과 같이 객체를 읽어들일때 런타임 에러가 난다.

``` java
for (Iterator i = stamps.iterator(); i.hasNest();)
  Stamp stamp = (Stamp)i.next();
```

위처럼 raw type을 사용하게 되면, 문제가 있는 코드에서 멀리 떨어진 코드에서 에러가 발생한다.

만약 아래와 같이 선언한다면, 컴파일 타임에 잘못된 객체 추가 오류를 감지할 수 있다.

``` java
private final Collection<Stamp> stamps = ...;
```

raw type을 사용할 수는 있지만 제너릭이 가진 안전성과 명료성을 잃어버리기 때문에 절대 사용하지 말아야 한다.

그렇다면 왜 raw type을 허용한 것일까? 바로 호환성 때문이다. 제너릭이 생기기 이전에 작성된 코드들이 정말 많기때문에, 이런 코드들도 새로운 버전의 코드들과 호환이 되어야 한다.

임의의 객체를 삽입하기 위해서 ```List``` 를 사용하는건 안되지만  ```List<Object>``` 는 사용해도 괜찮다. 그렇다면 둘의 차이는 무엇일까? 대략적으로 설명하자면, ```List``` 는 제너릭 타입 시스템을 채택하지 않은것이고, ```List<Object>``` 는 명시적으로 컴파일러에게 아무 타입의 객체들을 수용할 수 있어야 한다고 명시해둔 것이다. ```List<Object>``` 는 ```List``` 의 서브 타입이다. 즉 다음과 같은 코드가 가능하다.

``` java
// Map<String,String>, Map
// Set<Integer>, Set
public static void main(String[] args) {
  List<String> strings = new ArrayList<>();
  unsafeAdd(strings,Integer.valueOf(42));
  String s = strings.get(0);
}

private static void unsafeAdd(List list, Object o) {
  list.add(o);
}
```

위의 코드는 컴파일이 되지만, ```List<String>``` 타입 객체에 ```Integer``` 객체를 추가했으므로 런타임 에러가 난다.

 아래와 같이 수정하면 ```unsafeAdd(strings,Integer.valueOf(42));``` 에서  컴파일에러가 나기 시작한다.

``` java
private static void unsafeAdd(List<String> list, Object o) {
  list.add(o);
}
```

## unbounded wildcard 타입

만약 제너릭 타입을 사용하고 싶지만 실제 타입이 무엇있지 모를경우 unbounded wildcard 타입을 사용할 수 있다.

다음 예시를 보자.

```java
public class GenericTest {
    static int numElementsInCommon(Set s1, Set s2) {
        int result = 0;
        for (Object o1 : s1) {
            if (s2.contains(o1)) result++;
        }
        return result;
    }
}
```

위 코드의 문제는 raw type을 사용한다는 것이다. 이를 unbounded wildcard 타입으로 바꾸면 아래와 같다.

``` java
   static int numElementsInCommon(Set<?> s1, Set<?> s2) {
        int result = 0;
        for (Object o1 : s1) {
            if (s2.contains(o1)) result++;
        }
        return result;
    }
```

위 코드와 아래 코드의 차이점은 ```Set``` 타입 객체에는 어떤 아이템도 추가 할수 있지만, ```Set<?>``` 타입 객체에는 null을 제외하고는 어떤 객체도 추가할 수 없다. 읽을때도 또한 타입을 가정해서 객체를 읽을 수가 없다. 이러한 제약을 극복하기 위해서는, 제너릭 메소드(Item 30)이나 bounded wildcard type(Item 31)을 사용할 수 있다.

## Raw type을 사용할 수 있는 몇가지 예외

### 1. 클래스 리터럴에서는 raw type을 사용해야 한다. 
```List.class```, ```String[].class```, ```int.class``` 들은 합법적이며 ```List<String>.class``` 와 ```List<?>.class``` 는 그렇지 않다.

### 2. instanceof 연산자를 사용하는 경우

런타임에 제너릭 타입정보가 지워지기 때문에, ```unbounded wildcard type```을 제외하고 ```parameterized type```에 대해서 ```instaceof``` 연산자를 사용할 수 없다.  다음과 같이 런타임에 제너릭 타입을 체크해야 하는 경우는 아래와 같이 할 수 있다.

``` java
if (o instanceof Set) {
  Set<?> s = (Set<?>)o; //여기서 반드시 wildcard type으로 캐스팅해야한다.
}
```

## 요약

raw type을 사용하는것은 런타임에 exception을 발생시킬 수 있으므로 사용하지 마라. raw type은 원래 제너릭이 도입되기 이전 코드들과의 호환성을 위해서 존재하는 것이다. 
