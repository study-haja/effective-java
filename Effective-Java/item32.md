# Item 32 : 제너릭과 가변인자를 현명하게 결합하라.

## 제너릭 가변인자 배열을 사용할때 발생하는 문제

가변인자 함수를 호출할때, 가변인자 파라미터들을 저장하기 위한 배열이 생성되고, 다음과 같은 문제가 발생할 수 있다.

``` java
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList; // Heap pollution
  String s = stringLists[0].get(0); // ClassCastException
}
```

위 코드는 다음의 warning을 발생시킨다.

> warning : [unchecked] Possible heap pollution from parameterized varargs type List<String>

 위 코드에서 볼수 있듯이, 제너릭 가변인자 배열에 값을 저장하는 것은 안전하지 못하다.

## 제너릭 가변인자 함수를 허용하는 이유

실제로 유용하게 사용되는 케이스가 있다. 대표적으로 ```Arrays.asList(T... a)``` , ```Collections.addAll(Collection<? super T> c, T... elements)```, ```EnumSet.of(E first, E... rest)``` 등이 있다. 이 함수들은 타입 안전하다. 이 함수들을 사용하는 클라이언트들에서도 warning이 발생하지 않는데, 그 이유는 Java 7 버전부터 제공하는 ```SafeVarargs``` 어노테이션이 자동으로 warning을 제거해주기 때문이다. ```SafeVarargs``` 어노테이션은 함수의 작성자가 해당 함수가 타입 안전함을 보장해줌을 의미한다.

## 제너릭 가변인자 함수가 타입 안전함을 판단하는 방법

1. 함수가 제너릭 가변인자 배열에 값을 저장하지 않는 경우
2. 제너릭 가변인자 배열에 대한 레퍼런스를 제공하지 않는 경우

여기서 2번 조건이 왜 필요한지 알아보자.

``` java
static <T> T[] toArray(T... args) {
  return args;
}
```

위 함수는 제너릭 가변인자 배열을 리턴하고, 이 배열을 사용하는 측에서 힙 오염을 발생시킬 수 있다.

``` java
public class GenericVarargsTest {

    static <T> T[] toArray(T... args) {
        return args;
    }

    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a,b);
            case 1: return toArray(a,c);
            case 2: return toArray(b,c);
        }
        throw new AssertionError();
    }

    public static void main(String[] args) {
        String[] attributes = pickTwo("Good","Fast","Cheap"); //ClassCastException
    }
}
```

위 함수에서는 toArray 함수에서 ```Object[]``` 타입의 객체를 리턴하게 되고, 이를 그대로 ```pickTwo``` 함수에서 리턴하게 되며 결국 ```String[]``` 타입으로 캐스팅하면서 ```ClassCastException``` 이 발생한다.

그러나 무조건 다른 함수에서 제너릭 가변인자 배열에 접근하는 것이 위험한 것은 아니다. 다음 두가지 예외상황에서는 위험하지 않다.

1. 제너릭 가변인자 배열을 ```@SafeVarargs``` 로 표기된 제너릭 가변인자 함수에게 전달하는 경우
2. 배열의 컨텐츠의 함수를 단지 호출만하는 비가변인자 함수에게 배열을 넘기는 경우

## 제너릭 가변인자 배열의 안전한 사용예시

#### 1. @SafeArgs 사용

``` java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list: lists) result.addAll(list);
    return result;
}
```

```@SafeVarargs``` 를 언제 사용할지에 대한 규칙은 간단하다. 제너릭 가변인자 배열 또는 parameterized 타입 가변인자 배열을 가지는 모든 함수에 대해서 사용하면 된다. 그렇게 함으로써 유저는 컴파일러 warning에서 자유로울 수 있다.

그리고 해당 어노테이션은 오버라이딩 될 수 없는 함수에서만 사용가능하다. 왜냐하면 해당 함수를 재정의한 함수들에서 안전성을 보장해줄 수 없기 때문이다.

#### 2. List 파라미터 사용

``` java
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) result.addAll(list);
    return result;
}
```

List 파리미터를 사용하면, SafeVarargs 어노테이션의 안전성을 보장해줄 필요가 없다. 하지만 **이 방법의 단점은 코드가 약간 장황하고 느리다는 것이다.** 이 방법은 SafeVarargs 함수를 작성하는 것이 불가능할때 사용할 수 있다.

아래는 List 파라미터를 사용한 예시이다.

```java
public class GenericVarargsTest {

    static <T> List<T> pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return List.of(a,b);
            case 1: return List.of(a,c);
            case 2: return List.of(b,c);
        }
        throw new AssertionError();
    }

    public static void main(String[] args) {
        List<String> attributes = pickTwo("Good","Fast","Cheap");
    }
}
```

위 코드는 제너릭만 사용하므로 타입 안전하다.

## 요약

가변인자의 동작은 배열을 기반으로 추상화 되어있어서, 가변인자와 제너릭은 서로 잘 호환 되지 않는다. 비록 제너릭 가변인자 파라미터가 타입 불안전 하지만, 문법상 합법이다. 만약 제너릭(또는 parametrized) 가변인자 파리미터 함수를 작성한다면, 먼저 함수가 타입 안전한지 점검해보고 ```@SafeVarargs``` 를 추가하라.
