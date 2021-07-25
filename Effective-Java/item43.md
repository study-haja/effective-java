# Item43 : 람다보다는 메소드  참조를  사용하라

람다가 익명 클래스보다 나은 점 중 가장 큰 특징은 간결함이다. 하지만 람다보다도 더 간결하게 만드는 방법이 존재한다. 바로 메소드 참조를 통해 더 간결하게 만들 수 있다. 다음 코드를 보자

* 람다를 사용한  map.merge() 코드

 ```java
 //merge 매개변수 : key, value, BiFunction(? super value, ? super vlaue, ? extends value,)
 map.merge(key, 1, (count, incr) -> count + incr);
 ```

이  코드는 키가 맵 안에 없다면 키와 숫자 1을 매핑하고, 이미 있다면 기존 매핑 값을 증가 시킨다.

깔끔해 보이지만, 매개변수인 count와 incr은 크게 하는 일 없이 공간을 꽤 차지한다. 즉 이 람다는 두 인수의 합을 단순히 반환할 뿐이다.

자바 8의 Integer 클래스는 람다와 기능이 같은 정적 메소드 sum을 제공하기 시작했다. 따라서 람다 대신 이 메소드의 참조를 전달하면 똑같은 결과를 더 보기 좋게 얻을 수 있다.

* 메소드 참조를 사용한 map.merge() 코드

```java
map.merge(key, 1, Integer::sum);
```

이렇게 메소드 참조로 제거할 수 있는 코드양도 늘어난다. 하지만 어떤 람다는 길이는 더 길지만 메소드 참조보다 읽기 쉽고  유지보수도 쉬울 수 있다. 다음 예를 보자.

* GoshThisClassNameIsHumongous 클래스 안에 있는 코드

 ```java
 service.execute(GoshThisClassNameIsHumongous::action);
 ```

이를 람다로 대체하면 다음과 같다.

* GoshThisClassNameIsHumongous 코드를 람다로 대체

```java
service.execute(() -> action());
```

위 코드를 보면 메소드 참조보다 람다 코드가 짧고 명확한 것을 볼 수 있다. 따라서 적절하게 보기 좋은 것으로 사용하면 된다.



###  메소드 참조 유형

메소드 참조 유형은 다음과 같이 다섯가지로 구분할 수 있다. (참고로 람다를 사용 못하면 메소드 참조도 사용 못한다.)

1. 정적 메소드 참조

   * Example

     ```java
     // 람다 버전
     str -> Integer.parseInt(str)
     
     // 정적 메소드 버전
     Integer::parseInt
     ```

     

2. 한정적 인스턴스 메소드 참조 :  수신 객체를 특정

   * 근본적으로  정적 메소드 참조와 비슷

   * Example

     ```java
     // 람다 버전
     Instant then = Instant.now();
     t -> then.isAfter(t);
     
     // 정적 메소드 버전
     Instant.now()::isAfter
     ```

     

3. 비한정적 인스턴스 메소드 참조 : 수신 객체를 특정하지 않음

   * 함수 객체를 적용하는 시점에서 수신 객체를 알려줌

   * 수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가

   * 그 뒤로 참조되는 메소드 선언에 정의된 매개변수들이 뒤따름

   * 주로 스트림 파이프라인에서의 매핑과 필터 함수에 사용

   * Example

     ```java
     // 람다 버전
     str -> st.toLowerCase()
     
     // 정적 메소드 버전
     String::toLowerCase
     ```

     

4. 클래스 생성자

   * Example

     ```java
     // 람다 버전
     () -> new TreeMap<K,V>()
       
     // 정적 메소드 버전
     TreeMap<K,V>::new
     ```

     

5. 배열 생성자

   * Example

     ```java
     // 람다 버전
     len -> new int[len]
     
     // 정적 메소드 버전
     int[]::new
     ```

