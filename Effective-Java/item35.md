# Item35 : ordinal 메서드 대신 인스턴스 필드를 사용하라

모든 열거 타입은 해당 상수가 몇 번째 위치인지를 반환하는 `ordinal()` 이라는 메소드를 제공한다. 

* java.lang.Enum 추상 클래스의 ordinal 정수형 필드

![img](https://blog.kakaocdn.net/dn/6btrV/btqDw3x2mpV/a8xWx96g3o9boz5hGYxGwk/img.png)

* ordinal 메소드

![img](https://blog.kakaocdn.net/dn/b84rbg/btqDtAYBBk8/iYKQJCE2p9GVmqz7S5gaN1/img.png)

출처: [https://javabom.tistory.com/49](https://javabom.tistory.com/49)

<br>

하지만 이 메소드는 여러 문제점을 야기한다. 다음 예를 보자.

* ordinal 메소드를 잘못 사용한 예

```java
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET,
	SEXTET, SEPTET, OCTET, NONET, DECET;
	
	public int numberOfMusicians() { return ordinal() + 1; }
}
```

이 코드를 보면 동작은 하지만 유지보수하기가 끔찍한 코드다. 이유는 다음과 같다.

1. 상수 선언 순서를 바꾸는 순간 numberOfMusicians가 오작동을 한다.
2. 이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없다.
3. 중간에 값을 비워둘 수 없다.

즉 이러한 문제로 코드가 깔끔하지 못할 뿐 아니라, 쓰이지 않는 값이 많아질수록 실용성이 없다.

그렇다면 어떻게 해결할면 좋을까? 방법은 간단하다. **열거 타입 상수에 연결된 값은 ordinal 메소드로 얻지말고 인스턴스 필드에 저장하면 된다.** 다음 예를 보자

* 인스턴스 필드에 저장한 예

```java
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET,(4) QUINTET(5), // 순서가 바뀌어도 상관 없음
	SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8), // 이미 사용중인 상수가 있어도 사용 가능
	NONET(9), DECET(10), TRIPLE_QUARTET(12); // 중간에 값을 비울 수 있음
	
	private final int numberOfMusicians;
	Ensemble(int size) { this.numberOfMusicians = size; }
	public int numberOfMusicians() { return ordinal() + 1; }
}
```

<br>

### 결론

Enum의 API 문서를 보면 ordinal에 대해 이렇게 쓰여 있다. ***대부분 프로그래머는 이 메소드를 쓸 일이 없다. 이 메소드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다.*** 따라서 이런 용도가 아니라면 ordinal 메소드는 절대 사용하지 말자.



