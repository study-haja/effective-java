# Item33 : 타입 안전 이종 컨테이너를 고려하라

제네릭은 `Set<E>`, `Map<K, V>`등의 컬렉션과 `ThreadLocal<T>`, `AtomicReference<T>` 등의 단일 원소 컨테이너에도 흔히 쓰인다. 이렇게 매개변수화되는 대상은 컨테이너 자신이기 때문에 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다.

예를 들어 Set에서 원소의 타입을 뜻하는 단 하나의 타입 매개변수만 있으면 되고, Map에서는 key와 value의 타입 2개만 필요하다는 식으로 보면 된다.

하지만 종종 프로그램을 구현하다보면 좀 더 유연한 수단이 필요할 때도 있다. 이는 쉽게 해결 할수 있는데 바로 타입 안전 이종 컨테이너 패턴으로 해결하면 된다.



###  타입 안전 이종 컨테이너 패턴

이 패턴은 다음과 같다. 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 것이다. 다음 예를 보자.

* 타입 안전 이종 컨테이너 패턴 - API

```java
public class Favorites {
	public <T> void putFavorite(Class<T> type, T instance);
	public <T> T getFavorite(Class<T> type);
}
```

위의 Favorites 클래스는 타입 안전 이종 컨테이너 패턴을 사용한 예이다. 즉 각 타입의 Class 객체를 매개변수화한 키 역할로 사용하고, 해당 키로 값을 불러오면 되는 것이다.

이 방식이 동작하는 이유는 class의 클래스가 제네릭이기 때문이다. class의 리터럴 타입은 `Class<T>`이다. 즉 `Class<String>` 은 String.class의 타입이고 `Class<Integer>`은 Integer.class의 타입인 것이다.

또한 타입 토근이란 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메소드들이 주고받는 class 리터럴 타입을 말한다.

따라서 위의 Favorites 클래스를 사용한 예는 다음과 같이 사용할 수 있다.

* 타입 안전 이종 컨테이너 패턴 - 클라이언트

```java
public static void main(String[] args) {
  Favorites f = new Favorites();
  
  f.putFavorite(String.class, "Java");
  f.putFavorite(Integer.class, "0xcafebabe");
  f.putFavorite(Class.class, Favorites.class);
  
  String favoriteString = f.getFavoritie(String.class);
  String favoriteString = f.getFavoritie(Integer.class);
  String favoriteString = f.getFavoritie(Class.class);
 
  // Java cafebabe Favorites를 출력 
  System.out.println("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
}
```

Favorites 인스턴스는 type safe하다. 그리고 타입이 제각각이라, 일반적인 맵과 달리 여러가지 타입의 원소를 담을 수 있다. 따라서 Favorites는 타입 안정 이종 컨테이너라 부를 수 있다. 그러면 이번에는 구현을 한번 보자.

* 타입 안전 이종 컨테이너 패턴 - 구현

```java
public class Favorites {
  private Map<Class<?>, Object> favorites = new HashMap<>();
  
	public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
  }
  
	public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
  }
}
```

여기서 Map의 타입은 비한정적 와일드 카드 타입을 써서 아무것도 넣을 수 없다고 생각할 수 있지만, 사실 와일드카드 타입이 중첩되어있다는점을 알아야 한다. 즉 맵이 아니라 키가 와일드카드 타입인 것이다. 이는 모든 키가 서로 다른 매개변수화 타입일 수 있다는 뜻으로, 첫번째는 `Class<String>`, 두번째는 `Class<Integer>` 식으로 될 수 있다는 것이다.

다음으로 favorites 맵의 값 타입은 단순히 Object라는 것이다. 즉 키와 값 사이의 타입 관계를 보증하지 않는다. 자바에서는 이 관계를 명시할 방법이 없지만, 우리는 이 관계가 성립함을 알고 있고, 즐겨찾기를 검색할때 그 이점을 누리게 된다.

putFavorite 메소드를 보면 키와 값 사이의 타입 링크 정보는 버려진다. 즉, 그 값이 그 키 타입의 인스턴스라는 정보가 사라진다. 하지만 getFavorite에서 이 관계를 되살릴 수 있으니 상관없다.

getFavorite 코드를 보면 주어진 Class 객체에 해당하는 값을 favorites 맵에서 꺼낸다. 이 객체가 바로 반환해야 할 객체가 맞지만, 잘못된 컴파일 타입을 가지고 있다. 이 객체의 타입은 Object이니, 이를 T로 바꿔 반환해야 한다. 따라서 Class의 cast 메소드를 사용해 이 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환한다.



### Class의 cast() 메소드

Cast 메소드는 형변환 연산자의 동적 버전이다. 이 메소드는 단순히 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지를 검사한 다음, 맞다면 그 인수를 그대로 반환하거나, 아니면 ClassCastException을 던진다. 다음 예가 cast 메소드 이다.

* Class의 cast 메소드

```java
public class Class<T> {
	T cast(Object obj);
}
```

그런데 cast 메소드가 단지 인수를 그대로 반환하기만 한다면 왜 굳이 사용하는 걸까? 이유는 간단하다. T로 비검사 형변환하는 손실 없이도 위와 같은 코드를 안전하게 사용할 수 있기 때문이다. 



### 타입 안전 이종 컨테이너 패턴 심화 버전

위에서 봤던 Favorites 클래스를 보면 알아두어야 할 제약이 두가지 있다.

