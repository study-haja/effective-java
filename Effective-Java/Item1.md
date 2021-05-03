# Item1 - 생성자 대신 정적 팩토리 메소드를 고려하라



클라이언트가 클래스의 인스턴스를 얻는 수단은 2가지의 방법이 있다.

1. public 생성자로 인스턴스를 제공하는 방법

2. 생성자와 별도로 정적 팩토리 메서드로 제공하는 방법 - **클래스 인스턴스를 반환하는 단순한 정적 메소드**

   * Ex)

     ```java
     public static Boolean valueOf(boolean b) {
       return b ? Boolean.TRUE : Boolean.FALSE;
     }
     ```

2번째 방법은  public 생성자 대신 정적으로 생성자를 제공할 수 있다. 이 방식에는 장점과 단점이 모두 존재한다. 이 장단점을 지금 부터 알아보려고 한다.

### 정적 팩토리 메소드의 장점

1. **이름을 가질 수 있다.**

   생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다. 하지만 정적 팩토리 메소드를 사용하면 반환될 객체의 특성으르 쉽게 묘사할 수 있다.

   

   다음 멤버의 클래스의 인스턴스를 생성할 때 이름, 나이, 전화번호를 받는 생성자와 이름, 나이만 받는 생성자가 있다고 하자.

   ```java
   // public 생성자로 인스턴스를 제공하는 방법
   class Member {
     private final String name;
     private final int age;
     private final String phoneNumber;
   
     // 이름, 나이, 전화번호 받는 생성자
     public Member(String name, int age, String phoneNumber) {
       this.name = name;
       this.age = age;
       this.phoneNumber = phoneNumber;
     }
     
     // 이름, 나이 받는 생성자
     public Member(String name, int age) {
       this.name = name;
       this.age = age;
       this.phoneNumber = "none";
     }
   }
   ```

   ```java
   public Member enrollMember(MemberDto memberDto) {
   	memberRepository.save(
       new MemberDto(memberDto.getName(), memberDto.getAge(), memberDto.getPhoneNumber())
     );
     
     memberRepository.save(
       new MemberDto(memberDto.getName(), memberDto.getAge())
     );
     
     ...
   }
   ```

   이 방법은 멤버를 저장 할때 어떤 인스턴스를 생성하는지 쉽게 구분하지 못한다. 

   물론 이 방법도 좋은 방법이지만 생성자가 각 필드마다 만들어 줘야 하거나, Dto를 직접 줄 경우등 생성자가 점점 늘어나면 개발자는 기억하기 어려워 엉뚱한 것을 호출하거나 실수를 할 수 있다. 또한 코드를 읽는 사람도 클래스 설명 문서를 찾아보지 않고는 의미를 알지 못 할 것이다.

   

   하지만 정적 팩토리 메소드 방법을 사용해 본다면?

   ```java
   // public 생성자로 인스턴스를 제공하는 방법
   class Member {
     private final String name;
     private final int age;
     private final String phoneNumber;
   
     // 이름, 나이, 전화번호 받는 생성자
     public Member(String name, int age, String phoneNumber) {
       this.name = name;
       this.age = age;
       this.phoneNumber = phoneNumber;
     }
     
     // 이름, 나이 받는 생성자
     public Member(String name, int age) {
       this.name = name;
       this.age = age;
       this.phoneNumber = "none";
     }
     
     public static Member newInstance(String name, int age, String phoneNumber) {
       return new Member(name, age, phoneNumber);
     }
     
     public static Member newInstanceExceptPhoneNumber(String name, int age) {
       return new Member(name, age);
     }
   }
   ```

   ```java
   public Member enrollMember(MemberDto memberDto) {
   	memberRepository.save(
       Member.newInstance(memberDto.getName(), memberDto.getAge(), memberDto.getPhoneNumber())
     );
     
     memberRepository.save(
       Member.newInstanceExceptPhoneNumber(memberDto.getName(), memberDto.getAge())
     );
     
     ...
   }
   ```

   다음과 같이 볼 수 있듯이 네이밍을 통해 어떤 일을 하는지 명확하게 보여주고 있다. 

   

   즉 이름을 가질 수 있는 정적 팩토리 메서드에는 이런 제약이 없다. 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩토리 메서드로 바꾸고 각각의 차이를 잘 드러나는 이름을 지어주는 것이 효과적이라 본다.

   

