# Item89 : 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

* 싱글턴 패턴 예제

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	
	private Elvis() { ...  }
	
	public void leaveTheBuilding() { ... }
}
```

  위와 같이 아이템 3에서 싱글턴 예제를 보았다. 이 클래스는 바깥에서 생성자를 호출하지 못하게 막는 방식으로  인스턴스가 오직 하나만 만들어짐을 보장했다. 

 하지만 이 클래스는 implement Serializable을 선언하는 순간 더 이상 싱글턴이 아니게 된다. 기본 직렬화를 쓰지 않더라도, 그리고 명시적인  readObject를 제공하더라도 소용없다. 어떤 readObject를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다.

readResolve 기능을 이용하면 readObject가 만들어낸 인스턴스를 다른것으로 대체할 수 있다. 이때 readObject 가 만들어낸 인스턴스는 가비지 컬렉션의 대상이 된다. 

 위의 Elvis 클래스가 Serializable을 구현한다면 다음의 readResolve 메소드를 추가해 싱글턴이라는 속성을 유지할 수 있다.

* readResolve 예제

```java
private Object readResolve() {
    // 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
  	// 기존에 생성된 인스턴스를 반환한다.
    return INSTANCE;
}
```

이 메소드는 역직렬화한 객체는 무시하고 클래스 초기화 때 만들어진 Elvis 인스턴스를 반환한다. 따라서 Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없으니 모든 인스턴스 필드를 transient로 선언해야 한다. 

사실, readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다. 그렇지 않으면 아이템 88에서 살펴본 MutablePeriod 공격과 비슷한 방식으로 readResolve 메소드가 수행되기전에 역직렬화된 객체의 참조를 공격할 여지가 남는다. 

즉, 역직렬화 과정에서 역직렬화 인스턴스를 가지고 올 수 있으며 이는 싱글턴이 깨지게 된다는 점이다.



### 해결 방법

위의 문제는 enum을 사용해서 해결하면 된다. 그 이유는 자바가 선언한 상수 외에 다른 객체가 없음을 보장해주기 때문이다. 물론 AccessibleObject.setAccessible 메서드와 같은 리플렉션을 사용했을 때는 예외다. 임의의 네이티브 코드를 수행할 수 있는 특권을 가로챈 공격자에게는      모든 방어가 무력화된다. 다음 예를 보자

* 열거 타입 싱글턴 - 전통적인 싱글턴보다 우수하다.

```java
public enum Elvis {
	INSTANCE;
  
  private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};
  
  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs));
  }
}
```

인스턴스 통제를 위해 readResolve를 사용하는 방식이 완전히 쓸모없는 것은 아니다. 직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데, 컴파일타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능 하기 때문이다. 이 때는 열거 타입으로 표현하는 것이 불가능하기 때문에 `readResolve` 메서드를 사용할 수 밖에 없다.

readResolve 메소드의 접근성은 매우 중요하다. final 클래스에서라면 readResolve 메소드는 private이어야 한다. final이 아닌 클래스에서는  다음의  몇가지를 주의해서 고려해야 한다.

1. private으로 선언하면 하위 클래스에서 사용할 수 없다.
   1. package-private으로 선언하면 같은 패키지에  속한 하위 클래스에서만 사용할 수 있다.
2. protected나 public으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다.
   1. protected나 public이면서 하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의인스턴스를 생성해서 ClassCastException을 일으킬 수 있다.