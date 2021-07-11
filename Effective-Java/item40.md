# ITEM 40 : 일관적으로 OVERRIDE 어노테이션을 사용하라.

## @Override를 사용하지 않았을때, 발생할 수 있는 문제 상황

``` java
package item40;

import java.util.HashSet;
import java.util.Set;

public class Bigram {

    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++) s.add(new Bigram(ch, ch));
        System.out.println(s.size()); // 260
    }
}
```

위 코드에서 코드 작성자는 ```equals``` 함수를 오버라이딩 하려고 의도한 것으로 보인다. 하지만 실제로는 파라미터 타입이 ```Bigram``` 타입이므로 오버로딩이 되어 버렸다. (원래는 ```Object``` 타입의 파라미터로 선언해야 했음)이러한 실수를 방지 하기 위해서  ```equals``` 함수에 ```@Override``` 어노테이션을 추가할 수 있다. 이렇게 할 경우, 해당 함수가 오버라이딩 되지 않았다는 컴파일 에러가 발생하게 된다.

> Bigram.java:10: method does not override or implement a method from a supertype
>
> @Override public boolean equals(Bigram b) { ^

그리고 아래와 같이 수정할 수 있다.

``` java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Bigram))
        return false;
    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;
}
```

## @Override 사용해야 하는 이유

1. 위의 예에서처럼, 정확히 오버라이딩 했는지 확인가능하다.
2. 대부분의 IDE에서는 오버라이딩된 함수에 대해서 @Override가 사용되었는지 체크를 한다. 만약 오버라이딩된 함수가 사용되지 않았으면, Wanring을 발생시킨다. IDE에서 발생하는 Warning을 제거하기 위해서, @Override를 사용할 필요가 있다.

## 요약

슈퍼타입 선언에 있는 함수를 오버라이딩하는 모든 함수에 대해서, @Override를 사용하면, 컴파일러가 적절하게 오버라이딩이 되었는지 알려준다. 하지만 한가지 예외가 있다. 구체 클래스(concrete class)에서 추상 클래스를 상속한 경우, 오버라이딩 할 함수에 @Override 를 반드시 붙일 필요는 없다. 왜냐하면 컴파일러가 강제로 구현하도록 강제해주기 때문이다. (그렇게 하는것이 해롭지는 않다. - 이 부분 좀 이상함.)
