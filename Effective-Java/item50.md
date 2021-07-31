## ITEM 50: 필요하면 defensive 복사를 하라.

Java는 C나 C++ 에 비해 안전한 언어라고 할수 있다. 하지만 우리가 작성한 클래스의 클라이언트에서 의도적이든, 실수로든 불변성을 파괴할 수 있음을 명심해야 한다.

### 첫번째 가능한 공격

아래 코드는 클라이언트가 불변성을 파괴할 수 있는 예시이다.

``` java
public final class Period {
  private final Date start;
  private final Date end;
  
  public Period(Date start, Date end) {
    if (start.compareTo(end) > 0)
      throw new IllegalArgumentException(start+"after"+end);
    this.start = start;
    this.end = end;
  }
  
  public Date start() {
    return start;
  }
  
  public Date end() {
    return end;
  }
  //... 나머지 생략
}
```

여기서 문제점은 ```Date``` 타입이 mutable 하다는 점이다.

``` java
Date start = new Date();
Date end = new Date();
Period p = new Period(start,end);
end.setYear(78); // 여기서 p의 내부 필드값을 변경해버렸다!
```

Java 8에서 이러한 문제를 해결하는 방법은 immutable type인 ```Instant``` (또는 ```Local-DateTime``` 또는 ```ZonedDateTime```) 를 대신 사용하는 것이다.  사실상 ```Date```는 더이상 사용되어서는 않된다.

하지만 이러한 API를 사용하지 않고도 보호하는 방법이 있다. 바로 생성자에서  mutable parameter에 대한 defensive copy를 생성해서, ```Period``` 인스턴스의 구성 필드로 사용하는 것이다.

바로 다음처럼 말이다.

``` java
public Period(Date start, Date end) {
  this.start = new Date(start.getTime());
  this.end = new Date(end.getTime());
  
  if (this.start.compareTo(this.end) > 0) 
    throw new IllegalArgumentException(this.start+"after"+this.end);
}
```

여기서 Date 객체를 복사한다음에 복사된 객체를 대상으로 validation을 해야한다. 그렇지 않으면, TOCTOU 공격에 취약해진다.

>TOCTOU attack : 파라미터가 체크되는 시간과 복사되는 시간 사이에 다른 스레드에서 파리미터값에 변경을 가하는것을 말한다.

그리고 여기서 파림터에 파라미터 타입의 subclass 타입이 들어올 수 있는 경우, ```clone``` 함수를 사용해서는 않된다. 왜냐하면 해당 subclass에서 레퍼런스를 저장할 수 있기 때문이다. 

### 두번째 가능한 공격

여전히 ```Period``` 의 accessor 함수를 통해서 내부 값을 변경할 수 있다.

``` java
Date start = new Date();
Date end = new Date();
Period p = new Period(start,end);
p.end().setYear(78);
```

이를 방어하기 위해서, accessor에서 defensive copy를 반환하도록 하면된다.

``` java
public Date start() {
  return new Date(start.getTime());
}

public Date end() {
  return new Date(end.getTime());
}
```

### Defensive copy를 하지 않아도 좋은 경우

1. 만약 클래스가 클라이언트가 절대 자신의 내부 값을 변경하지 않으리라고 확신할 수 있다면 (아마 클래스와 클라이언트가 같은 패키지에 있는 경우), 클래스 문서에 호출 부분에서 절대 영향받는 파라미터를 수정해서는 않된다고 명시해야 한다.
2. 클래스의 불변성이 파괴되더라도, 하나의 클라이언트에게만 타격이 있는 경우. 예를들며 wrapper class pattern을 들수 있다. 예를들어, wrapper class는 wrapped class의 불변성을 파괴할 수 있지만, 오직 자신만이 영향을 받는다.

## 요약

만약 클래스에 클라이언트로 부터 값을 읽거나, 클라이언트에게 리턴하는 mutable component가 있다면, 클래스는 이 component들을 defensive copy 해야한다. 만약 클라이언트를 신뢰할 수 있거나, copy 비용이 용납되지 않는경우에는 문서로 해당 component는 수정되어서는 않된다고 명시해야 한다. 



