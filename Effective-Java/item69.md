# Item69 : 예외는 진짜 예외 상황에만 사용하라

다음 코드를 보자

*  예외를 완전히 잘못 사용한 예

```java
try {
	int i = 0;
	while(true)
		range[i++].climb();
} catch (ArrayIndexOutOfBoundException e) {
	...	
}
```

위 코드는 전혀 직관적이지 않기 때문에 이렇게 코드를 작성하면 안된다는 것을 보여준다. 이 코드는 배열의 원소를 순회하는데, 무한 루프를 돌다가 배열의 끝에 도달해 ArrayIndexOutOfBoundsException을 발생 시켜 끝내는 코드이다.

이 코드를 다음과 같이 표준적인 관용구대로 작성했다면 모든 자바 프로그래머가 이해했을 것이다.

* 올바른 예제

```java
for (Mountion m : range)
	m.climb();
```

그런데 왜 예외를 써서 루프를 종료시키려 했던 것일까? 그 이유는 다음과 같다.

> JVM은 배열에 접근할 때마다 경계를 넘지 않는지 검사하는데, 이 검사를 줄이기 위해 다음과 같이 코드를 구성했다.
>
> 즉 일반적인 반복문도 배열 경계에 도달하면 종료되지만, 예외 구문을 써가며 성능을 높이려고 한 것이다.

하지만 이 코드는 세 가지 면에서 잘못된 추론을 했다.

1. 예외는 예외 상황에 쓸 용도로 설계되었으므로 JVM 구현자 입장에서는 명확한 검사만큼 빠르게 만들어야 할 동기가 약하다.
2. 코드를 try-catch 블록 안에 넣으면 JVM이 적용할 수 있는 최적화가 제한된다.
3. 배열을 순회하는 표준 관용구는 앞서 걱정한 중복 검사를 수행하지 않는다. JVM이 알아서 최적화해 없애준다.

예외를 사용한 반복문의 해악은 코드를 헷갈리게 하고 성능을 떨어뜨리는 데서 끝나지 않는다. 심지어 제대로 동작하지 않을 수도 있다. 반복문 안에 버그가 숨어 있다면 흐름 제어에 쓰인 예외가 이 버그를 숨겨 디버깅을 훨씬 어렵게 할 것이다.



따라서 정리하자면 **예외는 오직 예외 상황에서만 써야 한다. 절대로 일상적인 제어 흐름용으로 쓰여선 안 된다. 표준적인 관용구를 사용하고 성능 개선 목적으로 과하게 머리 쓴 기법은 자제하라.** 실제로 성능이 좋아지더라도 자바 플랫폼이 꾸준히 개선되고 있으니 최적화로 얻은 상대적인 성능 우위가 오래가지 않을 수 있다.

이 원칙은 API 설계에도 적용된다. **잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 한다.** 특정 상태에서만 호출할 수 있는 상태 의존적 메소드를 제공하는 클래스는 상태 검사 메소드도 함께 제공해야 한다. 다음 예를 보자

* iterator를 사용한 코드

```java
// Iterator의 next(), hasNext()는 상태 의존적 메소드와 상태검사 메소드에 해당
for (Iterator<Foo> i = collection.iterator(); i.hasNext();) { 
	Foo foo = i.next();
	...
}
```

Iterator가 hasNext 를 제공하지 않았다면 이 일을 클리이언트가 대신했을 것이다.

* iterator가 hasNextI()를 제공 안 했을 경우

```java
try {
	Iterator<Foo> i = collection.iterator();
	while(true) {
		Foo foo = i.next();
		...
	}
} catch (NoSuchElementException e) {
	...
}
```

위 코드를 사용하면 장황하고 헷갈리며 속도도 느리고, 엉뚱한 곳에서 발생한 버그를 숨기기도 한다. 그러니 hasNext를 사용해 iterator를 사용하자.

또한 상태 검사 메소드 대신 사용할 수 있는 선택지도 있다. 올바르지 않은 상태일 때 빈 옵셔널 혹은 null 같은 특수한 값을 반환하는 방법이다. 상태 검사 메소드, 옵셔널, 특정 값 중 하나를 선택하는 지침 몇 가지를 소개하겠다.

1. 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 상태가 변할 수 있다면 옵셔널이나 특정 값을 사용한다. 상태 검사 메소드와 상태 의존적 메소드 호출 사이에 객체의 상태가 변할 수 있기 때문이다.
2. 성능이 중요한 상황에서 상태 검사 메소드가 상태 의존적 메소드의 작업 일부를 중복 수행한다면 옵셔널이나 특정 값을 선택한다.
3. 다른 모든 경우에 상태 검사 메소드 방식이 조금 더 낫다고 할 수 있다. 가독성이 더 좋고 잘못 사용했을 때 발견하기가 쉽다. 상태 검사 메소드 호출을 깜빡 잊었다면 상태 의존적 메소드가 예외를 던져 버그를 확실히 잡을 것이다. 반면 특정 값은 검사하지 않고 지나쳐도 발견하기 어렵다.