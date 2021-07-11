# Item37: ordinal 인덱싱 대신 EnumMap을 사용하라

배열이나 리스트에서 원소를 꺼낼 때 ordinal 메소드로 인덱스를 얻는 코드가 있다. 다음 예를 보자

* Plant 코드

```java
class Plant {
    enum LifeCycle {
        ANNUAL, PERENNIAL, BIENNIAL // 한해살이, 여러해살이, 두해살이
    }

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

이 코드는 정원에 심는 식물들을 배열 하나로 관리하고, 이들의 생명주기를 나타내주고 있다. (ANNUAL, PERENNIAL, BIENNIAL)

즉 총 3개의 생애주기 집합을 만들고 정원에 있는 식물들을 한 바퀴 돌며 해당하는 집합에 넣는다. 다음 코드가 정원을 한바퀴 돌아 모든 식물을 분류한다.

* ordinal()을 배열 인덱스로 사용하여 정원의 모든 식물을 맞는 집합에 넣는 코드

```java
public void plantClassification(Plant[] graden) {
    Set<Plant>[]  plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
    for (int i = 0; i < plantsByLifeCycle.length; i++) {
        plantsByLifeCycle[i] = new HashSet<>();
    }

    for (Plant p : graden) {
        plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
    }

    for (int i = 0; i < plantsByLifeCycle.length; i++) {
        System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
    }
}
```

위 코드들에서 동작은 하지만 문제가 한가득이다. 바로

1. 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다.
2. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
3. 정확한 정숫값을 사용한다는 것을 우리가 보증해야 한다는 점이다.

즉 잘못된 값을 사용하면 잘못된 동작을 수행해 최악이면 `ArrayIndexOutOfBoundsException`이 발생 할 것이다.

따라서 해결책은 다음과 같다. 여기서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일이니 Map을 활용해서 해결할 수 있다. 또한 열거 타입을 키로 사용하도록 설계한 EnumMap을 사용하면 훨씬 성능이 좋아질 수 있다. 다음 예가 그렇다.

* EnumMap을 사용해 데이터와 열거 타입 매핑

```java
private void plantClassification(Plant[] garden) {
    Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

    for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
        plantsByLifeCycle.put(lc, new HashSet<>());
    }

    for (Plant p : garden) {
        plantsByLifeCycle.get(p.lifeCycle).add(p);
    }

    System.out.println(plantsByLifeCycle);
}
```

위 코드를 사용하면 다음과 같은 이점을 얻을 수 있다.

1. 안전하지않는 형변환은 쓰지 않아 컴파일이 깔끔하게 되고 개발자는 신경을 안써도 된다.
2. 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공할수 있다. ( 레이블을 달 필요가없다. )
3. 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다.
4. 성능도 원래 버전과 비등하다. ( EnumMap 내부에서 배열을 사용하기 때문 )

여기서 EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다.

스트림을 사용해 맵을 관리하면 코드를 더 줄일 수 있다. 다음 예를 보자

* 스트림 사용한 코드

```java
private void plantClassification(Plant[] garden) {
	 Map<Plant.LifeCycle, List<Plant>> map = Arrays.stream(garden)
                .collect(groupingBy(p -> p.lifeCycle));

   System.out.println(map);
}
```

위 에처럼 스트림을 사용해 최적화를 할 수 있다. 하지만 이 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻는 공간과 성능 이점이 사라진다는 문제가 있다. 이 문제를 해결하기 위해서 매개변수 3개짜리 Collectors.groupingBy 메소드를 사용하면된다. 다음 예가 그렇다.

* EnumMap으로 만든 스트림 코드

 ```java
 private void plantClassification(Plant[] garden) {
 	 EnumMap<Plant.LifeCycle, Set<Plant>> enumMap = Arrays.stream(garden)
                 .collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(Plant.LifeCycle.class), toSet()));
   
    System.out.println(enumMap);
 }
 ```

위 코드들은 단순해서 최적화를 안해도 되지만 Map을 빈번히 사용하는 프로그램에서는 효과적으로 최적화를 할 수 있다. 

또한 스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다. EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다. 예를 들면 정원에 한해살이와 여러해살이 식불만 살고 두해살이 식물이 없다면, EnumMap은 3개 맵을 다 만들고 Stream은 2개만 만든다.

<br>

여기까지 이해했다면 더 심화된 코드를 한번 보자. 다음 예는 ordinal을 두번 쓴 배열들의 배열 코드다.

* 배열들의 배열의 인덱스에 ordinal()을 사용한 코드

```java
public enum Phase {
	SOLID, LIQUID, GAS; //고체, 액체, 기체
  
