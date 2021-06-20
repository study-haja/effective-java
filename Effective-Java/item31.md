# Item31: 한정적 와일드카드를 사용해 API 유연성을 높이라

매개변수화 타입은 불공변이다. 즉 서로 다른 타입 예를들어 `String`, `Object`이 있을 때 `List<String>` 와 `List<Object>`은 서로 하위 타입도 상위 타입도 아니라는 얘기다.

좀 더 쉽게 말하자면, `List<Object>`는 어떤 객체든 넣을 수 있지만 `List<String>`은 문자열만 넣을 수 있다. 즉 `List<String>`은 `List<Object>`가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다.

하지만 코딩을 하다보면 유연한 방식이 필요할 때가 있다. 다음 예를 보자

* Stack 예제 - 와일드 카드 사용 x

```java
public class Stack<E> {
	public Stack() {....}
	public void push(E e) {...}
	public E pop() {...}
	public boolean isEmpty() {...}
  
  public void pushAll(Iterable<E> src) { // 매개변수 타입이 불공변
    for (E e: src) {
      push(e);
    }
  }
  
}
```

pushAll 메소드는 컴파일은 되지만 완벽하지 않다. Iterable src의 원소 타입이 스택의 원소 타입과 일치하면 잘 작동하지만, `Stack<Number>` 로 선언후 Integer value를 가지는 `Iterable<Integer>`를 넣으면 오류가 발생한다. 예를 들면 다음과 같다.

* 오류 예제

```java
드Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);
```

그 이유는 매개변수화 타입이 불공변이기 때문이다. 

다행이 자바에서 이런상황에 대처할 수 있는 한정적 와일드카드라는 타입을 지원한다. pushAll의 입력 매개변수 타입을 E의 Iterable이 아니라 E의 하위 타입의 iterable로 변경해 오류를 해결할 수 있다. 다음 예가 그렇다.

*  한정적 타입 매개변수를 사용해 해결한 코드 - <? Extends E>

```java
public class Stact<E> {
  ...
    
  public void pushAll(Iterable<? extends E> src) { // 한정적 타입 매개변수를 사용해 해결
    for (E e : src)
      push(e);
  }
}
```

이렇게 확장하여 E를 포함한 하위 타입까지 받기 때문에 문제를 해결할 수 있다. 또 다른 예를 한번 보자.

* 와일드 카드를 사용하지 않은 popAll 메소드

  ```java
  public void popAll(Collection<E> dst) {
  	while (!isEmpty()) {
  		dst.add(pop());
  	}
  }
  ```

이 메소드도 위와 같이 컴파일은 되지만 오류가 발생할 수 있다. 다음 예가 그렇다.

* `Stack<Number>`의 원소를 Object용 컬렉션으로 옮기는 메소드

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> obejcts = ...;
numberStack.popAll(objects);
```

해당 코드를 컴파일하면 `Collection<Object>는 Collection<Number>의 하위타입이 아니다` 라는 오류가  발생한다. 위와 비슷한 문제라 볼 수 있다. 따라서 이는 다음과 같이 해결 할 수 있다.

* 한정적 타입 매개변수를 사용해 해결한 코드 - <? super E>

```java
public void popAll(Collection<? super E> dst) {
  while (!isEmpty()) {
		dst.add(pop());
	} 
}
```

이렇게 상속하여 E를 포함한 상위 타입까지 매개변수로 받을 수 있어 해결 할 수 있다.

 따라서 정리하면 다음과 같다. **유연성을 극대화 하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라!** 이다. 한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다. 타입을 정확히 지정해야 하는 상황이므로, 이때는 와일드카드 타입이 아닌 E와 같은 매개변수 타입으로 설정해야 한다. 



## 와일드 카드 사용 공식 - PECS

> 펙스(PECS) : producer-extends, consumer-super

다음 팩스 공식을 알아두면 언제 와일드카드를 사용해야 할지 알 수 있다. 매개변수화 타입이 T가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<? super T>`를 사용하면 된다. 여기서 생산자란 타입 매개변수를 사용하는 클래스 인스턴스를 생산할 때이고, 소비자는 타입 매개변수를 사용하는 클래스 인스턴스를 소비할 때를 말한다. 

Stack을 예를 들면 pushAll의 src매개변수는 Stack이 사용할 E 인스턴스를 생성한다고 보면되고, popAll의 dst 매개변수는 Stack으로부터 E 인스턴스를 소비한다고 보면 된다. 

이 공식을 기억해두고, 다른 예를 한번 보자.

* Chooser 생성자

```java
public Chooser(Collection<T> choices)
```

이 생성자로 넘겨지는 choices 컬렉션은 T 타입의 값을 생산하기만 하니 T를 확장하는 와일드카드 타입을 사용해 선언하면 좋다. 즉 다음과 같다.

* Chooser 생성자 - 한정적 와일드 카드 적용 모습

```java
public Chooser(Collection<? extends T> choices)
```

이렇게 변경하면 실질적으로 `Chooser<Number>` 에 `List<Integer>`를 넘길 수 있게 된다.

