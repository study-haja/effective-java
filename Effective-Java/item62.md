# ITEM 62 : 다른 타입이 적절한곳에 String을 피하라.

보통 파일이나 네트워크로 부터 데이터가 넘어오면, String 형태의 데이터인 경우가 많다. 하지만 이를 그대로 String 변수에 저장하기 보다 적절한 자료형으로 저장하는것이 좋다.

## String은 aggregate type을 표현하기 에는 부적절하다.

``` java 
String compoundKey = className + "#" + i.next()
```

위 처럼 표현하면 결국, className을 뽑아내기 위해서는 String을 파싱해야한다.

그보다는 item 24에서 배웠듯이 ```static class``` 를 사용해서 표현하는것이 좋다.

``` java
private static class CompoundKey {
  String className;
  String next;
  public CompoundKey(String className, String next) {
    this.className = className;
    this.next = next;
  }
}
```

## 문자열은 권한을 표현하기에 적절하지 못하다.

``` java
public class ThreadLocal {
    private ThreadLocal() {} //객체 생성 불가
    
    // 현 스레드의 값을 키로 구분해 저장한다.
    public static void set(String key, Object value);
    
    // (키가 가르키는) 현 스레드의 값을 반환한다.
    public static Object get(String key);
}
```

> ThreadLocal 은 하나의 쓰레드에서 실행되는 코드가 동일한 객체를 사용할 수 있도록 한다. 주요 용도는 아래와 같다.
>
> - 사용자 인증정보 전파 - Spring Security에서는 ThreadLocal을 이용해서 사용자 인증 정보를 전파한다.
> - 트랜잭션 컨텍스트 전파 - 트랜잭션 매니저는 트랜잭션 컨텍스트를 전파하는 데 ThreadLocal을 사용한다.
> - 쓰레드에 안전해야 하는 데이터 보관

위 방식의 문제점은 두가지 문제점이 있다.

- 여러 클라이언트가 같은 String을 key로 사용할 수 있어서, 에러를 발생시킬 수 있다.
- 보안상, malicious client가 의도적으로 다른 클라이언트와 같은 String을 전달해서, 데이터를 엿볼 수 있다.

이를 계선하기 위해서 ```Key``` 라는 클래스를 생성한다.

``` java
public class ThreadLocal {
    private ThreadLocal() {} //객체 생성 불가
    
    public static class Key {
        key() {}
    }
    
    //위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
		return new Key();
    }
    
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

이 시점에 top-level 클래스는 아무기능을 해주지 못하므로, nested class를 ```ThreadLocal```로 이름을 변경할 수 있다.

``` java
public final class ThreadLocal { 
  public ThreadLocal();
  public void set(Object value); 
  public Object get();
}
```

이렇게 하면 Key는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라 그 자체가 스레드 지역변수가 된다.

이 API는 get으로 얻은 Object를 실제 타입으로 타입 캐스팅 해야 해서 타입안전하지 않다.

이를 제너릭으로 타입 안전하게 만들 수 있다.

``` java
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```

## 요약

- 더 적합한 데이터 티입이 있거나 새로 작성할 수 있다면, 문자열을 쓰지 말자
- 문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고 오류 가능성도 크다.
- 문자열을 잘못 사용하는 흔한 예로는 기본 타입, 열거 타입, 혼합 타입이 있다.