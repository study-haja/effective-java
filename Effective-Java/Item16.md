# Item 16 : public class에서 public field 말고 accessor method(접근자 함수) 를 사용하라.

다음과 같은 클래스가 있다고 가정하자.

``` java
public class Point {
  public double x;
  public double y;
}
```

위 클래스는 캡슐화의 이점을 누리지 못한다. API의 변경없이 표현방식을 바꿀수도 없고(예를들어, 위의 변수를 지우고 다른 변수를 정의하면 표현방식이 변경되는 동시에 API또한 변경이 되어 버린다.), 특정 규칙을 강요할수도 없으며(예를들어, 범위를 강제하는 것) 필드에 접근될때 추가적인 작업을 수행할 수도 없다.

따라서 다음과 같이 정의하여, 클래스의 내부표현 방식을 변경할수 있도록 해야한다.

``` java 
public class Point { 
 
  private double x; 
  private double y; 
  
  public Point(double x, double y) { 
    this.x = x; 
    this.y = y; 
  } 
  
  public double getX() { return x; } 
  public double getY() { return y; } 
  public void setX(double x) { this.x = x; } 
  public void setY(double y) { this.y = y; } 
}
```

하지만, 클래스가 package-private 이거나 중첩된 private 클래스인 경우, 데이터 필드들을 public으로 선언해도 상관없다. 클라이언트 코드가 내부표현에 묶이게 된다고 해도, 그 코드는 클래스와 같은 패키지에만 위치하기 때문이다. 즉, 외부 패키지에 있는 코드는 변경될 일이 없다. 중첩된 private 클래스인 경우에도, public 변수에 접근은 바깥 클래스에서만 가능하다(즉, 이 경우에도 같은 패키지에서만 해당 값에 접근할 수 있다). 

Java Platform의 라이브러리 중에, 위 규칙을 위배하는 경우가 있는데, 대표적인 예로 ```java.awt``` 패키지에 있는 ```Point```, ```Dimension``` 클래스를 들수있다. ```item67``` 에서 명시된 것처럼, ```Dimension``` 클래스를 외부에 노출함으로써, 심각한 성능 문제를 야기했다. public class가 필드를 직접적으로 노출하는 일은 절대 없어야 되지만, 그 필드가 immutable인 경우는 덜 나쁘다. 왜냐하면 특정 규칙을 강요할 수 있기 때문이다.

예를들어, 다음 클래스는 각각의 instance가 valid 시간값을 갖게 한다.

``` java
public final class Time { 
  private static final int HOURS_PER_DAY = 24; 
  private static final int MINUTES_PER_HOUR = 60; 
  
  public final int hour; 
  public final int minute; 

  public Time(int hour, int minute) { 
    if (hour < 0 | hour >= HOURS_PER_DAY) throw new IllegalArgumentException("Hour: " + hour); 
    if (minute < 0 | minute >= MINUTES_PER_HOUR) throw new IllegalArgumentException("Min: " + minute); 
    this.hour = hour; 
    this.minute = minute; 
  }

```

위 코드를 보면 hour, minute 변수가 항상 유효한 값을 갖도록 보장함을 알수가 있다.

## 요약

public 클래스는 절대 mutable field들을 노출해서는 안된다. immutable 필드들을 노출하는 경우는 덜 나쁘다고 할수는 있다. 하지만 package private 또는 중첩된 private 클래스에서는 mutable or immutable 필드들을 노출해도 괜찮다.

