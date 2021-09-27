# Item79 : 과도한 동기화는 피하라

동기화를 과도하게 사용할 경우 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳기도 한다.

즉 **응답 불가와 안전 실패를 피하려면 동기화 메소드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.** 예를 들면 다음과 같다.

1. 동기화된 영역 안에서는 재정의할 수 있는 메소드는 호출하면 안된다.
2. 클라리언트가 넘겨준 함수 객체를 호출해서도 안된다.

위와 같은 것들을 동기화된 영역에서는 바깥 세상에서 온 외계인 메소드라 부른다. 외계인 메소드가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다. 구체적인 예를 한번 보자.

* 잘못된 코드, 동기화 블록 안에서 외계인 메소드를 호출한 경우

```java
public class ObservableSet<E> extends ForwardingSet<E> {
	public ObservableSet(Set<E> set) { super(set); }
  
  private final List<SetObserver<E>> observers = new ArrayList<>();
  
  public void addObserver(SetObserver<E> observer) {
    synchronized(observers) {
      observers.add(observer); // 구독 신청
    }
  }
  
  public void removeObserver(SetObserver<E> observer) {
    synchronized(observers) {
      observers.remove(observer); // 구독 해지
    }
  }
  
  private void notifyElementAdded(E element) {
    synchronized(observers) {
      for (SetObserver<E> observer : observers) {
        observer.added(this, element);
      }
    }
  }
  
  @Override
  public boolean add(E element) {
    boolean added = super.add(element);
    if (added)
      	notifyElementAdded(element); // 집합에 원소가 추가되면 알림을 받음
    return added;
  }
  
  @Override
  public boolean addAll(Collection<? extends E> c) {
    boolean result = false;
    for (E element : c)
      result |= add(element);  // 비트 연산자 or
    return result;
  }
}

@FunctionalInterface 
public interface SetObserver<E> {
  // ObservableSet에 원소가 더해지면 호출된다.
  void added(ObservableSet<E> set, E element);
}
```

위  SetObserver 인터페이스는 BiConsumer<ObservableSet<E>, E>와 같다. 하지만 커스텀 함수형으로 재정의한 이유는 이름이 더 직관적이고 다중콜백을 지원하도록 확장할 수 있어서 구현 했다. 

그렇다면 위 코드로 아래 예제를 돌려보자.

* 0부터 99까지를 출력하는 예제

```java
public static void main(String[] args) {
  ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
  
  set.addObserver((s, e) -> System.out.println(e)); // SetObserver 함수형 인터페이스 정의
  
  for (int i = 0; i < 100; i++)
    set.add(i); // 0 ~ 99 까지 출력
}
```

위 예제는 잘 동작 할 것이다. 그렇다면 다음과 같은 예제는 어떨까?

* 23이면 관찰자를 제거하는 코드

```java
public static void main(String[] args) {
  ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
  
	set.addObserver(new SetObserver<>() {
  	@Override
	  public void added(ObservableSet<Integer> set, Integer element) {
  		System.out.println(element);
	    if (element == 23)
    		set.removeObserver(this);
		}
	});
  
  for (int i = 0; i < 100; i++)
    set.add(i); // 0 ~ 99 까지 출력
}
```

이 프로그램은 0~23까지 출력한 후 관찰자 자신을 구독해지한 다음 조용히 제거할 거라 생각했지만, 실제로 실행해보면 그렇게 진행되지 않는다. 이 프로그램은 23까지 출력한 후  ConcurrentModificationException을 던진다. 그 이유는 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중에 SetObserver를 제거하려고 했기 때문이다.

> add() -> notifyElementAdded()					 -> 					added()		-> 		removeObserver()	 ->		 remove()
>
> ​										Lock을 건체 SetObserver 리스트 순회중													 	SetObserver 제거
>
> ​																								락이 걸린 순회 도중 제거하는건 허용되지 않음

이번에는 구독해지 할때 직접 호출하지 않고 실행자 서비스를 사용해 다른 쓰레드한테 부탁하는 예제를 한번 보자.

*  구독해지할 때 ExecutorService를 사용한 코드

```java
set.addObserver(new SetObserver<>() {
  public void added(ObservableSet<Integer> s, Integer e) {
    System.out.println(e);
    if (e == 23) {
      ExecutorService exec = Executor.newSingleThreadExecutor();
      try {
        exec.submit(() -> s.removeObserver(this)).get();
      } catch (ExecutionException | InterruptedException ex) {
        	throw new AssertionError(ex);
      } finally {
        exec.shutdown();
      }
    }
  }
});
```