1. 악의적인 클라이언트가 Class 객체를 제네릭이 아닌 로 타입으로 넘기면 Favorites 인스턴스의 타입 언정성이 쉽게 깨진다.

다음 예를 보자

* Type Un-Safe

```java
// 비검사 경고
f.putFavorite((Class)Integer.class, "Integer의 인스턴스가 아닙니다.");
int favoriteInteger = f.getFavorite(Integer.class);

HashSet<Integer> set = new HashSet<>();
((HashSet)set).add("문자열 입니다.");
```

물론 사용자가 어떤 타입을 사용해야 하는지 알겠지만 악의적으로 사용한다면 위와 같이 타입 안정성에 위협을 받을 수 있다. 하지만 우리는 한번더 타입 불변식을 어기지 않게 보장 할 수 있다. 바로 putFavorite 메소드의 인수로 주어진 instance의 타입이 type으로 명시한 타입과 같은지 확인하면 된다. 다음 예가 그렇다.

* 동적 형변환으로 런타임 타입 안전성 확보

```java
public <T> void putFavorite(Class<T> type, T instance) {
	favorites.put(Objects.requireNonNull(type), type.cast(instance));
}

// 런타임 타입 안정성을 확보한 후에는 다음처럼 호출하면 ClassCastException 발생
f.putFavorite((Class)Integer.class, "Integer의 인스턴스가 아닙니다.");
```

이렇게 타입을 체크하면서 사용하면 문제없이 사용할 수 있다. 이 방식은 Collections 에서 checkedSet, checkedList, checkedMap 같은 메서드에서 사용한다. 해당 메소드는 제네릭이라 Class 객체와 컬렉션의 컴파일타임 타입이 같음을 보장하고 내부 컬렉션들을 실체화한다. 이렇기 때문에 오류가 발생했을때, stack trace에서도 많은 도움을 준다.

2. 실체화 불가 타입에는 사용할수 없다.

putFavorite에서는 String이나 String[]은 저장 할수 있어도 `List<String>`은 저장을 하지 못한다. 그 이유는` List<String>`용 Class 객체를 얻을 수 없기 때문이다. `List<String>.class`, `List<Integer>.class`은 `List.class` 객체로 변환되기 때문에 알 방법이 없다. 

그렇다고 해결할 방법이 아예 없는건 아니다. 바로 슈퍼 타입 토큰으로 해결하면 된다. 닐 개프터가 고안한 방식으로, 실제로 아주 유용하며 스프링에서 `ParamterizedTypeReference`라는 클래스로 미리 구현했다. 다음 예를 보자.

* 슈퍼 타입 토큰을 적용한 Favorites 클래스 사용

```java
public static void main(String[] args) {  
  Favorites f = new Favorites();
  
  List<String> pets = Arrays.asList("개", "고양이", "앵무");
  
  f.putFavorite(new. TypeRef<List<String>>(){}, pets);
  List<String> listofStrings = f.getFavorite(new TypeRef<List<String>>(){});
}
```

이렇게 사용하여 2번 문제를 해결 할수 있다. 하지만 슈퍼 타입 토큰도 완벽하지 않으니 주의해서 사용해야 한다. 



### 한정적 타입 토큰

Favroties가 사용하는 타입 토큰은 비한정적이다. 즉 getFavorite, putFavorite은 어떤 Class 객체든 받아들인다. 때로는 이 메소드들이 허용하는 타입을 제한하고 싶을 수 있는데 한정적 타입 토큰을 활용하면 가능하다.

한정적 타입 토큰이란 단순히 한정적 타입 매개변수나 한정적 와일드카드를 사용해 표현 가능한 타입을 제한하는 타입토큰이다. 다음 AnnotatedElement 인터페이스에 선언된 메소드를 보자.

* 한정적 타입 토큰을 사용한 코드

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```

annotationType 인수는 어노테에션 타입을 뜻하는 한정적 타입 토큰이다. 이 메소드는 토큰으로 명시한 타입의 어노테이션이 대상 요소에 달려있다면 그 어노테이션을 반환하고, 없다면 null을 반환한다.

즉 어노테이션된 요소는 그 키가 어노테이션 타입인, 타입 안정 이종 컨테이너인 것이다.

`Class<?>` 타입의 객체가 있고, 이를 getAnnotation 처럼 한정적 타입 토큰을 받는 메소드에 넘기려면 어떻게 해야 할까? 물론 `Class<? Extends annotation>`으로 형변환할 수 있지만, 이 형변환은 비검사이므로 컴파일을 하면 경고가 발생할 것이다. 

그렇기 때문에 Class 클래스에서 제공해주는 asSubclass 메소드로, 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환 해주면 된다. 형변환이 되면 이 클래스가 인수로 명시한 클래스의 하위 클래스라는 것이기 때문에, 성공하면 인수로 받은 클래스 객체를 반환하고, 만약 실패하면 ClassCastException을 발생시킨다. 

다음 코드가 `asSubclass` 메소드를 사용해 런타임에 읽어내는 예이다.

* asSubclass를 사용해 한정적 타입 토큰을 안전하게 형변환

```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
  Class<?> annotationType = null;
  try {
    annotationType = Class.forName(annotationTypeName);
  } catch(Exception ex) {
    throw new IllegalArgumentException(ex);
  }
  // asSubClass 메서드는 오류나 경고 없이 컴파일 된다.
  return element.getAnnotaiton(annotationType.asSubclass(Anntotation.class));
}
```

