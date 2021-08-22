# Item 56 : 공개된 API에는 항상 문서화 주석을 작성하라

## 서론

만약 API가 사용가능하면, 반드시 문서화 해야한다. Java 환경에서는 ```Javadoc utility``` 로 이 작업을 쉽게 할수 있다. ```Javadoc``` 은 소스코드로 부터 API 문서를 자동으로 생성한다. 

## 모든 공개된 리소스에 주석을 달것

- API를 올바로 문서화 하려면 공개된 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.

## 메소드용 문서화 주석에는 규약을 명료하게 기술할것

- 메소드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.
- 메서드가 어떻게 동작하는지를 적는게 아니라 무엇을 하는지 기술해야 한다.
- 클라이언트가 해당 메서드를 호출하기 위한 전제조건을 모두 나열해야한다.
- 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건도 나열해야한다.
- 전제조건은 @throw 태그로 암시적으로 기술한다.

## 전제조건과 사후조건 뿐 아니라 부작용도 문서화 하라

- 부작용이란 사후조건으로 명확히 나타나지는 않지만 시스템의 상태에 따라 어떠한 변화를 가져오는 것을 의미한다.
- 예를들어 메서드에서 백그라운 스레드를 실행시킨다면, 이를 밝혀야 한다.

## 문서화 태그

- @param
  - 메서드의 파라미터에 대한 정보
- @return
  - 반환타입이 void가 아니라면 반환 타입 명시
- @throws
  - 발생가능성이 있는 예외 명시
- @code
  - 태그로 감싼 내용을 코드용 폰트로 렌더링
- @implSpec
  - 해당 메서드와 하위 클래스 사이의 관계를 설명하여, 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 호출할때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용할 수 있도록 해야한다.

``` java
/**
* Returns the element at the specified position in this list.
*
* <p>This method is <i>not</i> guaranteed to run in constant time.
* In some implementations it may run in time proportional to the element position.
* @param index index of the element to return
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
*         ({@code index < 0 || index >= size()})
*/
E get(int index);
```

## 요약설명

위에서 ``` Returns the element at the specified position in this list.``` 에 해당한다. 문서화 주석의 (주어가 없는) 동사구 이다.

## 색인 기능

- 자바 9부터는 JavaDoc이 생성한 HTML 문서에 대해 검색(색인) 기능이 추가되어 광대한 API문서를 누비는 일이 수월해졌다.

  ``` java
  This method compiles with the {@index IEEE 754} standard.
  ```

## 제너릭 타입이나 제너릭 메서드의 주석

- 제네릭 타입이나 제네릭 메서드를 문서화 할 때는 모든 타입 매개변수에 주석을 달아야 한다.

```java
 /* 
 * An object that maps keys to values.  A map cannot contain duplicate keys;
 * each key can map to at most one value.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> {
```

## 열거타입에는 상수별로 주석을 달아라

- 열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다.
- 열거 타입 자체와 열거 타입의 public 메서드도 물론이다.

```java
/**
* An instrument section of a symphony orchestra
*/
public enum OrchestraSection {
    /** WoodWinds, such as flute, clarinet and oboe */
    WOODWIND,
    /** Brass instruments, such as french horn and trumper */
    BRASS,
    /** Percussion instruments, such as timpani, cymbals */
    PERCUSSION,
    /** Stringed instruments, such as violin and cello */
    STRING
```

## 애너테이션 타입을 문서화 할 때는 멤버에도 주석을 달아라

- 애너테이션 타입 자체도 물론이다.
- 필드 설명은 명사구로 한다.
- 애너테이션 타입의 요약 설명은 프로그램 요소에 이 애너테이션을 단다는 것이 어떤 의미인지를 설명하는 동사구로 한다.

```java
/**
 * Indicates that the annotated method is a test method that 
 * must throw the designated exception to pass
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     *  The exception that the annotated test method must throw
     *  in order to pass. (The test is permitted to throw any subtype
     *  of the type described by this class object.)
     */
    Class<? extends Throwable> value();
}
```

## 패키지를 설명하는 문서화 주석은 package-info.java에 작성한다.

- 패키지를 설명하는 문서화 주석은 package-info.java에 명시한다.
- 패키지 선언을 반드시 포함해야 하며 패키지 선언관련 애너테이션을 추가로 포함할 수도 있다.
- 모듈 시스템을 사용한다면, module-info.java 파일에 작성하면 된다.

## 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.

- 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.
- 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.

## JavaDoc은 메서드 주석을 상속 시킬 수 있다.

- 문서화 주석이 없는 API 요소를 발견하면 JavaDoc이 가장 가까운 문서화 주석을 찾아준다.
- 상위 클래스보다 구현한 인터페이스 주석을 더 먼저 찾는다.
- @inheritedDoc 태그를 사용해 상위 타입의 문서화 주석 일부를 상속할 수 있다.
- 클래스는 자신이 구현한 인터페이스의 문서화 주석을 재사용할 수 있다.

## 정리

- 문서화 주석은 API를 문서화 하는 가장 훌륭하고 효과적인 방법이다.
- 공개 API라면 빠짐없이 설명을 달아야 한다.
- 표준 규약을 일관되게 지키자.
- 문서화 주석이외에 HTML 태그를 사용할 수 있다.
