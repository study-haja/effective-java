# Item 10 : EQUALS를 오버라이드 할때 일반적인 규칙을 따르라.

## 1. 일반적인 규칙

1. Reflexive : null이 아닌 레퍼런스 value x에 대해서, x.equals(x) == true 여야한다.
2. Symmetric : null이 아닌 레퍼런스 value x,y에 대해서, x.equals(y) == true 이면 y.equals(x) == true여야 한다.
3. Transitive : null이 아닌 레퍼런스 value x,y,z에 대해서, x.equals(y) == true, y.equals(z) == true이면 x.equals(z) == true
4. Consistent : null이 아닌 레퍼런스 value x,y에 대해서, x.equals(y)는 여러번 호출해도 같은 값을 리턴해야한다. (내부 값이 변경되지 않았다는 가정하에)
5. null이 아닌 value x에 대해서, x.equals(null) == false여야한다.

### Symmetric(대칭성)

다음 코드의 경우, 대칭성을 위반한다.

``` java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String) return s.equalsIgnoreCase((String)o);
        return false;
    }
  
    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";
        boolean flag1 = cis.equals(s); //true
        boolean flag2 = s.equals(cis); //false
    }
}
```

### Transitivity(이행성)

서로 상속관계인 클래스들 사이에 이행성을 만족하면서 비교함수를 구현하는것은 불가능하다. 

``` java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return p.x == x & p.y == y;
    }
}

public class ColorPoint extends Point {
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y); this.color = color;
    }
    
    @Override 
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        if (!(o instanceof ColorPoint))
            return o.equals(this);
        return super.equals(o) && ((ColorPoint)o).color == color;
    }

    public static void main(String[] args) {
        ColorPoint p1 = new ColorPoint(1,2, Color.RED);
        Point p2 = new Point(1,2);
        ColorPoint p3 = new ColorPoint(1,2,Color.BLUE);
        boolean flag1 = p1.equals(p2); // true
        boolean flag2 = p2.equals(p3); // true
        boolean flag3 = p1.equals(p3); // false!
    }
}
```

이 경우는 상속관계의 클래스들을 분리하여, 각각의 필드를 구성(Composition)하여 해결할수 있다.

```java
public class ColorPoint {
    private final Point point;
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        point = new Point(x,y);
        this.color = Objects.requireNonNull(color);
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }

    public static void main(String[] args) {
        ColorPoint p1 = new ColorPoint(1,2, Color.RED);
        Point p2 = new Point(1,2);
        ColorPoint p3 = new ColorPoint(1,2,Color.BLUE);
        boolean flag1 = p1.equals(p2); // false
        boolean flag2 = p2.equals(p3); // false
        boolean flag3 = p1.equals(p3); // false
    }
}
```

### Consistency(일관성)

equals method에서 신뢰할수 없는 리소스들에 의존하면 않된다. 예를들어, ```java.net.URL``` 의 equals 함수는 host의 ip주소를 비교하는 작업이 있는데, 비교작업이 인터넷 연결을 필요로 하므로 결과가 매번 같다는 보장이 없다. 이런방식은 좋지 못한 방식이다. 

## 2. 고퀄리티 equals 함수작성하기

1. 자기자신 객체와 비교하는지 체크할때, == 연산자 사용하기
2. argument가 정확한 타입인지 검사할때, instance of 연산자 사용하기
3. argument를 정확한 타입으로 캐스팅하기
4. 클래스 내에있는 모든 변수들을 서로 비교하기

``` java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 9, "are code");
        this.prefix = rangeCheck(prefix, 9, "prefix");
        this.lineNum = rangeCheck(lineNum, 9 9, "line um");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max) throw new IllegalArgumentException(arg + ":" + val);
        return (short) val;
    }

    @Override 
  	public boolean equals(Object o) { 
        if (o == this) return true; // 1.
        if (!(o instanceof PhoneNumber)) return false;  // 2.
        PhoneNumber pn = (PhoneNumber)o;  // 3.
        return pn.lineNum == lineNum & pn.prefix == prefix & pn.areaCode == areaCode; // 4.
    }
}
```

5. equals를 오버라이드 할때, hashcode 함수도 오버라이드 하기.
6. 전혀 다른 타입들과 무리해서 비교하려고 하지말기 (위에 대칭성 위반 경우 참고)
7. equals 함수의 파라미터 타입을 Object 타입 이외의 것으로 바꾸지 말기 (함수 오버라이드가 안됨)

## 3. equals, hashcode 함수를 작성을 자동화하는 방법

1. Google의 오픈소스인 ```AutoValue``` Framework 사용  : 단일 annotation으로 함수 자동생성
2. IDE에서 제공하는 자동함수 생성기능 사용 : ```AutoValue``` 보다 verbose해지고 코드 가독성이 낮아지지만, 직접 작성하는 것보다 낫다.

## 4. 결론

> 꼭 필요한 경우 아니면 equals 함수를 작성하지 마라, Object 함수가 구현하는 equals함수로 보통 충분하다. 만약 override 하게된다면, 앞에서 언급한 equals 계약조건을 만족하도록 작성하라.