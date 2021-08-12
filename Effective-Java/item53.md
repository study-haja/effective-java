# Item53 : 가변인수는 신중히 사용하라

가변인수 메소드는명시한 타입의 인수를 0개 이상 받을 수 있다. 가변인수 메소드를 호출하면, 가장 먼저 인수의  개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메소드에 건네준다. 다음 예를 보자.

* 간단한 가변인수 활용 예

```java
static int sum(int... args) {
	int sum = 0;
	for (int arg : args)
		sum += arg;
	return arg;
}
```

다음 예제는 args 들의 합을 도출하는 메소드이다. 런타임시 args의 배열을 만들고 해당 메소드에 인수를 전달해 처리한다. 그렇다면 다음 예를 한번 보자.

* 인수가 1개 이상이어야 하는 가변인수 메소드 - 잘못 구현한 예

```java
static int min(int... args) {
  if (args.length == 0)
    throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
  int min = args[0];
  for (int i = 1; i < args.length; i++)
    if (args[i] < min)
      min = args[i];
  return min;
}
```

위 예제는 가변인수를 통해 최솟값을 구하는 메소드이다. 이 방식에는 몇가지 문제가 있는데 다음과 같다.

1.  인수를 0개만 넣어 호출하면 컴파일 타임이 아닌 런타임에서 실패를 한다.
2. 코드가 지저분 한다.

 args 유효성 검사를 명시적으로 해야하고, min의 초깃값을 Integer.MAX_VALUE로 설정하지 않고는 for-each 문도 사용할 수 없다.

다행히 더 나은 방법이 있는데 다음 예를 보자.

* 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법

```java
static int min(int firstArg, int... remainingArgs) {
  int min = firstArg;
  for (int arg : remainingArgs) 
    if (arg < min)
      min = arg;
  return min;
}
```

위 코드처럼 매개변수를 2개 받도록 하면 된다. 이러면 위와 같은 문제를 말끔히 해결할 수 있다. 



만약 성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다. 그 이유는 가변인수 메소드는 호출될 때마다 배열을 새로 하나 할당하고 초기화하기 때문이다.

다행히, 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 패턴이 있다. 예를 들면 다음과 같다.

* 가변인수 성능 향상

```java
/**
* foo()의 메소드 호출의 95%가 인수를 3개 이하로 사용한다고 가정하면 다음과 같이 오버로딩 가능
*/
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {} 
public void foo(int a1, int a2, int a3, int... rest) {} // 5%의 호출을 담당 -> 단 5%만이 배열을 생성
```

대다수의 성능 최적화와 마찬가지로 이 기법도 보통 때는 별 이득이 없지만, 필요한 특수 상황에서는 사막의 오아시스가 되어줄 것이다.

EnumSet의 정적 팩토리 도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화 한다. EnumSet은 비트 필드(Item36)를 대체하면서 성능까지 유지해야 하므로 아주 적절하게 활용한 예라 할 수 있다.

### 정리

정리하자면 인수 개수가 일정하지 않은 메소드를 정의해야 한다면 가변인수가 반드시 필요하다.

메소드를 정의할 때 필수 매개변수는 가변인수 앞에 둔다.

마지막으로 가변인수를 사용할 때는 성능 문제까지 고려하자.

