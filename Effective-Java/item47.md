# Item47 : 반환 타입으로는 스트림보다 컬렉션이 낫다

원소 시퀀스, 즉 일련의 원소를 반환하는 메소드는 수없이 많다. 자바 7까지는 이런 메소드의 반환 타입으로 다음과 같은 타입을 사용했다.

1. Collection, Set, List
2. E[]와 같은 배열
3. Iterable 인터페이스

즉 기본인 컬렉션 인터페이스를 사용했다. for-each문에서만 쓰이거나 반환된 원소 시퀀스가 일부 Collection 메소드를 구현할 수 없을 때는 Iterable 같은 인터페이스를 사용했다. 아니면 성능이  민감한 상황에서는 배열을 썼다.

하지만 자바8에 스트림이 나오면서 다음과 같은 선택들이 더 복잡해지게 되었다.



### 스트림 문제

스트림 문제는 다음과 같다.

> 스트림은 반복을 지원하지 않는다.

따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다. API를 스트림만 반환하도록 짜놓으면 반환된 스트림을 for-each로 반복하길 원하는 개발자는 당연히 불만을 가질 것이다. 

번외로 사실 스트림 인터페이스는 Iterable 인터페이스가 정의한 추상 메소드를 포함하고 있지만, Iterable을 확장하지 않아 for-each로 스트림을 반복할 수 없다.

그렇다면 stream의 iterator 메소드에 메소드 참조를 건네면 해결 되지 않을까라고 생각할 수 있지만, 그렇게 완벽한 방법은 아니다. 다음 예를 보자

* 자바 타입 추론의 한계로 컴파일 되지 않은 코드

```java
// ProcessHandle class
static Stream<ProcessHandle> allProcesses() {
    return ProcessHandleImpl.children(0);
}

// concrete class or business logic class
static void test() {
  ...
 	
  for (ProcessHandle ph: ProcessHandle.allProcesses()::iterator) {
    // handle process
  }
}
```

위 코드는 자바 타입추론의 한계로 컴파일되지 않는다.

> Test.java:6 error: method reference not expected here
> for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
>                         ^

이 오류를 바로 잡으려면 메소드 참조를 매개변수화 된 Iterable로 적절히 형변환 해줘야 한다.

* 오류 해결 코드

```java
for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
	// handle process
}
```

이렇게 타입 캐스팅을 통해 형변환을 시켜주면 동작을 한다. ( 하지만 실제로는 ClassCastException이 발생 )

하지만 이 코드는 실전에서 사용하기에는 너무 난잡하고 직관성이 떨어진다. 다행이 어댑터 메소드를 사용하면 상황이 나아진다.

* Stream<E>를 Iterable<E>로 중개해주는 어댑터

```java
// 자바의 타입 추론이 문맥을 잘 파악하여 어댑터 메소드안에서는 따로 형변환하지 않아도 된다.
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
	return stream::iterator;
}

...
 
 for (ProcessHandle ph: iterableOf(ProcessHandle.allProcesses())) {
    // handle process
  }

...
```

이렇게 어댑터로 손쉽게 해결할 수 있다.



### Iterable 문제

API가 Iterable만 반환하면 이를 스트림 파이프라인에서 처리하려는 프로그래머는 불편할 것이다. 따라서 이 또한 어댑터를 제공해서 해결할 수 있다.

* Iterable<E>를 Stream<E>로 중개해주는 어댑터

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
	return StreamSupport.stream(iterable.spliterator(), false);
}
```

객체 시퀀스를 반환하는 메소드를 작성하는데, 이 메소드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 마음 놓고 스트림을 반환하게 해주면 된다. 



### 결론

하지만  공개 API를 작성할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 사람 모두 배려 해야한다. 즉 Collection 인터페이스를 사용해 둘다 만족하는 코드를 작성하는 것이 좋다.

Collection 인터페이스는 iterable의 하위 타입이고 stream 메소드도 제공하니 반복과 스트림을 동시에 지원한다. **따라서 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.** 예를 들어 Arrays 역시 Arrays.asList와 Stream.of 메소드로 손쉽게 반복과 스트림을 지원할 수 있다.

하지만 **단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다.** 만약 반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토해보는것이 좋다. 다음 멱집합 예제를 한번 보자.

* 입력 집합의 멱집합을 전용 컬렉션에 담아 반환한다.

```java
public class PowerSet {
 	// {a, b, c} 멱집합 : {a}. {b}, {c}, {a, b}, {a. c}, {b, c}, {a, b, c}
 	// 멱집합의 원소 개수 2^n -> 표준 컬렉션 구현체에 저장하면 절대 안됌
  // AbstractList를 이용하면 전용 컬렉션을 손쉽게 구현할 수 있음
  public static final <E> Collection<Set<E>> of(Set<E> s) {
  	List<E> src = new ArrayList<>(s);
    // 입력 집합의 원소수가 30을 넘으면 power.of 예외를 던진다. (size는 int이기 때문에 2^31 -1 로 제한)
    // 이는 Stream, Iterable이 아닌 Collection을 쓸 때 단점을 보여준다. (stream이나 iterable은 사이즈 고민 x)
		if(src.size() > 30) {
			throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최대 30개).: " + s);
		}

