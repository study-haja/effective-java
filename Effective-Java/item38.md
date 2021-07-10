# ITEM 38 : 인터페이스를 사용해서 확장성있는 ENUM을 재현하라.

## 개요

Enum을 확장하고 싶을때, enum을 상속하면 된다고 생각할 수 있는데 문제가 있다. 왜냐하면 상속된 enum type은 부모 enum type 이라고 볼수 있지만, 부모 enum type은 자식 enum type이라고 볼수 없기 때문에, 서로 확장 관계로 보기에는 해깔리기 때문이다.

## 인터페이스를 사용해서 확장성있는 Enum 구현하기

``` java
public interface Operation {
    double apply(double x, double y);
}
```

``` JAVA
// BasicOperation.java
public enum BasicOperation implements Operation {

    PLUS("+"){
        @Override
        public double apply(double x, double y) {
            return x+y;
        }
    },
    MINUS("-"){
        @Override
        public double apply(double x, double y) {
            return x-y;
        }
    },
    TIMES("*"){
        @Override
        public double apply(double x, double y) {
            return x*y;
        }
    },
    DIVIDE("/"){
        @Override
        public double apply(double x, double y) {
            return x/y;
        }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}

```

``` java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

위 방식으로 하면 BasicOperation 타입을 사용하는 모든 곳에서 ExtendedOperation 타입으로 치환해도 무방하다. 

``` java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test( Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants()) System.out.printf("%f %s %f = %f%n",
            x, op, y, op.apply(x, y));
}
```

## 사소한 단점

하나의 enum이 다른 enum타입을 상속할 수 없다는 단점이 있다. 이를 보완하기 위해서 두개의 enum에 있는 공통 로직을 인터페이스에 default 함수로 구현할수 있다.

만약, default 함수에서 정의하기에는 코드량이 많다면 helper 클래스나 static helper 함수를 사용해서 코드 중복을 제거할 수 있다.

## 요약

확장성있는 enum type을 구현하고 싶다면, 인터페이스를 사용해라. 그리고 이 인터페이스를 상속하는 basic enum 타입을 정의해라. 이렇게 하면 클라이언트는 인터페이스를 상속하여 자신만의 enum을 만들수 있다. 이렇게 하면 basic enum type이 사용되는 모든 곳에서 해당 enum이 사용가능해진다.

