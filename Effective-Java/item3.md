# Item3 : 생성자나 열거 타입으로 싱글턴임을 보증하라



싱글톤 패턴이란 인스턴스를 오직  하나만 생성할 수 있는 클래스를 말한다. 즉 함수와 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트를  들 수 있다. 싱글톤 패턴으로 클래스 오브젝트가 만들어지면 어플리케이션 내에서 전역으로 해당 오브젝트를 사용할 수 있다.

하지만 싱글턴 패턴은 다음과 같은 단점이 있다. 바로

``클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워 질수 있다는 것이다. `` 싱글톤이 인터페이스로 정의해서 만든 것이 아니라면  Mock 으로 대체해 테스트 하기가 어렵기 때문이다.

물론 테스트가 아예 안되는 것은 아니다.  PowerMock같은 도구를 통해 정적 메소드를 테스트 하게 할 수 있고 Mockito 3.4.0 버전에서는 정적 메소드를 지원해 준다고 한다.

싱글톤 패턴은 보통 다음과 같이 2가지로 만든다.

* public static final 필드 방식의 싱글턴

  ```java
  public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
  }
  ```

  private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화 할 때 딱 한번만 호출한다. public 이나 protected 생성자가 없으므로 Elvis 클래스가 초기화 될때 만들어진 인스턴스가 전체 시스템에서 하나 뿐임을 보장하게 된다.

  하지만 클라이언트에서 AccessibleObject.setAccessible을 사용해 private를 호출할 수 있다.

  즉 리플렉션을 통 새로운 인스턴스가 생성 될 수 있기 때문에 count같은 값을 두고 예외를 던져 해결할 수 있다.

* 정적 팩토리 방식의 싱글턴

  ```java
  public class Elvis {
  	private static final Elvis INSTANCE = new Elvis();
  	private Elvis() { ... }
  	
  	public static Elvis getInstance() { return INSTANCE; }
  	
  	public void leaveTheBuilding() { ... }
  }
  ```

  Elvis.getInstance는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis 인스턴스를 만들지 않는다. 또한 위와 마찬가지로 리플레게션을 통한 예외도 똑같이 해결할 수 있다.

이하 싱글톤 패턴을 만드는 방식 2개를 설명해 봤는데 각각의 장점을 살펴보고자 한다.

|      | public static final 필드 방식의 싱글턴          | 정적 팩토리 방식의 싱글턴                                    |
| ---- | ----------------------------------------------- | ------------------------------------------------------------ |
| 장점 | 해당 클래스가 싱글턴임이 API에 명백히 드러난다. | API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다. <br />( 유일한 인스턴스를 반환하던 팩터리 메소드가 호출하는 <br />스레드별로 다른 인스턴스를 넘겨주게  할 수 있다. ) |
|      | 코드가 간결하다                                 | 정적 팩터리를 제네릭 싱글턴 패턴으로 만들 수 있다.           |
|      |                                                 | 정적 팩터리의 메소드 참조를 공급자(supplier)로 사용할 수 있다.<br />( `Elvis::getInstance`를 `Supplier<Elvis>`로 만들수 있다.) |

 하지만 직렬화를 할 시 둘 중 어떠한 방법으로도 직렬화에 대한 역직렬화가 같다는 보장을 못받는다. 즉 새로운 인스턴스가 생겨 싱글톤임을 보장받지 못한다는 것이다. 따라서 이를 해결하기 위해서는 readResolve 메소드를 제공해야 한다. 이렇게  하지 않으면 인스턴스를 역직렬화할때 마다 새로운 인스턴스가 만들어 질것이다.

* readResolve() 메소드를 통해 싱글톤임을 보장 

  ```java
  public class Elvis implements Serializable {
  	private static final Elvis INSTANCE = new Elvis();
  	private Elvis() { ... }
  	
  	public static Elvis getInstance() { return INSTANCE; }
    
    private Object readResolve() {
  		return INSTANCE; // 진짜 Elvis 반환 
  	}
  	
  	public void leaveTheBuilding() { ... }
  }
  ```

  

위에 코드는 직렬화를 해결할 수 있지만 코드가 길어지고 객체가 하는 일이 많다고 보여질 수 있다. 따라서 이 문제를 해결하기 위해 싱글턴을 열거형 타입으로 만들수 있다.

* 열거 타입 방식의 싱글턴

  ```java
  public enum Elvis {
  	INSTANCE;
  	
  	public void leaveTheBuilding() { ... }
  }
  ```

이 방식은 public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화 할 수 있다. 그리고 아주 복잡한 직렬화 상황이나 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.

즉 대부분 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다. 단 만들려는 싱글턴이 Enum외에 클래스를 상속해야 한다면 이 방법을 사용할 수 없다.