		return new AbstractList<Set<E>>() {
			@Override
			public int size() {
				return 1 << src.size();
			}

			@Override
			public boolean contains(Object o) {
				return o instanceof Set && src.containsAll((Set) o);
			}

			@Override
			public Set<E> get(int index) {
				Set<E> result = new HashSet<>();
				for (int i = 0; index != 0; i++, index >>=1) {
					if((index & 1) == 1) {
						result.add(src.get(i));
					}
				}
				return result;
			}
		};
  }
}
```

위의 예제처럼 AbstractCollection을 활용해서 Collection 구현체를 작성할 때는 Iterable용 메소드 외의  2개만 더 구현하면 된다. 바로 contain, size이다. 이 메소드는 손쉽게 효율적으로 구현할 수 있다.

하지만 contains와 size를 구현하는게 불가능할 경우 Collection 보다는 Stream, Iterable을 반환하는 편이 낫다. 원한다면 별도의 메소드를 두어  두 방식을 모두 제공해도 된다.



### 때로는 스트림이 나을 때도 있다.

예를 들어 입력 리스트의 부분리스트를 모두 반환하는 메소드를 작성한다고 하면, 필요한 부분 리스트를 만들어 표준 컬렉션에 담는 코드는 단 3줄이면 충분하다. 하지만 이컬렉션은 입력리스트 크기의 거듭제곱만큼 메모리를 차지한다. 기하급수적으로 늘어나는 멱집합보다는 낫지만, 역시나 좋은 방법은 아니다.

하지만 입력 리스트의 모든 부분리스트를 스트림으로 구현하기는 어렵지 않다. 다음 예를 보자.

* 입력 리스트의 모든 부분리스트를 스트림으로 반환한다.

```java
public class SubList {

    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()), 
                             prefixes(list).flatMap(SubList::suffixes));
    }

    public static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                        .mapToObj(end -> list.subList(0, end));
    }

    public static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.rangeClosed(0, list.size())
                        .mapToObj(start -> list.subList(start, list.size()));
    }
}
```

이렇게 약간의 통찰력으로 prefixes와 suffixes를 사용해 손쉽게 해결할 수 있다.

Stream.concat 메소드는 반환되는 스트림에 빈 리스트를 추가하며,  flatMap으로 모든 프리픽스의 모든 서픽스로 구성된 하나의 스트림을 만든다. 마지막으로 프리픽스들과 서픽스들의 스트림은 IntStream.range와 IntStream.rangClosed가 반환하는 연속된 정숫값들을 매핑해 만들면 쉽게 해결할 수 있다.

이뿐만 아니라 다양하게 구현할 수 있다. 다음 예제를 보면

* for-loop를 이용한 코드

```java
for (int start = 0; start < src.size(); start++) {
    for (int end = start + 1; end <= src.size(); end++) {
        System.out.println(src.subList(start, end));
    }
}
```

* stream 중첩

```java
public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0, list.size())
        .mapToObj(start -> 
                  IntStream.rangeClosed(start + 1, list.size())
                           .mapToObj(end -> list.subList(start, end)))
        .flatMap(x -> x);
}
```

이상으로 스트림을 반환하는 두 가지 구현을 알아봤는데, 모두 쓸만 하다. Stream이나 Iterable을 리턴하는 API에는 Stream -> Iterable, Iterable -> Stream으로 변환하기 위한 어댑터 메소드가 필요하다.

어댑터는 클라이언트 코드를 어수선하게 만들고 책에서는 2.3배정도 느린다고 하기 때문에 원소 시퀀스를 반환하는 메소드를 작성할 때는 Stream, Iterator를 모두  지원할 수 있게 작성하는 것이 좋다. 즉 되도록 Collection으로 하는 것이 좋다.

또한 원소의 갯수가 많다면, 멱집합의 예처럼 전용 컬렉션을 리턴하는 방법을 고민해보자. 그리고 만약 나중에 Stream 인터페이스가 iterable을 지원하도록 수정된다면, 그때는 안심하고 Stream을 반환하면 된다.