또 다른  예를 보면 다음과 같다.

* Union 메소드

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```

s1, s2 둘다 E의 생산자이니 PECS 공식에 따라 다음과 같이 바꿔야 한다.

* Union 메소드 - 한정적 와일드 카드 적용 모습

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

이렇게 적용하면 다음과 같은 코드는 멀쩡히 컴파일이 된다.

* Union 메소드 변경후 구현 로직 

```java
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0 4.0, 6.0);
Set<Number> numbers = union(integers, doubles);
```

이렇게 사용함으로써 사용자는 와일드카드 타입이 사용되었다는 의식조차 못할 것이다. **만약 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 크다.**

그리고 자바 7에서 이와 같은 코드를 사용한다면 오류를 볼 것인데 명시적 타입 인수를 사용해서 타입을 알려 해결해주면 된다. 예를 들면 다음과 같다.

* 자바 7까지의 명시적 타입 인수 사용

```java
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

* 번외 - 매개변수와 인수 차이

```
매개변수는 메서드 선언에 정의한 변수
인수는 메소드를 호출 시 넘기는 실제 값

ex) 
void add(int value){...}, class<T> { ... }, Set<Integer> = ...;
여기서 add의 value는 매개변수, class의 T는 타입 매개변수, Set의 Integer는 타입 인수
```

이번에는 max 메소드를 한번 보자.

* max 메소드와 와일드 카드로 다듬어진 모습

```java
// 다듬어지기전 모습
public static <E extends Comparable<E>> E max (List<E> list)

// 다듬어진 후 모습
public static <E extends Comparable<? super E>> E max (List<? extends E> list)
```

PECS 공식에 따라 입력 매개변수에서는 E 인스턴스를 생산하므로 원래의 `List<E>`를 `List<? Extends E>` 로 변경했다. 다음으로 타입 매개변수 E에서 이전에는 `Comparable<E>`를 확장한다고 정의했는데, 이때 `Comparable<E>`는 E 인스턴스를 소비한다. 따라서 `Comparable<E>`를 `Comparable<? super E>`로 대치했다. 

즉 Comparable은 언제나 소비하므로 `Comparable<? super E>`를 사용하는 것이 좋다. 

그렇다면 다음과 같이 복잡하게 수정함으로써 얻는 이득이 무엇일까? 다음 예를 보자.

```java
private List<ScheduledFuture<?>> scheduledFutures = ...;
```

수정 전의 max는 위와 같은 리스트를 처리 할 수 없었다. 그 이유는 ScheduledFuture가 `Comparable<ScheduledFuture>`를 구현하지 않았기 때문이다. 

``Comparable<E> -> Delayed -> ScheduledFuture<V> `` 이러한 형식으로 구성 되어있어 ScheduledFutrue의 인스턴스는 다른 ScheduledFuture 인스턴스뿐 아니라 Delayed 인스턴스와도 비교할 수 있어 수정 전 max가 이 리스트를 거부하는 것이다. 더 일반화 하자면 Comparable을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드가 필요하다.



## 그 외

타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다. 예를 들어 주어진 리스트에서 명시한 두 인덱스의 아이템들을 교환하는 정적 메소드를 두 방식 모두로 정의해 보자.

* 타입 매개변수로 정의

```java
public static <E> void swap(List<E> list, int i, int j);
```

* 와일드카드로 정의

```java
public static void swap(List<?> list, int i, int j);
```

만약 public API로 사용해야 한다면 두번째 방식이 낫다. 그 이유는 어떤 리스트든 이 메소드에 넘기면 명시한 인덱스의 원소들을 교환해 줄 것이기 때문이다. 또한 신경 써야 할 타입 매개변수도 없다.

따라서 기본 규칙은 이렇다. **메소드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하면 된다.** 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.

하지만 두 번째 swap 선언에는 문제가 하나 있는데, 다음 예를 보자

* 직관적으로 구현한 코드가 컴파일되지 않는 문제

```java
public static void swap(List<?> list, int i, int j) {
	list.get(i, list.set(j, list.get(i)));
}
```

컴파일이 되지 않는 이유는 와일드카드 타입으로 사용된 매개변수는 null외에는 어떤 값도 넣을 수 없기 때문이다. 다행히 이러한 문제는 와일드카드 타입의 실제 타입을 알려주는 private 도우미 메소드를 사용해 해결할 수 있다.

* private 도우미 메소드

```java
public static void swap(List<?> list, int i, int j) {
	swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```

이렇게 실제 타입을 알아내는 도우미 메소드에서 `List<E>` 임을 알고 있기 때문에, 컴파일러에서 값을 넣다 빼도 값이 변하지 않고 안전하다고 판단해 오류를 일으키지 않는다. 

와일드카드 타입과 타입 매개변수를 사용해 다소 복잡하게 구현했지만 덕분에 외부에서는 와일드카드 기반의 코드를 알 필요 없이 자유롭게 사용할 수 있다. 즉 사용자들은 복잡한 코드를 모른채 혜택을 누릴 수 있을 것이다.
