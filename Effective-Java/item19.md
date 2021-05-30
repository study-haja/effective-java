# Item 19 : 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

 상속을 고려한 설계와 문서화란 정확히 말하면 다음과 같다.

<br>

## 상속을 고려한 설계와 문서화

> 1. 메소드를 재정의하면 어떤 일이 일어나는지를 정확히 정리하여 문서로 남겨야한다. 달리 말하면 상속용 클래스는 재정의할 수 있는 메소드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.

 클래스의 API로 공개된 메소드에서 클래스 자신의 또다른 메소드를 호출할 수 있다. 그런데 마침 호출되는 메소드가 재정의 가능 메소드라면 그 사실을 호출하는 메서드의 API 설명에 적시해야 한다. 덧붙혀서 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 한다.

 즉 재정의 가능 메소드를 호출 할 수 있는 모든 상황을 문서로 남겨야 한다.

 대표적인 예로 `@implSpec`를 들 수 있다. Implementation Requirements로 그 메소드의 내부 동작 방식을 설명하는 어노테이션이다. 다음 예를 들어보자.

* java.util.AbstractCollection의 @implSpec 설명 예

```java
    /**
     ...
     * @implSpec
     * This implementation iterates over the collection looking for the
     * specified element.  If it finds the element, it removes the element
     * from the collection using the iterator's remove method.
     *
     * <p>Note that this implementation throws an
     * {@code UnsupportedOperationException} if the iterator returned by this
     * collection's iterator method does not implement the {@code remove}
     * method and this collection contains the specified object.
   	 ...
     */
    public boolean remove(Object o) {
      ...
    }
```

 이 설명에 따르면 iterator 메소드를 재정의하면 reomve 메소드의 동작에 영향을 준다고 한다. 또한 iterator 메소드로 얻은 반복자의 동작이 remove 메소드의 동작에 영향을 미친다고 한다. 

 이렇게 메소드의 동작을 알려주는데 좋은 API 문서를 만드는 것과 상당히 대조적이다. 그 이유는 좋은 API 문서는 어떻게가 아닌 무엇을 설명 해야하기 때문이다. 즉 상속이 캡슐화를 해치기 때문에 안전하게 상속시키려면 내부 구현 방식을 설명해야만 한다.

 여담으로 @implSpec은 자바 8에서 처음 도입되어 자바 9에서 본격적으로 사용하기 시작했다. 이 태그가 기본값으로 활성화되어야 바람직하다고 생각하지만 자바 11의 자바독에서도 선택사항으로 남겨졌다. 이 태그를 활성화하려면 명령줄 매개변수로 `-tag "implSpec:a:Implementation Requirements:"`를 지정해 주면 된다.

<br>

> 2. 효율적인 하위 클래스를 큰 어려움 없이 만들수 있게 하려면, 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅을 잘 선별하여 protected 메소드 형태로 공개해야 할 수도 있다.

 예를 들면 AbstractList의 removeRange를 들 수 있다.

* AbstractList의 removeRange()

```java
 /**
     ...
     
     * @implSpec
     * This implementation gets a list iterator positioned before
     * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
     * followed by {@code ListIterator.remove} until the entire range has
     * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
     * time, this implementation requires quadratic time.</b>
     
		 (이 메소드는 fromIndex에서 시작하는 리스트 반복자를 얻어 모든 원소를 제거할 때까지 
		 ListIterator.next와 ListIterator.remove를 반복호출하도록 구현되어 있다.
		 주의 : ListIterator.remove가 선형 시간이 걸리면 이 구현의 성능은 제곱에 비례한다.)
		 
     ...
     */
    protected void removeRange(int fromIndex, int toIndex) { //protected 필드
       ...
    }
```

 위 예제에서 List의 구현체의 최종 사용자는 removeRange 메소드에 관심이 없다. 그럼에도 이 메소드를 제공한 이유는 단지 **하위 클래스에서 부분리스트의 clear 메소드를 고성능으로 만들기 쉽게 하기위해서다.** removeRange 메소드가 없다면 하위 클래스에서 clear 메소드를 호출하면 제곱에 비례해 성능이 느려지거나 부분리스트의 메커니즘을 밑바닥부터 새로 구현해야 했을 것이다.

 그렇다면 상속용 클래스를 설계할 때 어떤 메소드를 protected로 노출해야 할지는 어떻게 결정할까? 안타깝게도 없다. 심사숙고 예측해본 다음, 실제 하위 클래스를 만들어 시험해보는 것이 최선이다. protected 메소드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 적어야 하며, 그렇다고 너무 적게 해서 상속으로 얻는 이점마저 없애지 않도록 주의해 한다.

 즉 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다. 꼭 필요한 protected 맴버를 놓쳤다면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러난다. 거꾸로, 하위 클래스를 여러 개 만들 때까지 전혀 쓰이지 않는 protected 멤버는 사실 private이었어야 할 가능성이 크다. 그리고 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야한다.

