# Item25 : 톱 레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 에러를 내지 않는다. 하지만 이러한 행위는 정말 위험하고 아무런 득이 없는 행위이다. 그 이유는 한 클래스를 여러 가지로 정의할 수 있으며, 그중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하냐에 따라 달라지기 때문이다.

다음 예를 보자.

* Main.java

```java
class Main {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
}
```

* Utensil.java

```java
class Utensil {
  static final String NAME = "pan";
}

class Dessert {
  static final String NAME = "cake";
}
```

* Dessert.java

```java
class Utensil {
  static final String NAME = "pot";
}

class Dessert {
  static final String NAME = "pie";
}
```

여기서 `javac Main.java Dessert.java` 컴파일을 한다면 오류가 날 것이다. 그 이유는 Main에서 Utensil 클래스를 먼저 호출하기 때문에 Utensil 자바 파일 안에 Utensil하고 Dessert와 Dessert 자바 파일 안에 있는 Utensil과 Dessert 클래스를 중복 정의했다고 알려줄 것이다. ( 명령어 Dessert.java를 실행한 순간 중복 정의했다고 컴파일러는 알려 줄 것이다. )

하편 `javac Main.java`나 `javac Main.java Utensil.java` 명령어로 컴파일하면 아무 오류 없이 pencake을 출력할 것이다. 그러나 `javac Dessert.java Main.java` 명령어를 치면 potpie를 출력할 것이다.

이처럼 컴파일러에 어느  소스파일을 먼저 건네느냐에 따라 동작이 달라지므로 반드시 바로 잡아야 할 문제이다.

<br>

## 톱레벨 클래스들을 서로 다른 소스로 파일을 분리하자

위와 같은 문제의 해결법은 간단하다. 바로 톱레벨 클래스들을 서로 다른 소스로 파일을 분리하면 된다. 굳이 여러 톱레벨 클래스를 한파일에  담고 싶다면 정적 멤버 클래스를 사용하는 것도 좋다.

다음 예를 보자

* 정적 멤버 클래스로 바꾼 여러개의 톱레벨 클래스

```java
public class Test {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
  
  private static class Utensil {
    static final String NAME = "pan";
	}

	private static class Dessert {
	  static final String NAME = "cake";
	}
}
```

이런식으로 정적 멤버 클래스로 만들어 읽기 수월하고 private이기 때문에 접근 범위도 최소로 관리할 수 있어 좋다.

