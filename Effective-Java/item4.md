## ITEM 4 : Private 생성자로 Non-Instantiability를 강요하라. (객체 생성을 막아라.)

Utility Class의 객체 생성을 막기 위해서는 private constructor를 선언해서 default constructor가 생기지 않도록 해야한다.

``` java
public class UtiliyClass { 
  private UtilityClass () { 
    throw new AssertionError(); 
  }
}
```

abstract class를 만듬으로써, 객체 생성을 막을 수도 있지만, 여기서 문제가 있다. 

1. 다른 클래스가 이를 상속해서 객체를 생성할 수 있다.
2.  사용자도 상속을 해야하는 class로 오해할수 있다.

### Private 생성자의 부작용

> 다른 클래스가 상속할 수 없다. (자식 클래스가 부모 클래스의 생성자에 접근 할 수 없기 때문)

