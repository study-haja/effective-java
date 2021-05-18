# ITEM 14: COMPARABLE 인터페이스를 구현하는것을 고려하라.

``` java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s,args);
        System.out.println(s);
    }
}
```

```String``` 은 ```Comparable``` 을 구현하고 있기 때문에, 위와 같이 ```TreeSet``` 에 추가하면 알파벳순서대로 ```String```이 정렬되고 중복이 제거된다.``` Comparable``` 을 구현함으로써, 많은 generic 알고리즘과 컬렉션 구현과 호환되도록 할수 있다. 

## 일반적인 규칙

> sgn 함수는 -1,0,1 을 반환하는 함수.
>
> sgn(x) == -1 if x < 0
>
> sgn(x) == 0 if x == 0
>
> sgn(x) == 1 if x > 0

1. 모든 x,y에 대해서, sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
2. x.compareTo(y) > 0 && y.compareTo(z) > 0 이면 x.compareTo(z) > 0
3. x.compareTo(y) == 0이면,  모든 z에 대해서 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 
4. (필수는 아님) (x.compareTo(y) == 0) == (x.equals(y)) . 이 규칙은 필수가 아니지만 만약 어길경우, **주의 : 이 클래스는 equals와는 일관적이지 않은 순서를 갖고있음** 이라고 명시해야한다.

``` equals``` 함수의 일반적인 규칙과 유사하다는 사실을 알수 있고, 마찬가지로 ``` compareTo``` 규칙을 지키면서 하위 클래스에서 새로운 필드를 추가하는 것은 불가능하다. 만약 ```Comparable``` 을 구현하는 클래스에 새로운 필드를 추가하고 싶다면, 상속하지 말고 그 클래스를 포함하는 새로운 클래스를 정의해야 한다. 그리고 해당 인스턴스를 리턴해주는 view 함수를 제공해야 한다.

4번 규칙을 만족하지 않더라도 여전히 동작하지만, collection interface(```Collection```,```Set```,```Map```)의 일반적인 규칙을 위배하는 경우가 발생할 수 있다. 예를들면 ```BigDecimal``` 클래스는 ```compareTo``` 의 결과와 ```equals``` 결과가 일관되지 않은 경우이다. ```new BigDecimal("1.0")``` 과 ```new BigDecimal("1.00")``` 이 두가지 객체를 ```HashSet```에 추가한 경우, 이 두 객체는 서로 다른 객체로 저장이 된다. 왜냐하면 정의된 ```equals``` 함수에서 이 두객체를 다르다고 판단하기 때문이다. 하지만 이 두 객체를 ```TreeSet``` 에 추가한다면, ```TreeSet``` 객체는 하나의 ```BigDecimal``` 만 포함하게 된다. 왜냐하면 ```compareTo``` 함수에서 이 두객체를 같다고 판단하기 때문이다.

## < , > 연산자로 비교하기 보다는 Type.compare 함수를 사용하라.

이 책의 이전버전에서는 <, > 연산자와 ``` Double.compare```, ```Float.compare``` 함수를 사용해서 비교하는것을 권장했지만 Java 7 에서는 모든 boxed primitive 클래스에 ```compare``` 함수를 제공하고 있다. 따라서 모든 경우에 ```Type.compare```함수를 사용해야 한다.

## static comparator construction 함수를 사용해서 가독성과 성능을 개선하라.

Java 8에서는 여러 ```comparator construction``` 함수들을 제공하고 있다. 아래와 같이 정의하여, comparator함수를 재사용 할 수 있다.

``` java 
public class PhoneNumber {
    int areaCode;
    int prefix;
    int lineNum;
    private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);

    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this,pn);
    }
}
```

## compare 함수에서 값을 비교할때,  minus 연산을 피하라.

``` java
static Comparator<Object> hashCodeOrder = new Comparator<>() { 
  public int compare(Object o1, Object o2) { 
    return o1.hashCode() - o2.hashCode(); 
  } 
};
```

이 경우 값을 빼는 과정에서 integer overflow가 발생할 수 있고 성능이 느릴수 있다. 이 보다는 ```compare``` 함수를 사용하는 것이 좋다.

``` java
static Comparator<Object> hashCodeOrder = new Comparator<>() { 
  public int compare(Object o1, Object o2) { 
    return Integer.compare(o1.hashCode(), o2.hashCode()); 
  } 
}; 
```

또는

``` java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode()) ;
```

## 정리

정렬이 필요한 value class에는 ```Comparable``` 인터페이스를 구현해야한다. ```compareTo``` 함수를 구현할때 필드를 비교하는 과정에서 <, > 연산자를 피하고 boxed primitive 클래스에 정의 되어있는 static ```compare``` 함수를 사용하거나 ```Comparator``` 인터페이스에 있는 construction method를 사용하라.