# Item13 : clone 재정의는 주의해서 진행하라

Clonable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다. 하지만 의도한 목적을 제대로 이루지 못했고 가장 큰 문제는 clone 메소드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저 protected라는데에 있다. 그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메소드를 호출할 수 없다.

* Object 클래스와 Cloneable 인터페이스에서 clone 메소드와의 관계

  ```java
  public class Object {
    ...
    
    @HotSpotIntrinsicCandidate // clone 메소드가 Object에 선언, protected로 선언
  	protected native Object clone() throws CloneNotSupportedException; 
  }
  
  public interface Cloneable {} // 정작 Cloneable 인터페이스에는 clone 메소드가 선언되어 있지 않음!!
  ```

 물론 리플렉션을 사용하면 가능하겠지만 100%가능한것은 아니다. 그 이유는 해당 객체가 접근이 허용된 clone 메소드를 제공한다는 보장이 없기 때문이다. 하지만 Cloneable 방식은 넓리 쓰이고 있기 때문에 clone 메소드를 잘 동작하는 구현 방법과 언제 써야하는지, 가능한 다른 선택제에 관해 설명하려고 한다.

<br>

## Cloneable 인터페이스

 Cloneable 인터페이스는 Object의 protected 메소드인 clone의 동작 방식을 결정한다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면  그 객체의 필드들을 하나하나  복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedException`을 던진다. 

실무에서 Cloneable을 구현한 클래스는 clone 메소드를 public 으로 제공하며 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. 이 기대를 만족하기 위해서 해당 클래스와 상위 클래스는 복잡하고 , 강제할 수 없고, 허술하게 기술된 프로토콜을 지켜야만 하는데, 그 결과로 깨지기 쉽고, 위험하고, 모순된 메커니즘이 탄생하게 된다. 즉 생성자를 호출하지 않고 객체를 생성할 수 있게 되는 것이다.

<br>

## Clone 메소드의 일반 규약

Object에서 Clone의 일반 규약은 다음과 같다. 

> 1. 복사의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있지만 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
>    1. x.clone() != x
>    2. x.clone().getClass() == x.getClass()
> 2. 하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다. 다음을 보면
>    1. x.clone().equals(x) 은 참이지만 필수는 아니다.
> 3. 관례상, 이 메소드가 반환하는 객체는 super.clone을 호출해  얻어야 한다. 이 클래스와 Object를 제외한 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
>    1. x.clone().getClass() == x.getClass()
> 4. 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수 도 있다.

강제성이 없다는 점만 빼면 생성자 연쇄와 살짝 비슷한 메커니즘이다. 즉 clone 메소드가 super.clone이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 오류가 나지 않을 것이다. 하지만 이 클래스의 하위 클래스에서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 Clone 메소드가 제대로 동작하지 않게 된다.

* 상속 관계에서 잘못된  clone 메소드 예제

  ```java
  abstract class Parent implements Cloneable {
  
      @Override
      protected Object clone() throws CloneNotSupportedException {
          return new parent() {
              @Override
              protected Object clone() throws CloneNotSupportedException {
                  return super.clone();
              }
          };
      }
  }
  
  class Child extends Parent {
  
      public static void main(String[] args) throws CloneNotSupportedException {
          Child child = new Child();
          System.out.println(child.clone().getClass() == child.getClass()); // false -> 부모 클래스를 clone 했기 때문에
          System.out.println(child.clone() != child); // true
      }
  
      @Override
      protected Object clone() throws CloneNotSupportedException {
          return super.clone();
      }
  }
  ```

<br>

## 가변 상태를 갖는 클래스의 Clone 방법

 clone을 재정의한 클래스가 final이라면 걱정해야 할 하위 클래스가 없으니 이 관례는 무시해도 된다. 하지만 final 클래스의 clone 메소드가 super.clone을 호출하지 않는다면 Cloneable을 구현할 이유도 없다. Object의 clone 구현의 동작 방식에 기댈 필요가 없기 때문이다.

 제대로 동작하는 clone 메소드를 가진 상위 클래스를 상속해 Cloneable을 구현하고 싶다면 먼저 super.clone을 호출한다. 그렇게 얻은 객체는 원본의 완벽한 복제본일 것이고 클래스에 정의되니 모든 필드는 원본 필드와 똑같은 값을 갖는다. 또한 모든 필드가 기본 타입이거나 불변 객체를 참조한다면 더욱더 완벽한 상태가 될 것이다. 다음 예제가 여기에 해당한다.

* 가변 상태를 참조하지 않는 클래스용 clone메소드

  ```java
  @Override 
  public PhoneNumber clone() {
  	try {
  		return (PhoneNumber) super.clone();
  	} catch (CloneNotSupportedException e) { // PhoneNumber에 Cloneable 인터페이스 구현
  		throw new AssertionError(); // 일어날 수 없는 일
  	}
  }
  ```

PhoneNumber에는 바뀌는 값이 없어 복제가 완벽해 진다. 여기서 중요한 점은 super.clone()은 Object로 반환하기 때문에 clone()의 반환타입을 Object로 하지 않고 PhoneNumber를 주어 사용자가 굳이 밖에서 형변환을 하지 않게 한다. 그 이유는 저 위에 코드가 절대 실패 할일이 없기 때문에 외부에서 신경쓰지 않도록 하는것이 좋다. 

<br>

 만약 가변 객체를 참조하는 클래스를 clone 한다면 어떻게 될까? 다음 예제를 보자

* 가변객체를 사용하는 stack 클래스

  ```java
  public class Stack {
  	private Object[] elements;
  	private int size = 0;
  	private static final int DEFAULT_INITIAL_CAPACITY = 16;
  	
  	public Stack() {
  		this.element = new Object[DEFAULT_INITIAL_CAPACITY];
  	}
  	
  	public void push(Object e) {
  		ensureCapacity();
  		elements[size++] = e;
  	}
  	
  	public Object pop() {
  		if (size == 0) throw new EmptyStackException();
  		Object result = elements[--size];
  		elements[size] null;
  		return result;
  	}
  	
  	private void ensureCapacity() {
  		if(elements.length == size)
  			elements = Arrays.copyOf(elements, 2 * size + 1);
  	}
  }
  ```

해당 클래스를 복제할때 단순히 super.clone을 반환하면 어떻게 될까? 반환된 stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, elements 필드는 원본 stack 인스턴스와 **똑같은 배열**을 참조할 것이다. 즉 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변식을 해친다는 이야기다. 이는 프로그램이 이상하게 동작하고 NPE를 발생시킬 것이다.

이를 해결하기 위해선 다음과 같이 하면 된다.

* 가변 상태를 참조하는 클래스용 clone 메소드

  ```java
  @Override
  public Stack clone() {
  	try {
  		Stack result = (Stack) super.clone();
  		result.elements = elements.clone(); // element들을 전부 clone 해버린다.
  		return result; 											// 배열의 clone은 순서조차 똑같이 반환 -> 배열을 복제할때는 clone()을 사용 권장
  	} catch(CloneNotSupportedExcetion e) {
  		throw new AssertionError();
  	}
  }
  ```

위와 같이 내부 정보 - element들을 새로 만들면 된다. 즉 clone 메소드는 사실상 생성자와 같은 효과를 내기 때문에, 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

한편 여기서 element의 필드가 final로 선언해버렸다면 위와 같은 코드는 동작하지 않는다. final 필드에는 새로운 값을 할당할 수 없기 때문에 `cloneable 아키텍처는 가변 객체를 참조하는 필드는 final 로 선언하라는 일반 용법과 충돌한다`. 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수 있다.

<br>

## 복잡한 가변 상태를 갖는 클래스의 Clone 방법

clone을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있다. 이번에는 해쉬 테이블용 clone 메소드를 생각해보자. 해쉬 테이블 내부는 버킷들의 배열이고, 각 버킷은 key-value라는 쌍을 담는 연결 리스트의 첫 번째 엔트리를 참조한다. 그리고 성능을 위해 linkedlist 대신 경량 연결리스트를 사용한 예를 보자.

* 경량 연결리스트를 사용한 Hash Table

  ```java
  public class HashTable implements Cloneable {
  	private Entry[] buckets = ...;
  	
  	private static class Entry {
  		final Object key;
  		Object value;
  		Entry next;
  		
  		Entry(Object key, Object value, Entry next) { // 복사할 경우 Entry가 또다른 Entry를 
  			this.key = key;															// 참조하고 있기 때문에 이를 해결해야 한다.
  			this.value = value;
  			this.next = next; // 참조가 되어있어 복제본이나 원본이 값이 바뀌면 둘다 영향을 받게 된다.
  		}
  	}
  	
  	...
  }
  ```

* Hash Table의 잘못된 clone 메소드 - 가변 상태 공유

  ```java
  @Override
  public HashTable clone() {
  	try {
  		HashTable result = (HashTable) supepr.clone();
  		result.buckets = buckets.clone();
  		return result;
  	} catch (CloneNotSupportedException e) {
  		throw new AssertionError();
  	}
  }
  ```

여기 잘못된 clone 메소드를 보면 복제본은 자신만의 버킷 배열을 갖지만, 이  배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다. 이를 해결하려면 각 버킷을 구성하는 연결 리스트를 복사해야 한다. 따라서 다음과 같이 해결하면 된다.

* 복잡한 가변 상태를 갖는 클래스용 재귀적 clone 메소드

  ```java
  public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
  
    private static class Entry {
      final Object key;
      Object value;
      Entry next;
      
      Entry(Object key, Object value, Entry next) {
        this.key = key;
        this.value = value;
        this.next = next;
      }
      
      Entry deepCopy() { // 깊은 복사를 통해 Entry를 재귀적으로 복사
        return new Entry(key, value, next == null? null : next.deepCopy()); // Entry가 널일 때까지 재귀적 호출
      }
    }
    
    @Override
    public HashTable clone() {
      try {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];
        for (int i = 0; i < buckets.length; i++) {
          if (buckets[i] != null)
            result.buckets[i] = buckets[i].deepCopy(); // 복사된 연결리스트들을 복사한 bucket에 넣음
        } 
        return result;
      } catch (CloneNotSupportedException e) {
      		throw new AssertionError();
      }
    }
  }
  ```

private 클래스인 HashTable.Entry는 깊은복사를 지원하고 clone 메소드에서 적절한 크기의 새로운 버킷 배열을 할당해 넣어주었다. 이 때 Entry의 deeppCopy 메소드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다. 이 기법을 통해 버킷이 너무 길지 않다면 잘 작동한다. 

 하지만 연결 리스트의 복제하는 방법으로는 그다지 좋지 않다. 재귀 호출 때문에 리스트가 길면, 오히려 StackOverFlow가 발생할 수 있다. 이 문제를 해결하기 위해서는 deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정해야 한다.

* 엔트리 자신이 가리키는 연결 리스트를 반복적으로 복사하는 방법

  ```java
  Entry deepCopy() {
  	Entry result = new Entry(key, value, next);
  	for (Entry p = result; p.next != null; p = p.next) { // Entry를 순회하며 next가 null일때 까지 new Entry() 생성
  		p.next = new Entry(p.next.key, p.next.value, p.next.next); 
  	}
  	return result;
  }
  ```

<br>

## 고수준의 Clone 방법

 이제 복잡한 가변 객체를 복제하는 마지막 방법 살펴보려고 한다. 먼저 super.clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메소드들을 호출한다.

만약 HashTable이라면 buckets 필드를 새로운 버킷 배열로 초기화한 다음 원본 테이블에 담긴 모든 key-value 쌍 각각에 대해 복제본 테이블의 **put(key, value)** 메소드를 호출해 둘의 내용이 똑같게 해주면 된다.

이처럼 고수준 API를 사용하면 간단하게 해결할 수 있지만, 저수준에서 보다는 처리 속도가 느리다. 또한 Cloneable 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회하기 때문에 Cloneable 아키텍처와는 어울리지 않는 방식이기도 하다.

<br>

## Clone시 주의 사항

생성자에서는 재정의 될 수 있는 메소드를 호출하지 않아야 하는데 clone 메소드도 마찬가지다. 만약 clone이 하위 클래스에서 재정의한 메소드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 크다. 따라서 예를들어 HashTable의 **put(key, value)** 메소드 같은 경우 final이거나 private이어야 한다.

Object의 clone메소드는 CloneNotSupportedException을 던지고 선언했지만 재정의한 메소드는 그렇지 않다. ***public인 clone 메소드에서는 throws 절을 없애야 한다.*** 검사 예외를 던지지 않아야 그 메소드를 사용하기 편하기 때문이다.

* throws를 안 없앨 경우

  ```java
  @Override
  protected Object clone() throws CloneNotSupportedException { // 이런식으로 해버리면
  	return super.clone();
  }
  
  public void copy() throws CloneNotSupportedException { // 매번 사용하는 곳에서 throws를 선언해줘야한다.
  	x.clone()
  }
  ```

또한 상속해서 쓰기 위한 클래스 설계 방식 두가지 중 어느 쪽에서든 상속용 클래스는 Cloneable을 구현해서는 안된다. 그 이유는 Object의 방식을 모방할 수도  있기 때문이다. 따라서 상속관계에서 clone() 방법은 다음과  같다.

1. 제대로 작동하는 clone 메소드를 구현해 protected로 두고 CloneNotSupportedException도 던질 수 있다고 선언하는 것이다.
2. clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 할 수 도 있다.

* 하위 클래스에서 Cloneable를 지원하지 못하게 하는 clone 메소드

  ```java
  @Override
  protected Object clone() throws CloneNotSupportedException {
  	throw new CloneNotSupportedException();
  }
  ```

그리고 마지막으로 Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메소드 역시 적절히 동기화해줘야 한다. Object의 clone 메소드는 전혀 동기화를 신경 쓰지 않았다. 그러니  super.clone 호출 외에 다른 할일이 없더라도 clone을 재정의하고 동기화 해주어야한다.

<br>

## 정리하자면...

Cloneable의 clone 메소드를 요약하자면 다음과 같다. 

> 1. Cloneable을 구현하는 모든 클래스는 clone을 재정의해야한다. 
> 2. 이때 접근자는 public으로, 반환 타입은 클래스 자신으로 변경한다.
> 3. clone 메소드는 super.clone을 먼저 호출한 후 필요한 필드를 전부 적절히 수정한다.
> 4. 일반적으로 객체의 내부 깊은 구조에 숨어 있는 모든 가변객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 한다.
> 5. 이러한 내부 복사는 주로 clone을 재귀적으로 호출해 구현하지만, 이 방식이 최선은 아니다.
> 6. 기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 아무 필드도 수정할 필요가 없다.
> 7. 단 일련번호나 고유 ID는 비록 기본 타입이나 불변일지라도 수정해줘야 한다.

그런데 이 모든 작업이 꼭 필요한 것은 아니다. Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 한다. `그렇지 않은 상황에서라면 복사 생성자와 복사 팩토리라는 더 나은 객체 복사 방식을 제공할 수 있다.` 여기서 말한 복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다. 그리고 복사 팩토리는 정적 팩토리 메소드를 말한다.

* 복사 생성자

  ```java
  public Yum(Yum yum) { ... }
  ```

* 복사 팩토리

   ```java
   public static Yum newInstance(Yum yum) { ... }
   ```

복사 생성자와 복사 팩토리는 Cloneable/clone 방식보다 나은 면이 많다. 언어 모순적이고 위험천만한 객체 생성 메커니즘(생성자를 쓰지 않고 객체를 생성하는 방식)을 사용하지 않으며, 엉성하게 문서화된 규약에 기대지 않고, 정상적인 final 필드 용법과도 충돌하지 않으며, 불필요한 검사 예외를 던지지 않고, 형변환도 필요 없다.

또한 복사 생성자와 복사 팩토리는 해당 클래스가 구현한 '인터페이스' 타입의 인스턴스를 인수로 받을 수 있다. 예컨대 관례상 모든 범용 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공한다. 이들을 이용하면 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다. 다음 예를 통해 HashSet에서 TreeSet으로 변경할 수 있다.

* HashSet에서 TreeSet으로 복사하는 방법

  ```java
  public class HashSet {
  	List<T> s;
  
  	...
  	
  	public TreeSet cloneTreeSet() {
  		return new TreeSet<>(s);
  	}
  }
  ```

따라서 이러한 방법도 잘 확인해서 clone을 하면 좋을 것 같다.

