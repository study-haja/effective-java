#  Item27:  비검사 경고를 제거하라

제네릭을 사용하다보면 컴파일러 경고가 나타나게 된다. 예를 들어 비검사형 변환경고,  비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고  등 다양하게  맞이할 것이다. 

대부분의  비검사 경고는 쉽게 제거할  수 있다. 다음 예를 보자.

* 비검사 매개변수 경고 - HashSet 선언

```java
Set<Lark> exaltation = new HashSet();
```

컴파일러에서 `-Xlint:uncheck` 옵션을 주면 어디가 잘못됬는지 알려준다. 여기서는 HashSet에 매개변수를 명시하지 않아서 경고가 나타나는건데 `<>` 연산자를 사용하면 경고를 해결할 수 있다. (컴파일러가 올바른 실제 타입 매개변수를 추론해준다.)

<br>

## 제거하기 어려운 경고

하지만 제네릭을 사용할 때 제거하기 어려운 경고도 있다. 하지만 경고를 제거하면 할 수록 그 코드의 안정성이 보장된다. 즉 런타임에서 `ClassCastException`이 발생할 일이 없고, 우리가 의도한 대로 잘  동작하리라 확신 할 수 있다. **그러니 할 수 있는 한 모든 비검사 경고를 제거하자!**

**만약 경고를  제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 `@Suppress Warnings("unchecked")` 어노테이션을 달아 경고를 숨기는 것도 좋다.** 단 타입 안전함을 검증하지 않은 채 경고를 숨기게 되면, 해당 코드는 안정성이 보장받지 못해 ClassCastException이 발생할 수 있고, 그리고 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다.

<br>

## @SuppressWarnings

`@SuppressWarnings` 어노테이션은 개별 지역변수  선언부터 클래스 전체까지 어떤 선언에도 달 수 있다. **하지만 `@SuppressWarnings`은 항상 가능한 한 좁은 범위에 적용을 해야 한다.** 그 이유는 자칫 심각한 경고를 놓칠 수 있으니 클래스 전체에 사용하지 말고 변수나 짧은 메소드등에 선언하는 것이 바람직하다.

한 줄이 넘는 메소드나 생성자에 `@SuppressWarnings`을 달아야 한다면 지역변수에 선언하는 것도 방법이다. 다음 예를 보자

* ArraysList의 toArray 메소드 - 일반

```java
public <T> T[] toArray(T[] a) {
	if (a.length < size) 
    return (T[]) Arrays.copyOf(elements, size, a.getClass()); // 캐스팅 경고 발생
 	System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
    a[size] = null;
  return a;
}
```

해당 코드를 컴파일하면 캐스팅 경고를 받게 될 것이다. 즉  `T[]` 로 캐스팅 했는데 컴파일러는 `Object[] `타입이 필요하다고 경고를 띄울 것이다.

하지만 `@SuppressWarnings`은 선언에만 달 수 있기 때문에 return문에 다는게 불가능 할 것이다. 그렇다고 전체 메소드에 달기에는 범위가 필요 이상으로 넓어져 사용하기에 알맞지 않다. 따라서 다음과 같이 하면 경고를 피할 수 있게  된다.

* ArraysList의 toArray 메소드 - `@SuppressWarnings` 사용시

```java
public <T> T[] toArray(T[] a) {
	if (a.length < size) {
    @SuppressWarnings("unckecked") T[] result =							// 지역변수로 비검사 경고 발생 해결
      (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }
    
 	System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
    a[size] = null;
  return a;
}
```

이와 같이 result라는 지역변수를 두어 `@SuppressingWarnings`을 통해 경고를 해결했다. 또한 범위도 최소로 좁혀 안정성도 보장받게 되었다. 

마지막으로 `@SuppressingWarnings`을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다. 다른 사람이 이 코드를 이해하는데 도움이 되며, 더 중요하게는, 다른 사람이 그 코드를 잘못 수정하여  타입 안전성을 잃는 상황을 만들지 않게 하기 위해서이다. 그러니 주석을 통해 안전한 근거가 떠오르지 않더라도 설명을 적어놓도록 하자.