  public enum Transition {
    MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT; // 융해, 응고, 기화, 액화, 승화, 승화
    
    // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 사용
    private static final Transition[][] TRANSITIONS = {
      { null, MELT, SUBLIME }, // 고체는 융해, 승화
      { FREEZE, null, BOIL }, // 액체는 응고, 기화
      { DEPOSIT, CONDENSE, null },  // 기체는 승화, 액화
    };
    
    public static Transition from(Phase from, Phase to) {
      return TRANSITIONS[from.ordinal()][to.ordinal()];
    }
  }
}
```

위 예제도 마찬가지로 컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없다. 

즉 Phasse나 Phase.Transition 열거 타입을 수정하면서 상전이 표 TRANSITIONS를 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 날 것이다. 또한 상전이표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어날 것이다.

이 문제를 해결하기 위해 아까 봤던 EnumMap을 사용하는 편이 훨씬 낫다. 전이 하나를 얻으려면 이전 상태와 이후 상태가 필요하니, Map 2개를 중첩하면  쉽게 해결 할 수 있다. 다음 예를 보자

* 중첩 EnumMap으로 데이터와 열거 타입 쌍으로 연결

```java
public enum Phase {
	SOLID, LIQUID, GAS;
  
  public enum Transition {
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
    
		private final Phase from;
    private final Phase to;
    
    Transition(Phase from, Phase to) {
      this.from = from;
      this.to = to;
    }
    
    private static final Map<Phase, Map<Phase, Transition>> m = 
      Stream.of(values())
                .collect(
                  groupingBy(t -> t.from,
                             () -> new EnumMap<>(Phase.class),
                             toMap(t -> t.to, 
                                   t -> t,
                                   (x, y) -> y, 
                                   () -> new EnumMap<>(Phase.class)
                                   )
                            )
                );
    
    public static Transition from(Phase from, Phase to) {
      return m.get(from).get(to);
    }
  }
}
```

위 코드에서 봤듯이 상전이 표를 초기화하는 코드는 복잡하다. 이 맵의 타입인 Map<Phase, Map<Phase, Transition>>은 "이전 상태에서 '이후상태에서 전이로의 맵' 에 대응시키는 맵" 이라는 뜻이다. 이러한 맵의 맵을 초기화하기 위해 Collector 2개를 차례로 사용했다.

첫번째 수집기인 groupingBy에서는 전이를 이전 상태를 기준으로 묶고, 두번째 수집기인 toMap 에서는 이후 상태를 전이에 대응시키는 EnumMap을 생성한다. 두번째 수집기의 병합 함수인 (x, y) -> y는 선언만 하고 실제로는 쓰이지 않는데, 이는 단지 EnumMap을 얻으려면 Map 팩터리가 필요하고 Collector들은 점충적 팩터리를 제공하기 있기 때문이다.

이렇게 복잡하지만 새로운 상태를 추가하게 되는 요구사항이 들어오면 쉽게 수정할 수 있다. 예를 들어 플라즈마라는 새로운 상태가 들어오게 된다면 코드는 다음과 같다.

* EnumMap버전에 새로운 상태 추가

```java
public enum Phase {
	SOLID, LIQUID, GAS, PLASMA; // 플라즈마 상태
  
  public enum Transition {
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
    IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS); // 이온화, 탈이온화
    
    ...
    
}
```

만약 위의 배열 코드를 사용했더라면 Transition에 2개를 추가해야하고, 배열원소를 16개짜리로 교체해야하는등 많은 수작업이 필요하겠지만, EnumMap 버전의 코드로 추가하게 된다면 저렇게 단순하게 사용되어진다.
