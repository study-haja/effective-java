# Item 20 : 추상화 클래스보다 인터페이스를 선호하라.

## Intro

먼저 추상화 클래스의 문제점부터 보자. 추상화 클래스를 상속하는 클래스는 다른 클래스를 상속 받을 수 없다. 즉, 한가지 타입에 종속되어 버린다.

``` java
//TypeA.java
public abstract class TypeA {}

//TypeB.java
public abstract class TypeB {}

//TypeTest.java => 컴파일에러!!
public class TypeTest extends TypeA,TypeB {
}
```

반면 인터페이스는 **mixin** 을 정의하는데 이상적이다. 여기서 말하는 **mixin** 이란 **Mixed In** 의 약자로 어느 한 인터페이스를 이미 구현하고 있는 클래스에서 새로운 인터페이스를 구현하여 새로운 선택적인 기능을 얻게된다는 뜻이다. 추상클래스는 두개 이상 상속할 수 없기 때문에 **mixin**을 정의할 수 없다.

아래 코드에서 보는것처럼, 인터페이스는 다른 인터페이스들을 상속할 수 있다. 만약 추상클래스로 구현했더라면, 이런 경우 구현하기 어렵다.

 ``` java
 public interface Singer {
   AudioClip sing(Song s);
 }
 
 public interface SongWriter {
   Song compose(int chartPosition);
 }
 
 public interface SingerSongWriter extends Singer,Songwriter {
   AudioClip strum();
   void actSensitive();
 }
 ```

## Default Method

Java8 부터는 인터페이스에 default method를 정의할 수 있다.

``` java
public interface TimeClient {
    void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year, int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
  
    static ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                    "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }
		  
    default ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }
}
```

위의 코드에서 ```getZonedDateTime``` 함수가 바로 default method이다. default method는 프로그래머에게 편의를 제공해준다.

## 인터페이스 사용시 주의할 점

1. 인스턴스 필드들을 포함해서는 않된다.
2. public이 아닌 static 변수들을 포함할 수 없다. (어차피 interface에 정의된 멤버들은 public으로 자동으로 선언되고, 다른 access modifier는 컴파일에러 처리된다.)
3. Default Method내에서, Object 함수들(```equals```, ```hashcode```)에 대한 행동을 정의하는 것이 금지된다.

## Abstract Skeletal Implementation 클래스

> <용어> 
> ```Primitive Method``` : 다른 함수를 호출하지 않고, 순수 기능만 있는 함수
> ```Non-Primitive Method ```: 다른 함수를 호출하는 함수

인터페이스에는 ```Primitive Method```를 ```default method``` 형태로 정의하고, 추상 클래스는 인터페이스를 구현하고, ```Non-Primitive ```함수를 구현한다.

다음과 같이 ```템플릿 메소드``` 패턴으로 보통 정의한다.

``` java
public interface Ivending {

    default void start() {
        System.out.println("Start Vending Machine");
    }

    void chooseProduct();

    default void stop() {
        System.out.println("Stop Vending Machine");
    }
  
  	default void process() {
        start();
        chooseProduct();
        stop();
    }
}

public abstract SkeletalCandyMachine implements Ivending {

  	@Override
    public void chooseProduct() {
        System.out.println("Produce diiferent candies");
        System.out.println("Choose a type of candy");
        System.out.println("pay for candy");
        System.out.println("collect candy");
    }
}

```

보통 ```Skeletal Implementation Class``` 를 ```Abstract Interface``` 라고 부르기도 한다. 여기서 Inteface란 추상클래스가 구현하는 인터페이스를 말한다. 예를들어 다음과 같이 네이밍을 붙인다. ``` AbstractCollection, AbstractSet, AbstractList, and AbstractMap``` 

## Skeletal Implementation 구현단계

1. 인터페이스에 어느 메소드가 클라이언트에 의해서 정의가 되어야하는지 연구해서 뽑아내기.
2. 1에서 뽑아낸 Primitive Method를 사용해서 정의할 수 있는 함수들을 default method로 정의해서 구현하기. 여기서 위에서 언급한 인터페이스를 정의할때 주의할점을 기억해야 한다. 여기서 모든 인터페이스 함수가 default method로 구현이 되었다면,  여기서 끝내도 좋다.
3.  2번에서 남아있는 인터페이스 함수를 구현하기위해서, 추상 클래스를 정의하고 인터페이스를 상속해서 나머지 함수들을 구현한다.

예를들어, ```Map.Entry``` 인터페이스의 경우로 살펴보자. 이 인터페이스에서 Primitive Method는 ```getKey``` 와 ```getValue``` 그리고 선택적으로 ```SetValue``` 도 될수 있다. 그리고 이러한 Primitive Method를 위해서 ```equals```,  ```toString``` , ```hashcode``` 또한 정의해야 한다. 이러한 작업은 default method에서 처리할 수 없기 때문에, 우리는 ```skeletal Implementation class``` 에 이를 정의한다. 

``` java
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {

    @Override
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Map.Entry)) return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(),getValue());
    }

    @Override public int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
}

```

## 정리

인터페이스는 여러 구현체를 허용하는 타입을 정의하기에 가장 좋은 방법이다. 인터페이스를 정의할때, Java 8 부터 지원하는 default method를 사용하되 위에서 언급한 주의사항에 위배된다면, ```Skeletal Implementation class```  를 정의해서, 나머지 인터페이스 함수(Object 클래스의 equals, hashcode, toString 함수 + 인스턴스 변수에 의존하는 함수)를 제공해야 한다. 

