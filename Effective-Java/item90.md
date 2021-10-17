# Item 90 : Serialized 인스턴스 대신 Serialization Proxy를 사용하라.

## Serialization Proxy Pattern

enclosing class의 logical state를 정확히 나타내는 nested class를 정의하는 방법

``` java
public final class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(start + " after " + end);
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }

    public String toString() {
        return start + " - " + end;
    }

    // Serialization proxy for Period class - page 312
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        private static final long serialVersionUID = 234098243823485285L; // Any
      	
        private Object readResolve() {
            return new Period(start, end); // Uses public constructor
        }
    }

    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    private void readObject(ObjectInputStream stream)
            throws InvalidObjectException {
        throw new InvalidObjectException("Proxy required");
    }

}
```

## 장점

- 사용이 쉽다.
- 생성자를 통해서 객체를 리턴하게 되므로 앞에서 이야기한 공격에 안전함.
- 직렬화 할때의 클래스타입과 역직렬화할때의 클래스타입이 달라도  상관없음.

## 단점

- 순환 참조를 가지고 있는 객체, 사용자에 의해서 상속될수 있는 클래스에는 사용이 불가능
- 직열화, 역직렬화 속도가 약간 느림