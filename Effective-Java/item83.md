# Item83 : 지연 초기화는 신중히 사용하라

지연 초기화란 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다. 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 유용하다. 그러나 대부분의 상황에서는 final로 필드를 선언하는 일반적인 초기화가 지연 초기화보다 낫다.

다음 예를 보자

* 지연 초기화 예제

```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
```

위와 같이 synchronized 키워드를 사용해서 초기화를 할 수 있다. 하지만 성능 때문에 정적 필드로 두어 해당 필드를 초기화해야 한다면 다음과 같이 하면 된다.

* 홀더 클래스 예제

```java
private static class FieldHolder {
  // 일반적으로 VM은 오직 클래스를 초기화할 때만 필드 접근을 동기화함
  // 클래스 초기화가 끝난 후에는 VM이  동기화 코드를 제거하여, 그 다음부터는 아무런 검사나 동기화 없이 필드에 접근 -> 동기화를 하지 않으니 성능이 느려질 거리가 없음
  static final FieldType field = computeFieldValue();
}


private static FieldType getField() { 
	return FieldHolder.field; 
}
```

위와 같이 홀더 클래스 관용구를 사용하여 성능을 좀 더 향상 시킬 수 있다. 

인스턴스 필드를 지연 초기화 할 때 성능을 높여 줄 수 있다. 바로 이중 검사 관용구를 사용하면 된다.

* 이중 검사 관용구 예제

```java
private volatile FieldType field;

private FieldType getField4() {
  FieldType result = field;
  if (result != null) // 첫 번째 검사 (락 사용 안 함)
    return result;

  synchronized(this) {
    if (field == null) // 두 번째 검사 (락 사용)
      field = computeFieldValue();
	  return field4;
  }
}
```

위와 같이 구현하면 초기화된 필드에 접근할 때 동기화 비용을 없애준다. 또한 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 volatile로 선언해주면 된다.

정적 필드에도 적용할 수 있지만 굳이 그럴 이유는 없다. 이보다는 지연 초기화 홀더 클래스 방식이 더 낫다.

또한 반복해서 초기화해도 상관없는 인스턴스 필드는 이중 검사에서 두 번째 검사를 생략할 수 있다. 예를 들면 다음과 같다.

* 단일 검사 예제

```java
private volatile FieldType field5;

private FieldType getField5() {
  FieldType result = field5;
  if (result == null)
    field5 = result = computeFieldValue();
  return result;
}
```

모든 쓰레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일 검사의 필드 선언에서 volatile을 삭제해도 좋다. 하지만 어떤 환경에서는 필드 접근 속도를 높여주지만, 중복해서 초기화가 일어날 수 있다.