2. **호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.**

   이로 인해 불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다. 

   특히 생성 비용이 큰 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어 올 수 있다. 

   

   Boolean 객체의 valueOf() 메소드를 보자.

   ```java
   public static final Boolean TRUE = new Boolean(true);
   public static final Boolean FALSE = new Boolean(false);
   
   // Boolean.class 안에 있는 메소드 
   public static Boolean valueOf(boolean b) {
      return (b ? TRUE : FALSE);
    }
   ```

   ```java
   ...
   
   public Boolean toBoolean(boolean flag) {
     return flag ? Boolean.valueOf(true) : Boolean.valueOf(false);
   }
   ```

   해당 예제는 boolean 데이터 타입을 통해 Boolean 객체를 만들어주는 메소드이다. 여기에 중요한 점은 valueOf를 통해 인스턴스를 매번 생성을 안해줘도 된다는 점이다. 위에 보면 static final 로 변하지 않은 전역 Boolean 객체를 미리 생성해 사용한다. 즉 매번 값비싼 Boolean 객체를 생성하는 것 보다 미리 생성되어있는 인스턴스를 반환하기 때문에 성능을 상당히 끌어준다.

   

   또한 정적 팩토리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지 통제해준다. 이런 클래스를 통제 클래스라 한다. 하지만 여기서 의문이 들었다. 인스턴스를 통제하는 이유는 무엇일까? 이펙티브 자바에서는 다음과 같이 말했다.

   1. 인스턴스를 통제하면 클래스를 싱글턴으로 만들 수도, 인스턴스화 불가로 만들 수 도 있다.
   2. 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장 할 수 있다.(a == b일 때만 a.equals(b) 성림)
   3. 인스턴스 통제는 플라이 웨이트 패턴의 근간이 되며, 열거 타입은 인스턴스가 하나만 만들어짐을 보장한다.

   즉 통제에 따라 불필요한 인스턴스 생성 관리를 해주거나 불변을 보장해주는 즉 말 그대로 컨트롤을 할 수 있다고 생각한다.

   

3. **반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.**

   결론 부터 말하자면 반환할 객체의 클래스를 자유롭게 선택할 수 있기 때문에 ***유연성*** 이 높다. 즉 API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환 할 수 있어 API를 작게 유지할 수 있다고 한다. 

   

   이해가 쉽지 않으니 아래 예제를 보자

   ```java
   // stream에 Collectors 라이브러리
   public final class Collectors {
   	
     public static <T> Collector<T, ?, List<T>> toList() {
       return new CollectorImpl<>((Supplier<List<T>>) ArrayList::new, List::add,
                                      (left, right) -> { left.addAll(right); return left; },
                                      CH_ID);
     }
     
     public static <T> Collector<T, ?, Set<T>> toSet() {
       return new CollectorImpl<>((Supplier<Set<T>>) HashSet::new, Set::add,
                                      (left, right) -> {
                                          if (left.size() < right.size()) {
                                              right.addAll(left); return right;
                                          } else {
                                              left.addAll(right); return left;
                                          }
                                      },
                                      CH_UNORDERED_ID);
       }
   }
   ```

   Collectors 인스턴스를 생성하지 않아도 정적 메소드를 통해 List 타입, Set타입 등 다양하게 만들 수 있다. 

   자바 8 이전에는 인터페이스에 정적 메소드를 선언할 수 없어 동반 클래스를 만들어 그 안에서 정의 하는 것이 관례였다고 한다. 그래서 컬렉션 프레임 워크는 45개의 유틸리티 구현체를 제공했는데, 이 구현체 대부분을 단 하나의 인스턴스화 불가 클래스인 Collections에서 정적 팩토리 메소드를 통해 얻도록 했다. 자바 8이후부터 인터페이스에 정적 메소드를 선언할 수 있게 되어 동반 클래스를 구현할 필요가 없었지만 package-private 클래스에 두어야 했다.

   

4. **입력 매개변수에 따라 매번 다른 클래스의 객체를 반환 할 수 있다.**

   반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관 없다. 예를들어 EnumSet 클래스 같은 경우 public 생성자 없이 오직 정적 팩토리만 제공한다. 

   

   OpenJDK에서 원소의 수에 따라 두가지 하위 클래스 중 하나의 인스턴스를 반환하는데 64개 이하면 원소들을 long 변수 하나로 관리하는 RegularEnumSet의 인스턴스, 65개 이상이면 long 배열로 관리하는 JumboEnumSet의 인스턴스를 반환한다. 물론 해당 인스턴스가 사용할 이점이 없어지면 삭제해도 아무 문제 없고, 성능을 늘리기 위해 다른 인스턴스로 바꿔주어도 상관이 없다. 

   

   여기서 제일 중요한건 클라이언트는 이들 존재를 몰라도 EnumSet의 하위 클래스이기만 하면 자유롭게 사용이 가능하다는 것이다.

   

