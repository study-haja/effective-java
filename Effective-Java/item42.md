# ITEM 42 : 익명 클래스 보다 Lambda를 선호하라.

``` java
Collections.sort(words, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(),s2.length())
    }
});
```

위와 같이 anonymous class를  사용해서 sort함수를 호출할 수 있다.

하지만 코드의 길이가 길어진다는 단점이 있다. Java8부터는 하나의 추상 메소드를 가진 인터페이스를 특별하게 취급하기 시작했다. 이러한 인터페이스를 ```functional interface``` 라고 부른다. 언어적 차원에서 이러한 인터페이스를 **lamda expression** 으로 부를 수 있도록 허락하게 되었다. 람다는 anonymous class와 유사하지만 더 간결하다.

``` java
Collections.sort(words,(s1,s2)->Integer.compare(s1.length(),s2.length()));
```

위 코드를 보면 comparator 함수의 인자(String s1, String s2)와 return 타입이 명시되지 않았다는 것을 알수 있다. 컴파일러는 이러한 타입을 ```type inference``` 라는 프로세스를 사용해서 추론한다.

이러한 타입을 선언하는 것이 가독성에 좋지 않다면 생략하는 것이 좋다. 하지만 컴파일러가 이러한 타입을 추론하지 못하는 경우에는 선언해야 한다.

이러한 ```type inference``` 를 가능하게 해준 요인중 하나가 바로 generic이다. 만약 위의 words가 ```List<String>``` 타입이 아니라면 불가능했을 일이다.

위의 코드는 ```comparator construction method``` 를 사용하면 훨씬 더 간결하게 표현할 수 있다.

```java
Collections.sort(words, Comparator.comparingInt(String::length));
```

사실 여기서 List 인터페이스의 sort 함수를 사용해서, 한단계 더 간단하게 표현할 수 있다.

```java
words.sort(Comparator.comparingInt(String::length))
```

람다는 이름과 문서화가 부족하다. 따라서 만약 계산이 self-explanatory 하지 못하거나 몇 줄이상 되는 경우 lamda에 넣지 않는것이 좋다. 람다는 한줄이 이상적이다. 그리고 3줄이 최대 라인수이다. 만약 이 규칙을 어기면 가독성이 엄청 안좋아지게 된다.

## 익명클래스 vs Lambda

람다가 도입되면서 익명클래스는 자주 사용되지 않는것은 사실이다. 하지만 여전히 익명 클래스만이 할 수 있는 일이 있다. 

1. 람다는 여전히 functional interface에 제한된다. 만약 추상클래스의 인스턴스를 생성하고 싶다면, 여전히 익명클래스를 사용할 수밖에 없다. 또한 한개 이상의 추상 메소드를 가지는 interface 또한 익명클래스로만 생성가능하다. 
2. lamda는 자기자신의 reference를 가질수가 없고, enclosing instance에 대한 reference만 가질 수 있다. 익명 클래스에서는 this 키워드를 사용해서 자기자신 reference를 가질 수 있다.

Lambda는 익명클래스와 같이 전체 구현체에 대해서 신뢰성있게 직렬화,역직렬화를 할수 없다는 특징이 있다. 그러므로 lamda는 드물게 직렬화 되어야 한다. 만약 직렬화하고자 하는 function 객체가 있다면, private static nested 클래스의 인스턴스를 사용하는 것이 좋다.

## 요약

functional interface를 사용하지 않는 경우에만, 익명 클래스를 사용해야한다. 나머지 경우에 대해서는 lamda를 사용하는 것이 좋다. 또한 lamda는 함수형 프로그래밍 테크닉을 가능하게 하는 작은 functional object를 생성하는 것을 쉽게 해준다.
