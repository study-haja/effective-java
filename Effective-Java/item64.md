# Item 64 : 인터페이스로 객체들을 참조하라.

만약 적절한 인터페이스 타입이 존재할때, 파라미터와 리턴 타입 그리고 필드들은 모두 인터페이스 타입으로 선언되어야 한다. 이유는 더 프로그램이 더 유연해지기 때문이다.

## 인터페이스 사용의 장점

### 1. 여러 구현체 사용가능

``` java
LinkedHashSet<Son> sonSet = new LinkedHashSet<>(); // 유연하지 않음.
Set<Son> sonSet = new HashSet<>(); // 유연함.
```

위에 코드 처럼 인터페이스 타입은 여러 구현을 사용할수 있다.

### 2. 구현 클래스의 변동사항에 영향을 받지 않음.

구현 클래스 타입으로 사용시, 구현 클래스가 후에 업데이트 되면, 클라이언트 코드는 더이상 동작하지 않을수 있다.

하지만 인터페이스는 변경될 가능성이 낮다.

## 적절한 인터페이스가 존재하지 않을 경우 어떻게 할까?

1. ```String```, ```BigInteger``` 와 같은 Value Class들은 보통 인터페이스로 구현이 되지 않는다.

2. ```OutputStream``` 과 같이 클래스 기반 프레임워크들 또한 적절한 인터페이스가 존재하지 않는다. 보통 인터페이스 대신 추상 클래스가 사용된다.

3. 인터페이스를 구현하지만, 인터페이스에 없는 추가 메소드를 구현하는 경우가 있다. 예를들면, ```PriorityQueue``` 클래스는 ```Queue```  인터페이스가 없는 ```comparator``` 함수를 가지고 있다.

위 세가지 경우에 클래스 계층구조에서 가장 덜 specific class를 구현하면 된다.