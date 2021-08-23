# ITEM 60 : 정확한 답이 요구되지 않으면, FLOAT와 DOUBLE을 피하라.

## float,double의 문제점

```float``` 과 ```double``` 타입은 주로 과학적,공학적 계산에 사용되도록 설계되었다. 하지만 항상 정확한 수치를 리턴하지는 않는다.

특히 환율계산 작업에는 적합하지 않다. 왜냐하면 0.1을 정확히 표현하는 것이 불가능하기 때문이다.

``` java
System.out.println(1.03 - 0.42)
```

위의 결과는 ```0.6100000000000001``` 이다.

그리고

``` java
System.out.println(1.00 - 9 * 0.10);
```

위의 결과는 ``0.09999999999999998`` 이다.

단지 반올림 함으로써 문제를 해결할 수 있다고 생각하지만, 항상 그렇게 할수 있는것은 아니다.

다음은 10cent,20cent,30cent, ..., 1 dollar 까지 candy 값을 올려가며, 최대로 살수 있는 candy의 갯수를 출력하는 코드이다.

``` java
public static void main(String[] args) {
  double funds = 1.00;
  int itemsBought = 0;
  for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price;
    itemsBought++; 
  }
  System.out.println(itemsBought + " items bought.");
  System.out.println("Change: $" + funds); 
}
```

위 코드는 float을 사용하기 때문에 부정확한 결과를 출력한다. 

## BigDecimal 을 사용해서 개선하기

이를 개선하는 방법은 ```BigDecimal``` 타입을 사용하는 것이다.

``` java
public static void main(String[] args) {
  final BigDecimal TEN_CENTS = new BigDecimal(".10"); int itemsBought = 0;
  BigDecimal funds = new BigDecimal("1.00");
  for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) { 
    funds = funds.subtract(price); itemsBought++;
  }
  System.out.println(itemsBought + " items bought."); System.out.println("Money left over: $" + funds);
}
```

위 코드는 정확한 결과를 리턴한다.

```BigDecimal```은 다음과 같은 단점이 있다.

1. primitive arithmetic type 을 사용할때보다 불편함.
2. 속도가 느림.

```BigDecimal``` 을 사용하는 대안은 ```int```, ```long``` 을 사용하는 것이다.

위 코드를  ```int```, ```long``` 을 사용하도록 수정해보자.

``` java
public static void main(String[] args) {
  int itemsBought = 0;
  int funds = 100;
  for (int price = 10; funds >= price; price += 10) {
    funds -= price;
    itemsBought++; 
  }
  System.out.println(itemsBought + " items bought.");
  System.out.println("Cash left over: " + funds + " cents"); 
}
```

## 요약

```BigDecimal``` 은 반올림 작업을 지원한다는 추가적인 장점이 존재한다. 이는 특히 법률적으로 정확한 계산을 해야하는 경우에 유용하다. 만약 성능이 필수적이라면, 소수점 계산은 프로그래머가 직접하고자 하는 의향이 있고, 값의 범위가 long 범위 안에 들어올때 ```int```  나 ```long``` 을 사용할 수 있다. 만약 18자리를 초과하는 숫자를 사용한다면, ```BigDecimal``` 을 사용하라.