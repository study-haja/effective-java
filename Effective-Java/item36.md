# Item 36 : bit 필드 대신 EnumSet을 사용하라.

다음은 비트 필드는 enum 타입에  집합연산을 사용할때 주로 사용한다.

``` java
public class Text{
  public static final int STYLE_BOLD = 1 << 0; // 1
  public static final int STYLE_ITALIC = 1 << 1; // 2
  public static final int STYLE_UNDERLINE = 1 << 2; // 4
  public static final int STYLE_STRIKETHROUGH = 1 << 3; // 5

  public void applyStyle(int styles){...}
}
```

 ```text.applySyles(STYLE_BOLD | STYLE_ITALIC)``` 처럼 OR연산을 사용할 수 있고 AND 연산또한 사용할 수 있다.

위 방법의 문제점은 아래와 같다.

1. 비트 필드를 해석하는 것이 어렵다.
2. 비트필드가 나타내는 각각의 요소들을 루프로 순회하기 어렵다.
3. 필요한 비트수를 예상해야 한다. 그리고 32 or 64 비트 자리수를 넘어서는 안된다.

## EnumSet

위 문제를 해결하기 위해서 EnumSet을 사용할 수 있다. 내부적으로 EnumSet은 비트 벡터로 구현이 되어있다. 그리고 64개 이하의 enum type을 저장할 수 있다. 전체 EnumSet은 하나의 long 값으로 표현될수 있으므로, 성능이 비트 필드만큼 우수하다.

```removeAll```, ```retainAll``` 과 같은 bulk 작업들은 bitwise 계산을 사용해서 구현되어 있다. 따라서 사용자가 이러한 기능을 직접 구현할 필요가 없다.

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

## 요약

enum type에 집합연산을 사용할 때는 비트필드 대신, EnumSet을 사용하라. EnumSet은 타입안전성과 비트 필드의 성능 두가지 장점 모두 가지고 있다.