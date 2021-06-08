# ITEM 22 : 인터페이스는 오직 타입을 정의하기 위해서 사용하라.

인터페이스를 만들때는 클라이언트에게 어떤 일을 할 수 있는지 정보를 제공해야 한다. 이러한 규칙에 위배되는것 중에 소위 constant interface라고 하는 것이있다. 보통 아래와 같이 정의되고, 상수에 접근할 필요가 있는 클래스에서 인터페이스를 구현해서 사용한다. 이렇게 인터페이스에 상수를 정의하는 이유는 클래스 이름을 사용해서 지칭해야 할 필요가 없기 때문이다.

``` java
public interface PhysicalConstants {
  static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
  static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

위와 같이 정의하면 다음과 같은 문제점이 있다.

1. 실제로는 클라이언트가 사용할 필요도 없지만, 이 상수 정보가 노출되어 혼란을 준다.
2. 미래에 클래스가 수정되어 위 상수가 필요없어진다고 하더라도 하위 클래스에서 위 상수 정보를 사용하는 경우, 인터페이스를 제거할 수 없다.

> 만약 상수가 존재하는 클래스 또는 인터페이스에 연관되어 있다면, 해당 클래스나 인터페이스에 정의해야 한다.

예를 들어, ```Integer``` , ```Double``` 과 같은 박싱된 숫자 클래스의 경우, 만약 상수가 enumerated type으로 가장 잘 표현된다면 enum type으로 표현해야한다.(ITEM 34) 그렇지 않으면, ```utility```  클래스로 노출해야 한다. 예를들어 ```Integer.MAX_VALUE``` 처럼 말이다.

위의 ```PhysicalConstants``` 인터페이스를 아래 처럼 수정할 수 있다.

``` java
public class PhysicalConstants {
    private PhysicalConstants(){}
    
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

> 위에서 숫자 사이에 _ 가 삽입 되어있는 것을 볼수 있는데, Java 7부터 숫자의 가독성을 위해서 _를 삽입할 수 있다.

만약 유틸리티 클래스에서 상수를 많이 사용하는 경우, ```static import``` 를 사용해서 클래스 이름을 생략할 수 있다.

``` java
import static item22.PhysicalConstants.AVOGADROS_NUMBER;

public class Test {
    double atoms(double mols) {
        return AVOGADROS_NUMBER * mols;
    }
    ...
}
```

## 요약

인터페이스는 오직 타입을 정의하기 위해서 사용되어야 하며, 단지 상수를 노출하기 위해서 사용되어서는 않된다.