<br>

> 3. 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메소드를 호출해서는 안된다.

 이 규칙을 어기면 프로그램이 오동작할 것이다. 그 이유는 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메소드가 하위 클래스의 생성자보다 먼저 호출되기 때문이다. 다음 예를 보자.

* 상위생성자 호출로 인한 오류

```java
public class Test {
    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}

class Super {
    public Super() {
        overrideMe(); // 하위 클래스 생성 하지 않아 Instant값은 null
    }

    public void overrideMe() { }
}

class Sub extends Super{
    private final Instant instant;

    public Sub() {
        instant = new Instant();
    }

    @Override
    public void overrideMe() {
        System.out.println(instant);
    }
}

class Instant { }
```

 이 예제에서 intant를 2번 호출할꺼라 예상했지만, 첫번째는 null을 출력하고 2번째는 instant를 출력한다. 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 override를 호출했기 때문이다. 즉 NullPointerException을 던지게 되어 치명적인 오류를 발생시킬 수 있다.

 더 나아가 Cloneable과 Serializable 인터페이스는 상속용 설계에 맞지 않다. 그 이유는 사용한 클래스에 확장하려는 프로그래머에게 엄청난 부담감을 주기 때문이다. 물론 이 인터페이스들을 하위 클래스에서 구현하도록 특별한 방법도 있지만 되도록 사용을 안하는 것이 좋다.

 clone과 readObject 메소드는 생성자와 비슷한 효과를 낸다. 따라서 상속용 클래스에서 Cloneable이나 Serializable을 구현할지 정해야 한다면, 이들을 구현할 때 따르는 제약도 주의해야 한다. **즉 clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메소드를 호출해서는 안된다.** 

 readObject의 경우 역직렬화되기 전에 재정의한 메소드부터 호출하게 된다. 그리고 clone의 경우 하위 클래스의 clone 메소드가 복제본의 상태를 수정하기 전에 재정의한 메소드를 호출한다. 그리고 clone은 잘못되면 복제본뿐만 아니라 원본을 수정시킬 수 있기 때문에 조심해야 한다. 이렇기에 프로그램 오작동으로 이어질 수 있어 조심해야 한다.

 Serializable을 구현한 상속용 클래스가 readResolve나 writeReplace 메소드를 갖는다면 이 메소드들은 private이 아닌 protected로 선언해야 한다. 그 이유는 private으로 선언하면 하위 클래스는 하위 클래스에서 무시되기 때문이다. 이 역시 상속을 허용하기 위해 내부 구현을 클래스 API로 공개하는 예 중 하나다.

<br>

## 정리하면

 설명한것을 토대로 정리하자면 클래스를 상속용으로 설계하려면 엄청난 노력과 제약이 따른다. 그렇다면 구체 클래스의 상속은 어떨까? 

 전통적으로 상속용으로 설계되어있다던지 문서화라던지 되어있지 않지만 그대로 두면 위험하다. 그 이유는 클래스에 변화가 생길 때마다 하위 클래스를 오동작하게 만들 수 있기 때문이다. 

 이 문제를 해결하기 위해서는 가장 좋은 방법으로 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이 좋다. 즉 final로 선언하거나 정적 팩토리 메소드를 사용해서 막아주면 된다. 

 만약 구체 클래스라도 허용해야겠다면 클래스 내부에서는 재정의 가능 메소드를 사용하지 않게 만들고 이 사실을 문서로 남기는 것이 최선이다. 재정의 가능 메소드를 호출하는 자기 사용코드를 완벽히 제거하라는 말이다. 이러면 메소드를 재정의해도 다른 메소드의 동작에 아무런 영향을 주지 않는다. 또다른 방법으로 재정의 가능 메소드는 자신의 본문 코드를 private 도우미 메소드로 옮기고, 이 도우미 메소드를 호출하도록 수정한다. 그런 다음 재정의 가능 메소드를 호출하는 다른 코드들도 모두 이 도우미 메소드를 직접 호출하도록 수정하면 된다.

