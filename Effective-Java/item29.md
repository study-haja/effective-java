# 이왕이면 제네릭 타입으로 만들라

JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운편이지만, 제네릭 타입을 새로 만드는 일은 쉽지는 않다. 다음 예를 보자.

* Object 기반 스택

```java
public class Stack {
  private Object[] elements; // Object 선언
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY]; // Object로 생성
  }
  
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }
  
  public Object pop() { // Object 사용
    if (size == 0)
      throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
  }
  
  public boolean isEmpty() {
    return size == 0;
  }
  
  private void ensureCapacity() {
    if (elements.length == size) {
      elements = Arrays.copyOf(elements, 2 * size + 1);
    }
  }
}
```

해당 Stack 클래스는 Object 기반으로 만들어졌다. 하지만 Object는 클라이언트 스택에서 꺼낸 객체를 매번 형변환을 해야하는데 런타임 오류가 날 위험이 있어 제네릭 타입으로 바꾸는것이 좋아보인다.

<br>

## 제네릭 클래스로 만드는 방법

일반 클래스에서 제네릭 클래스로 만드는 방법은 다음과 같다.

> 1. 클래스 선언에 타입 매개변수를 추가한다.

위 코드에서는 스택이 담을 원소의 타입 하나만 추가하면 된다. 이때 타입은 보통 Element인 `E`를 사용한다. 그런다음 코드에 쓰인 Object를 적절한 타입 매개변수로 바꾸고 컴파일하면 된다.

* 제네릭 스택 - 타입 매개변수 추가

```java
public class Stack<E> {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    elements = new E[DEFAULT_INITIAL_CAPACITY]; // 오류 발생
  }
  
  public void push(E e) {
    ensureCapacity();
    elements[size++] = e;
  }
  
  public E pop() { 
    if (size == 0)
      throw new EmptyStackException();
    E result = elements[--size];
    elements[size] = null;
    return result;
  }
  
  ...
}
```

<br>

> 2. 타입 매개변수 변경으로 오류나 경고가 나면 알맞게 수정한다.

모든 Object 타입을 E라는 타입 매개변수로 변환했다. 그리고 컴파일 한 결과 하나의 컴파일 오류가 발생하는데, E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다. 배열을 사용하는 코드를 제네릭으로 만들려 할 때는 이 문제가 항상 나타날 것인데 해결책은 다음과 같다.

1. 첫번째 제네릭 배열 생성을 금지하는 제약을  대놓고 우회하는 방법이다.

Object 배열을 생성한 다음 제네릭 배열로 형변환해보자. 이 방법은 오류 대신 경고를 내보낼 것이며 일반적으로 타입이 안전하지 않지만, 여기에서만큼은 우리가 실제로 증명할 수 있다. 해당 배열의 element들은 private 필드에 저장되고 클라이언트에 반환되거나 다른 메서드에 전달되는 일이 전혀 없다. 또한 push 메소드를 통해 배열에 저장되는 원소의 타입은 항상 E이다. 그렇기에 이 비검사 형변환은 확실히 안전하다고 판단할 수 있다.

따라서 이렇게 안전하다고  판단되면 @SuppressingWarnings을 통해 경고를 숨기고 주석을 남겨놓는다. 예를 들면 다음과 같다.

*  배열을 사용한 코드를 제네릭으로 만드는 방법 1

```java
// 배열 elements는 pssh(E)로 넘어온 E 인스턴스만 담는다.
// 타입 안정성이 보장되지만, 런타임 타입은 E[]가 아니라 Object[] 이다.
@SuppressWarnings("unchecked")
public stack() {
	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

2.  elements 필드의 타입을 E[]에서  Object[]로  바꾸는 것이다.

이렇게 하면 첫번째 오류와 다르게 나타나는데, 배열이 반환한 원소를 E로 형변환하면 오류 대신 경고가 뜬다. E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다. 하지만 이번에도 우리가 직접 증명하여 경고를 숨기면 된다. 다음 예를 한번 보자

*  배열을 사용한 코드를 제네릭으로 만드는 방법 2

```java
public E pop() {
  if (size == 0)
    throw new EmptyStackException();
  
  @SuppressWarnings("unchecked") E result = (E) elements[--size]; // 위 변수가 Object[]로 선언할 경우
  
  elemnents[size] = null;
  return result;
}
```

이 두가지 방법 다음과 같이 나름대로 장단점이 존재한다. (서로 정반대의 장단점을 가지고 있다.)

첫번째 장점은 다음과 같다.

1. 가독성이 좋다. - E[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실히 어필한다.
2. 코드가 짧다. - 보통 제네릭 클래스라면 코드가 이곳저곳에서 이 배열을 자주 사용할 것이다.
3. 형변환시 배열을 한번만 사용해 주면 된다.

하지만 단점은 다음과 같다.

1. 배열의 런타임 타입이 컴파일타임 타입과 달리 힙 오염을 일으킨다. - 위 예제에스는 오염이 되지 않지만 오염 될 가능성이 있어 조심해야한다.

<br>

## 정리

지금까지 설명한 Stack 예는 이전 아이템인 "배열보다는 리스트를 우선하라"에 모순되어 보인다. 사실 제네릭 타입 안에서 리스트를 사용하는게 항상 가능하지도, 꼭 더 좋은 것도 아니다. 자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을  사용해 구현해야 한다. 또한 HashMap 같은 제네릭 타입은 성능을 높일  목적으로 배열을 사용하기도 한다.

Stack 예처럼 대다수의 제네릭  타입은 타입 매개변수에 아무런 제약을 두지 않는다. 단 제네릭 타입의 근본적인 문제로 기본 타입은 사용할 수 없어 박싱된 기본 타입을 사용해야 한다.

그리고 타입 매개변수에 제약을 두는 제네릭 타입도 있다. 다음 예를 보자

* Java.util.concurrent.DelayQueue

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

위 클래스는 `<E extends Delayed>`를 통해 Delayed 하위타입만 받는다는 뜻이다. 이렇게 사용하여 DelayQueue뿐만 아니라 DelayQueue를 사용하는 클라이언트는  DelayQueue의 원소에서 곧바로 Delayed 클래스의 매서드를 호출할 수 있다. 이를 한정적 매개변수라 하며 ClassCastException을 걱정할 필요가 없다.