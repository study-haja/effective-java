# Item23 : 태그 달린 클래스보다는  클래스 계층구조를 활용하라

태그 값으로 알려주는 클래스는 단점이 많다. 다음 코드를 보면 알 수 있을 것이다.

*  태그 달린 클래스

```java
class Figure {
  enum Shape { RECTANGLE, CIRCLE };
  
  // tag 기능
  final Shape shape;
  
  double length;
  double width;
  
  double radius;
  
  Figure(double radius) {
    shape = Shape.CIRCLE;
    this.radius = radius;
  }
  
  Figure(double length, double width) {
    shape = Shape.RECTANGLE;
    this.length = length;
    this.width = width;
  }
  
  double area() {
    switch(shape) {
      case RECTANGLE:
        return length * width;
      case CIRCLE:
        return Math.PI * (radius * radius);
      default:
        throw new AssertionError(shape);
    }
  }
}
```

다음 코드에서 단점이 무엇인지 보면 다음과 같다.

1. 열거 타입 선언, 태그 필드, switch 문 등 쓸데 없는 코드가 많다.
2. 여러 구현이 한 클래스에 혼합돼 있어서 가독성이 나쁘다.
3. 다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용한다.
4. 필드들을 final로 선언하려면 해당 의미에  쓰이지 않는 필드들 까지 생성자에 초기화해야 한다. ( 불변을 만들기 힘들다. )
5. 생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드들을 초기화하는데 컴파일러가 도와줄 수 있는 건 별로 없다. ( 엉뚱한 필드를 초기화해도 런타임에야 문제가 들어난다. )
6. 다른 타입이 추가 될 경우 코드를 수정해야 한다.
7. 마지막으로 인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다.

> 즉 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율 적이다. 즉 태그 달린 클래스는 계층 구조를 어설프게 흉내낸 아류일 뿐이다.

<br>

## 객체 지향 언어의 계층 구조

자바같은 객체 지향 언어는 계층구조를 활용해 위와 같은 문제를 해결 할 수 있다. 그럼 어떻게 하면 될까?

첫번째로 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다. 여기서 area() 메소드가 이에 해당한다.

두번째로 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에  일반 메서드로 추가한다. 모든 하위  클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다. Figure 클래스에서는 태그 값에 상관없는 메소드가 하나도 없고, 모든 하위 클래스에 사용하는 공통 데이터 필드도 없다.

세번째로 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다. Figure에서는 Circle,Rectangle 클래스를 만들면 된다. 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드들을 넣는다. 그리고 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다.

위의 해결방법을 예를 들면 다음과 같다.

* 태그 달린 클래스를 클래스 계층구조로 변환

```java
abstract class Figure {
  abstract double area();
}

class Circle extends Figure {
  final double radius;
  
  Circle(double radius) {
    this.radius = radius;
  }
  
  @Override
  double area() {
    return Math.PI * (radius * radius);
  }
}

class Rectangle extends Figure {
  final double length;
  final double width;
  
  Rectangle(double length, double width) {
    this.length = length;
    this.width = width;
  }
  
  @Override
  double area() {
    return length * width;
  }
}
```

해당 계층 구조를 통해 간결하고, 명확하며, 쓸데 없는 코드가 사라진 것을 볼 수 있다. 각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 제거했다. 또한 살아 남은 필드는 final이다. 즉 위에 설명했던 모든 단점을 사라지게 만들었다.

또한, 타입 사이의  자연스러운 계층 관계를  반영시킬 수 있다. 예를들어 Square라는 타입을 만들면 다음과 같다.

*  파생된 계층 구조 - Square

```java
class Square extends Rectangle {
	Square(double side) {
    super(side, side);
  }
}
```

이런식으로 다음과 같이 정사각형이 사각형의 특별한 형태임을 자연스럽고 아주 간단하게 만들 수 있다.