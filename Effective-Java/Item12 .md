# Item 12 : 항상 TOSTRING 함수를 오버라이드 하라.

```PhoneNumber``` 라는 클래스가 있을때, ```toString``` 함수를 정의하지 않으면, ```{Jenny=PhoneNumber@163b91}``` 라는 값을 리턴한다. 하지만 이 보다는 ```{Jenny=707-867-5309}``` 와 같은 형식이 더 우용한 정보를 제공한다. 

```toString``` 함수는 가급적 객체가 담고있는 모든 정보를 리턴하는 것이 좋다. 하지만 String으로 변환하기 적절하지 않은 경우도 있다. 보통 그럴 경우 ```toString``` 함수는 최대한 요약한 정보를 리턴해야한다. (```Thread[main,5 main].```)

```toString``` 함수를 구현할때, 결정해야하는 것중 하나는 포맷을 명시할지 말지이다. 만약 포맷을 명시한다면, 표준적이고 가독성이 좋은 문자열을 리턴한다는 장점이 있다. 많은 value class(```BigInteger,BigDecimal```) 들이 포맷을 명시하고 있다. 단점은 한번 포맷을 명시하면, 나중에 변경하기가 어렵다는 것이다. 이미 다른 코드들이 포맷에 의존해서 구현이 되어있을 수 있기 때문이다. 포맷을 명시하든 안하든, 문서화를 통해 의도를 분명히 알려야한다.

다음은 포맷을 명시한 예이다.

``` java
/* string은 12개의 문자로 구성되어있으며, 포맷은 다음과 같다.
"XXX-YYY-ZZZZ", XXX는 area code이고, YYY는 prefix 이고, ZZZZ는 line number이다.
*/
@Override public String toString() {
  return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```

다음은 포맷을 명시하지 않은 예이다.

``` java
/* 세부사항은 추후에 바뀔수 있음. 다음의 경우처럼 보통 리턴함. 
"[Potion #9: type=love, smell=turpentine, look=india ink]" */
@Override public String toString(){...}
```

포맷을 명시하든 그렇지 않든간에, 항상 클래스에 있는 변수들에 접근가능한 메소드를 정의하는것이 중요하다. 예들들어, ```PhoneNumber``` 클래스는 area code, prefix, lineNum에 대한 메소드를 정의해야한다. 

## 정리

super class가 ```toString``` 함수를 정의해놓지 않았으면, 작성한 모든 클래스에서 ```toString``` 함수를 정의하라. 클래스를 사용하기가 편리해지고 디버깅하기 수월해질 것이다. ```toString``` 함수를 구현할때는, 정확하고 유용한 정보를 깔끔한 포맷으로 리턴하도록 구현하라. 