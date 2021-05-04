# Item5 : 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라



클래스를 사용할 때 대부분의 클래스는 하나 이상의 자원을 사용할 것이다. 그중 Utility 클래스나 싱글톤 클래스를 만들어 자원을 효율적으로 사용할 수 있다. 하지만 이는 다음과 같은 문제가 발생할 수 있다. 다음 예를 들어 보자.

* Utility 클래스의 검시기 사전

  ```java
  public class SpellChecker {
  	private static final Dictionary dictionary = ...;
  	
  	private SpellChecker() {}
  	
  	public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
  }
  ```

* Singleton 클래스의 검시기 사전

  ```java
  publicclass SpellChecker {
  	private final Dictionary dictionary = ...;
  	
  	private SpellChecker(...) { }
  	public static SpellChecker INSTANCE = new SpellChecker(...);
  	
  	public boolean isValid(String word) { ... }
  	public List<String> suggestions(String typo) { ... }
  } 
  ```

이 둘의 예제는 기존 item3, item4를 사용해 검시기 사전 클래스를 만들었는데 다음과 같은 단점이 보여진다. 바로

``하나의 사전만 사용한다 -> 여러 사전을 사용하기 어렵다 -> 클래스가 재활용되기 어렵다 -> 중복 코드가 발생한다``

바로 위와 같은 문제로 인해 유틸리티나 싱글톤 방식으로 클래스를 구현하는 것은 그다지 효율적이지 않아 보이게 된다. 그렇다고 final 키워드를 없애고 setter를 두어 매번 사전을 교체하기에는 흐름이 어색하고 오류를 내기 쉬우며 멀티 스레드 환경에서는 쓰기가 매우 힘들다. 

즉 ``사용하는자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.``

따라서 이러한 문제를 해결하기 위해 이펙티브 자바에서는 다음과 같이 말한다.

> 클라이언트가 원하는 자원을 사용해야 하며 클래스가 여러 자원 인스턴스를 지원하기 위해서는 다음과 같은 방식을 사용하는데 
>
> 바로 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이다.

한번 예를 들어보자

* 의존 객체 주입으로 만든 검사기 사전

  ```java
  public class SpellChecker {
  	private final Dictionary dictionary;
  	
  	public SpellChecker(Dictionary dictionary) {
  		this.dictionary = Objects.requireNonNull(dictionary);
  	}
  	
  	public boolean isValid(String word) { ... }
  	public List<String> suggestions(String typo) { ... }
  }
  ```

위의 예제를보면 생성자에다 매개변수를 두어 자원을 넣는 구조이다. 이 구조가 좋은 이유는 몇개의 자원이 오든 매개변수를 통해 자원을 사용할 수 있다는 것이다. 즉 자원이 몇개든 의존 관계가 어떻든 상관없이 잘 작동한다.

또한 final 키워드로 인한 불변을 보장하기 때문에 여러 클라이언트가 의존 객체들을 안심하고 공유해 사용할 수 있다. 

위 생성자 패턴에 변형으로 생성자에 자원 팩토리를 넘겨주는 방식이 있다. 여기서  팩토리란 호출할 때마다 특정 타입의 인스턴스를 반복해서  만들어주는 객체를 말한다. 즉 팩토리 메소드 패턴을 구현한 것이며 ``Supplier<T>`` 인터페이스가 대표적인 팩토리를 표현한 예이다.

즉 Supplier를 사용해 클라이언트는 자신이  명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.

* 모자이크 생성 메소드 - Supplier를 매개변수를 받아 하위 타입까지 제공

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```



따라서 정리하면 다음과 같다. 

1.  클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 

2.  이 자원들을 클래스가 직접 만들게 해서도 안된다.

따라서 필요한 자원을 생성자에게 넘겨주면 클래스의 유연성, 재사용성, 테스트 용이성을 효율적으로 개선해 줄것이다.