이 프로그램은 실행하면 교착상태가 일어난다. 백그라운드 스레드가 s.removeObserver를 호출하라면 관찰자를 잠그려 시도하지만 락을 얻을 수 없다. 메인 쓰레드가 이미 락을 쥐고 있기 때문이다. 그와 동시에 메인 쓰레드는 백그라운드 쓰레드가 관찰자를 제거하기만을 기다리는 중이다. 이렇게 서로 기다리고 있기 때문에 교착상태가 일어나게 된다.

이렇게 동기화된 영역 안에서 외계인 메소드를 호출하여 교착상태에 빠지는 사례는 자주 있다.

그렇다면 똑같은 상황이지만 불변식이 임시로 깨진 경우라면 어떨까? 자바 언어의 락은 재진입을 허용하므로 교착상태에 빠지지는 않는다. 예외를 발생시킨 첫 번째 예에서라면 외계인 메소드를 호출하는 쓰레드는 이미 락을 쥐고 있으므로 다음번 락 획득도 성공한다. 하지만 재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록  해주지만, 응답 불가가 될 상황을 안전 실패로 변모시킬 수도 있기 때문에 참혹한 결과가 빚어질 수도 있다.

다행이 이러 문제는 대부분 어렵지 않게 해결할 수 있다. 외계인 메소드 호출을 동기화 블록 바깥으로 옮기면 된다. 이를 열린 호출(Open Call)이라 한다. 다음 예제를 보자.

* 외계인 메소드를 동기화 블록 바깥으로 옮긴 코드

```java
private void notifyElementAdded(E element) {
  List<SetObserver<E>> snapshot = null;
  synchronized(observers) {
    snapshot = new ArrayList<>(observers); // 관찰자 리스트 복사해 교착 상태 문제 해결
  }
  for (SetObserver<E> observer : snapshot) {
    observer.added(this, element);
  }
}
```

하지만 위와 같은 방법보다 더 나은 방법이 있다. **자바의 동시성 컬렉션 라이브러리인 CopyWriteArrayList를 사용하면 된다.** 다음 예를 보자.

* CopyOnWriteArrayList를 사용해 구현한 쓰레드 안전하고 관찰 가능한 집합

```java
// ArrayList를 수현한 클래스로, 내부를 변경하는 작업은 항상 깨끗하게 복사본을 만들어 수행
// 내부 배열은 수정되지 않으니 순회할 때 락이 필요 없음
// 수정할 일이 드믈고, 순회만 빈번히 일어난다면 사용하기 좋은 라이브러리 
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
  observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
  return observers.remove(observer);
}

private void notifyElementAdded(E element) {
  for (SetObserver<E> observer : observers)
    observer.added(this, element); // remove해도 새로 생긴 리스트라 Exception 발생 x
}
```

이렇게 구현함으로써 실패 방지 효과 외에도 동시성 효율을 크게 개선해준다.

그렇다면 성능도 한번 보자. 자바의 동기화 비용은 빠르게 낮아져 왔지만, 과도한 동기화를 피하는 일은 오히려 과거 어느 때보다 중요하다. 동기화가 초래하는 비용은 락을 얻는데 드는 CPU 시간이 아니라, 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다. 

따라서 가변 클래스를 작성하려거든 다음 두가지 선택 중 하나를 따르자.

1. 동기화를 전혀 하지말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하자.
2. 동기화를 내부에서 수행해 쓰레드 안전한 클래스로 만들자.

단 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 두 번째 방법을 선택해야 한다. 자바에서는 이미 첫 번째 방식을 취했고, java.utill.concurrent는 두 번째 방식을 취했다. StringBuilder도 StringBuffer의 동기화 문제로 위와 같은 방법을 적용해서 나왔다. 그리고 java.util.Random 함수도 마찬가지로 java.util.concurrent.ThreadLocalRandom으로 나오게 되었다.



### 정리

정리하자면 다음과 같다. 기본 규칙은 동기화  영역에서는 가능한 한 일을 적게 하는 것이다. 락을 얻고, 공유 데이터를 검사하고, 필요하면 수정하고, 락을 놓는다. 일반화 해 이야기하면, 동기화 영역 안에서의 작업은 최소한으로 줄이자. 가변 클래스를 설계할 때는 스스로 동기화할지 고민하며, 멀티 코어인 지금 세상은 과도한 동기화를 피하는 것이 어느 때보다 중요하다. 합당한 이유가 있을 때만 내부에서 동기화하고, 동기화했는지 여부를 문서에 명확히 밝히자.