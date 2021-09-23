# Item73 : 추상화 수준에 맞는 예외를 던지라

메소드가 저수준 예외를 처리하지 않고 바깥으로 전파해 고수준에서 예외를 처리하는 일이 간혹 있을 것이다. 사실 이는 내부 구현 방식을 들어나기 때문에 윗 레벨 API를 오염시킨다. 다음 릴리스에서 구현 방식을 바꾸면 다른 예외가 튀어나와 기존 클라이언트 프로그램을 깨지게 할 수도 있다.

이 문제를 피하려면 **상위계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던저야 한다.** 이를 예외 번역이라 한다.

* 예외 번역

``` java
try {
	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
  // 추상화 주순에 맞게 번역한다.
  throw new HigherLevelException(...);
}
```

다음은 AbstractSequentialList에서 get 메소드에 수행하는 예외 번역의 예이다.

* AbstractSequentialList에서 수행하는 get 메소드

```java
public E get(int index) {
	ListIterator<E> i = listIterator(index);
	try {
		return i.next();
	} catch (NoSuchElementException e) {
		throw new IndexOutOfBoundsException("인덱스 : " + index); // 고수준으로 예외 번역 (인덱스 초과나 미만 시 발생)
	}
}
```

예외를 번역할 때, 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용하는게 좋다. 예외 연쇄란 문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식이다. 그러면 별도의 접근자 메소드를 통해 필요하면 언제든 저수준 예외를 꺼내 볼 수 있다.

* 예외 연쇄

```java
try {
	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
	// 저수준 예외를 고수준 예외로 실어 보낸다.
	throw new HigherLevelException(cause);
}
```

고수준 예외의 생성자는 상위 클래스의 생성자에 원인을 건내주어 최종적으로 Throwable(Throwable) 생성자까지 건네지게 한다.

* 예외 연쇄용 생성자

```java
class HigherLevelException extends Exception {
	HigherLevelException(Throwable cause) {
		super(cause);
	}
}
```

대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있다. 그렇지 않은 예외라도 Throwable, initCause 메소드를 이용해 원인을 직접 못밖을 수 있다. 예외 연쇄는 문제의 원인을 프로그램에서 접근할 수 있게 해주며, 원인과 고수준 예외의 스택 추적 정보를 잘 통합해준다.



### 결론

**무턱대고 예외를 전파하는 것보다야 예외 번역이 우수한 방법이지만, 그렇다고 남용해서는 곤란하다.** 가능하다면 저수준 메소드가 반드시 성공하도록하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다. 

차선책도 있다. 아래 계층에서의 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에까지 전파하지 않는 방법이 있다. 이 방법은 로깅과 같은 기능을 활용해 기록해 두면 좋다. 이렇게 해두면 클라이언트 코드와 사용자에게 문제를 전파하지 않으면서 프로그래머가 로그를 분석해 대처할 수 있다.