# ITEM 6 : 불필요한 객체생성을 피하라

## 불필요한 객체생성의 예시

``` java
String s = new String("bikini"); // 잘못된 사용의 예, "bikini" 자체가 String 객체를 생성함.
String s = "bikini"; // 옳은 사용의 예
```

``` String s = new String("bikini") ``` 처럼 사용하면 필요없는 String 객체가 생성된다.

## Static factory method로 불필요한 객체 생성 피하기

다음 코드는 올바른 로마 숫자 표기법인지 검사하는 함수이다.

``` java
static boolean isRomanNumeral(String s) {
  return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]| ?X{0,3})(I[XV]|V?I{0,3})$");
}
```

위 구현의 문제점은 반복적인 사용에 적합하지 않은 ```String.matches()``` 함수를 사용했다는 점에있다. ```String.matches()``` 함수는 내부적으로 ```Pattern``` 객체를 생성하고, 작업이 끝나면 ```Garbage Collector```에 의해서 회수된다. ```Pattern``` 객체를 생성하는 것은 비용이 많이 든다.(Regex에 대해 finite state machine을 생성하므로)

성능을 개선하는것은 ```Pattern``` 객체를 사용해서 명시적으로 Regex를 컴파일한후, 객체를 캐싱해두어 재사용하는 것이다.

즉, 다음처럼 코드를 작성한다.

``` java
public class RomanNumerals {
  private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]| ?X{0,3})(I[XV]|V?I{0,3})$")
    
    static boolean isRomanNumeral(String s) {
    return ROMAN.matcher(s).matches();
  }
}
```

개선된 버전은 이전 버전에 비해서 **성능이 6.5배 정도 향상**이 된다. 성능이 향상될 뿐만 아니라, Pattern 객체를 명시적으로 선언하게 됨으로써 Pattern의 존재가 드러나게 되면서, **코드가 더 명료해진다**.

여기서 ```Pattern``` 객체가 필요시 초기화 함으로써(lazy initialization) 성능을 개선할수는 있다. 하지만 뚜렷한 성능의 향상이 있지 않고 추가적인 구현이 따르기 때문에 보통 추천하지 않는다.

## 불필요한 객체 생성이 발생하는 경우

### Example

다음의 코드는 불필요한 ```Long``` 객체를 생성하는 예시이다.

``` java
private static long sum() {
  Long sum = 0L;
  for (long i=0; i<=Integer.MAX_VALUE;i++) 
    sum += i;
  return sum;
}
```

sum 변수가 Long 타입으로 선언되어있기 때문에 ```sum += i``` 과정에서 ```Long``` 객체가 생성된다. sum 변수를 단순히 long primitive type으로 바꾸면 불필요한 ```Long``` 객체 생성을 막을 수 있다. 

> 교훈 : boxed primitive 보다 primitive를 선호하라, 불필요한 autoboxing에 유의하라.

## 객체 생성자체가 나쁜가?

여기서 객체 생성은 비싸기 때문에, 무조건 피해야한다는 것을 의미하는 것은 아니다. 여기서 이야기하고 싶은 것은, 작은 객체들을 생성하고 회수하는 작업은 비용이 충분히 적은 작업이라는 것이다. 명료성, 간단함을 위해서 객체를 생성하는 것은 일반적으로 좋은 것이다.

반대로, 객체생성을 무조건 피하기 위해서 자체적인 object pool을 사용하는것은 나쁜 생각이다. 왜냐하면 **코드를 지저분하게 만들고 memory footprint를 증가시키고 퍼포먼스에 해를 끼치기 때문이다.** 현대 JVM 구현체는 매우 최적화된 Garbage Collector를 갖고있고, 작은 객체에 대해서는 object pool보다 훨씬 좋은 성능을 보인다.

> memory footprint : process가 사용하는 main memory 용량

ITEM 50(새로운 객체를 생성해야될때, 존재하는 객체를 재사용하지 마라.) 와 모순되는 것처럼 보이지만, 작은 객체의 경우 객체를 재사용하면서 얻는 penelty는 객체를 중복으로 생성하면서 얻는 penelty보다 더 크다. 보통 버그와 보안적인 취약점을 만들기 때문이다. 반면에 필요없는 개체를 생성하는 것은, 단지 성능에 영향을 줄 뿐이다.

## 정리 : 객체를 재사용해야 하는 경우 

매우 무거운 객체의 경우, 객체를 재사용하는것이 성능에 좋다. 전형적인 예시는 database connection을 위한 object pool이다. connection을 설정하는 것은 충분히 비용이 높은 작업이기 때문이다.