5. **정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**

   유연한 특성 때문에 서비스 제공자 프레임워크를 만드는 근간이 되었다. 가장 대표적인 예로 JDBC가 있다. 이 프레임워크는 클라이언트에 제공하는 역할을 프레임 워크가 통제하여, 클라이언트를 구현체로부터 분리해준다. 

   서비스 제공자 프레임 워크는 다음과 같이 3개의 핵심 요소로 이루어진다.

   1. 구현체의 동작을 정의하는 서비스 인터페이스
   2. 제공자가 구현체를 등록 할 때 사용하는 제공자 등록 API
   3. 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 서비스 접근 API

   

   3번을 봤을 때 클라이언트는 서비스 접근 API를 사용 할 때 원하는 구현체로 조건을 명시할 수 있다. 만약 조건을 명시하지 않으면 기본 구현체를 반환하거나 지원하는 구현체들을 하나씩 돌아가며 반환한다. 이것이 바로 위에서 말했던 유연성의 실체라 볼 수 있다. 그리고 더 나아가 다음 요소까지 쓰인다.

   4. 서비스 제공자 인터페이스

    해당 요소는 바로 서비스 인터페이스의 인스턴스를 생성하는 팩토리 객체를 설명해준다. 예를 들어 JDBC에서는 다음과 같이 수행한다.

   1. Jdbc Connection : 서비스 인터페이스 역할
   2. DriverManager.registerDriver : 제공자 등록 API 역할
   3. DriverManager.getConnection: 서비스 접근 API 역할
   4. Driver: 서비스 제공자 인터페이스 역할



### 정적 팩토리 메소드의 단점

1. **상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.**

   정리하자면 컬렉션 프레임워크이 유틸성 구현 클래스들은 상속할 수 없다는 것이다. 이 말은 사실 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이제약을 지켜야 한다는 것인데 오히려 장점으로 받아들일 수 있는 부분이다. 

   나중에 item17, item18에서 더 자세히 다루겠다.

   

2. **정적 팩토리 메소드는 프로그래머가 찾기 어렵다.**

   생성자처럼 API 설명에 명확히 들어나지 않는다. 따라서 개발자는 해당 정적 메소드를 직접 찾거나 인스턴스화할 방법을 알아내야하는 수고가 있다. 이 수고를 덜하려면 API 문서를 잘 써놓고 메소드 이름도 널리 알려진 규약을 따라 짓는 식으로 문제를 완화해줘야 한다.



### 정적 팩토리 메소드에서 흔히 사용하는 명명 방식들

다음은 정적 팩토리 메소드의 흔히 사용하는 명명 방식들이다.

* from() : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메소드이다.

  ```java
  Date d = Date.from(instant);
  ```

* of() : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메소드이다.

  ```java
  Set<Rank> faceCards = EnumSet.ok(JACK, QUEEN, KING)
  ```

* valueOf() : from과 of의 더 자세한 버전이다.

  ```java
  BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
  ```

* instance() or getInstance() : 매개변수를 받는다면 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.

  ```java
  StackWalker luke = StackWalker.getInstance(options);
  ```

  ```java
  //StackWalker 클래스
  ...
    
  	private final static StackWalker DEFAULT_WALKER = new StackWalker(DEFAULT_EMPTY_OPTION); 
  	...
    
  	public static StackWalker getInstance() {
      // 디폴트 워커로 보장하지 않음
      return DEFAULT_WALKER;
    }
  ```

  

* create() or newInstance() : instance혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.

  ```java
  Object newArray = Array.newInstance(classObject, arrayLen);
  ```

  ```java
  // Array 클래스
  ...
  
    public static Object newInstance(Class<?> componentType, int length) 
      throws NegativeArraySizeException {
  
      // 새로운 인스턴스 보장
      return newArray(componentType, length);
    }
    ...
  
    @HotSpotIntrinsicCandidate
    private static native Object newArray(Class<?> componentType, int length)
      throws NegativeArraySizeException;
  ```

  

* getType() : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메소드를 정의할 때 쓴다. 뒤에 Type은 팩토리 메소드가 반환할 객체의 타입이다.

  ```java
  FileStore fs = Files.getFileStore(path);
  ```

  ```java
  //Files 클래스
  ...
  
    public static FileStore getFileStore(Path path) throws IOException {
      return provider(path).getFileStore(path);
    }
  ```

* newType() : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메소드를 정의할 때 쓴다. 마찬가지로 Type은 팩토리 메소드가 반환할 객체의 타입이다.

  ```java
  BufferedReader br = Files.newBufferedReader(path);
  ```

  ```java
  //Files 클래스
  ...
    
    public static BufferedReader newBufferedReader(Path path, Charset cs)
            throws IOException {
      CharsetDecoder decoder = cs.newDecoder();
      Reader reader = new InputStreamReader(newInputStream(path), decoder);
      return new BufferedReader(reader);
    }
  ```

  

* Type() : getType과 newType의 간결한 버전이다.

  ```java
  List<Complaint> litany = Collections.list(legactLitany);
  ```

