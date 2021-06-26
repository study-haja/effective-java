# Item 34 : INT 상수 대신 ENUM을 사용하라.

## int enum 패턴

``` java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int ORANGE_NAVEL = 0;
// ...
```

위와 같이 정의할 경우, ```APPLE_FUJI == ORANGE_NAVEL``` 과 같은 연산에 컴파일 에러가 발생하지 않아서 타입 안전하지 않다. 그리고 함수에 특정 enum 타입을 강요할 수 없다.

두번째 문제점은 int enum 상수를 출력 문자열로 변환하는 것이 쉽지 않다는 것이다. 왜냐하면 값을 출력하면 숫자만 나오기 때문이다. 

## enum 타입

``` java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange {NAVEL, TEMPLE, BLOOD }
```

자바에서 enum은 다른 언어의 enum보다 덜 강력한데, 그 이유는 내부적으로 int 값을 가지는 것이 아니라 public static final field를 통한 하나의 인스턴스를 노출하는 클래스이기 때문이다. 

enum 타입의 첫번째 장점은 만약 Apple이라는 타입을 파라미터로 선언한다면, 해당 파라미터에 넘겨진 객체는 Apple 값중에 하나임을 보장할 수 있다.

두번째 장점은 enum을 출력가능한 문자열로 변환하는 것이 쉽다. 왜냐하면 toString함수를 호출하면 되기 때문이다.

세번째 장점은 임의의 함수와 필드들을 추가할수 있고 임의의 인터페이스를 구현할 수도 있다. 

## rich enum 타입

아래와 같이 ```surfaceGravity``` 필드를 추가해서, enum 생성자에서 값을 설정할 수도있다.

```java
public enum Planet {
    MERCURY(3.302e+23,2.439e6),
    VENUS(4.869e+24,6.052e6),
    EARTH(5.975e+24,6.378e6),
    MARS(6.419e+23,3.393e6),
    JUPITER(1.899e+27,7.149e7);
    // ...
    
    private final double mass;
    private final double radius;
    
    private final double surfaceGravity;
    
    private static final double G = 6.67300E-11;
    
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    
    public double mass() { return mass;}
    public double radius() { return radius;}
    public double surfaceGravity() { return surfaceGravity;}
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

enum type은 아래와 같이 ```values``` 함수를 호출해서 모든 enum 값들을 받아올수 있기 때문에 강력하다. (int enum 타입으로는 이것이 불가능하다.)

``` java
public class WeightTable {
    public static void main(String[] args) {
        double earthWeight = Double.parseDouble(args[0]);
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        for (Planet p : Planet.values()) 
            System.out.printf("Weight on %s is %f%n",p,p.surfaceWeight(mass));
    }
}
```

## enum 타입별로 다른 행동을 정의해야하는 경우

다음과 같은 enum을 정의해야 한다고 하자.

``` java
public enum Operation {
    PLUS,MINUS,TIMES,DIVIDE;

    public double apply(double x, double y) {
        switch(this) {
            case PLUS: return x+y;
            case MINUS: return x-y;
            case TIMES: return x*y;
            case DIVIDE: return x/y;
        }
        throw new AssertionError("UNknown op:" + this);
    }
}
```

위 코드는 새로운 enum 타입을 추가할 경우 switch문에 새로운 enum 타입에 대한 처리를 추가해줘야 한다. 만약 잊어버릴 경우, 여전히 컴파일은 되지만 런타임에 apply 메소드를 호출할때 에러가 날것이다.

반면 아래와 같이 선언할경우, apply 메소드를 추가하는 것을 잊어버리더라도 컴파일러가 이를 알려준다.

```java
public enum Operation {
    PLUS {public double apply(double x, double y) {return x+y;}},
    MINUS {public double apply(double x, double y) {return x-y;}},
    TIMES {public double apply(double x, double y) {return x*y;}},
    DIVIDE {public double apply(double x, double y) {return x/y;}};
    
    public abstract double apply(double x, double y);
}
```

## Constant specific 함수의 문제점과 개선

다음 enum은 주말과 평일인 경우에 overtime pay를 다르게 계산하여 급여를 구하는 함수를 보여준다.

```java
public enum PayrollDay {
    MONDAY,TUESDAY,WEDNESDAY,THURSDAY,FRIDAY,SATURDAY,SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8*60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        
        int overtimePay;
        switch (this) {
            case SATURDAY: case SUNDAY: // Weekend
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? o : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
```

위 코드의 경우 만약  vacation 이라는 enum을 추가하고 해당 처리를  pay 함수 switch문에 정의해두지 않으면, 컴파일은 되지만 런타임에 weekday 때와 같이 계산이 되고만다.

아래와 같은 경우, 새로운 enum 타입을 추가했을때, 그에 해당하는 PayType을 선택하도록 강요할 수 있으므로 위의 부작용을 해결할 수 있다. 아래와 같은 형태를 strategy enum pattern이라고 한다.

``` java
public enum PayrollDay {
    MONDAY,TUESDAY,WEDNESDAY,THURSDAY,FRIDAY,SATURDAY(PayType.WEEKEND),SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;
    
    PayrollDay(PayType payType) {this.payType = payType;}
    PayrollDay() {this(PayType.WEEKDAY);}
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
  // strategy enum type
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        }
        
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked,payRate);
        }
    }
}
```

## 요약

enums은 일반적으로 int constant와 성능이 비슷하다. 사소한 단점은 enum type을 초기화하고 저장할때 발생하는 시간, 공간 비용이 있다는 것이다. 하지만 실무에서 이러한 단점은 미미하다. enum을 사용해야하는 경우는 일련의 상수들이 컴파일 타임에 알려진 경우에 사용하면 된다. 예를들어, planet, days of the week, 체스 조각 등이다. 

enum의 장점은 분명하다. enums은 가독성이 좋고, 더 안전하고 강력하다. enum에서 명시적인 생성자를 정의하여 데이터를 넘기고 해당 데이터에 영향을 받아 함수를 생성할 수 있는 장점도 있다. 상대적으로 드물지만, 단일 메소드에 각각의 행동을 정의하는 방법도 있다. 하지만 enum 상수가 공통된 행동을 공유한다면 strategy enum pattern을 사용하라.
