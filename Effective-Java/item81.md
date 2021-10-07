# Item81 : wait와 notify보다는 동시성 유틸리티를 애용하라

 동시성 컬렉션은 List, Queue, Map 과 같은 표준 컬렉션 인터페이스에 동시성을 고려하여 구현한 고성능 컬렉션이다. 

 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행하는데 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.

따라서 동시성 컬렉션의 동시성을 무력화하지 못하기 때문에 여러 메소드를 원자적으로 묶어 호출하는 것도 못한다. 그래서 여러 동작을 하나의 원자적 동작으로 묶는 상태 의존적 메소드가 추가 되었다.

예를 들면 다음과 같다.

* putIfAbsent을 사용한 ntern 메소드

```java
private static final ConcurrentMap<String, String> map =
        new ConcurrentHashMap<>();

public static String intern(String s) {
    String result = map.get(s); // get이 putIfAbsent보다 훨신 빠름 (검색 기능에 최적화)
    if (result == null) {
      	// 기존값이 있으면 그 값을 반환하고 없는 경우에는 null을 반환
        result = map.putIfAbsent(s, s);
        if (result == null) {
            result = s;
        }
    }
    return result;
}
```

위에 예제에서 putIfAbsent는 Map의 디폴트 메소드인데 인자로 넘겨진 key가 없을 때 value를 추가한다. 

동기화한 컬렉션보다 동시성 컬렉션을 사용해야 한다. 예를 들어 Collections의 synchronizedMap 보다는 ConcurrentHashMap을 사용하는 것이 훨씬 좋다. 동기화된 맵을 동시성 맵으로 교체하는 것 하나만으로 성능이 개선 될 수 있다.



### 동기화 장치

쓰레드가 다른 쓰레드를 기다릴 수 있게 하여 서로의 작업을 조율할 수 있도록 해준다. 대표적인 동기화 장치로는 CountDownLatch와 Semaphore가 있으며 CyclicBarrier와 Exchanger도 있다. 가장 강력한 동기화 장치로는 Phaser가 있다.

CountDownLatch는 하나 이상의 쓰레드가 또 다른 하나 이상의 쓰레드 작업이 끝날 때까지 기다리게 한다. 생성자는 int형으로 받아 latch의 countdown() 메소드를 몇 번 호출해야 대기 중인 쓰레드를 깨우는지 결정한다.

예를 들면 다음과 같다.

* CountDownLatch 예제

```java
public class CountDownLatchExam {
    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
      // 작업자 쓰레드들이 준비가 완료됐음을 타이머 쓰레드에 통지할 때 사용
      CountDownLatch ready = new CountDownLatch(concurrency);
      CountDownLatch start = new CountDownLatch(1);
      CountDownLatch done = new CountDownLatch(concurrency);

      for (int i = 0; i < concurrency; i++) {
      	executor.execute(() -> {
					// 타이머에게 준비 완료를 알림, 잠들어 있던 모든 작업자 쓰레드가 깨어나서 start.countDown() 호출
					ready.countDown();
					try {
						start.await(); // 모든 작업자 스레드가 준비될 때까지 await
						action.run();
					} catch (InterruptedException e) {
						Thread.currentThread().interrupt();
					} finally {
						done.countDown(); // 타이머에게 작업을 마쳤음을 알림
					}
				});
			}

			ready.await();  // 모든 작업자가 준비될 때까지 await
			long startNanos = System.nanoTime();
			start.countDown();  // 작업자들을 깨움 -> 이후 action.run() 호출
			done.await();   // 모든 작업자가 일을 끝마치기를 기다림
			return System.nanoTime() - startNanos;
		}
}
```

위 코드에서 **executor는 concurrency 매개변수로 지정한 값만큼의 쓰레드를 생성할 수 있어야 한다.** 그렇지 않으면 메소드 수행이 끝내지 않는데 이를 쓰레드 기아 교착 상태라고 한다. 또, 시간을 잴 때는 시스템 시간과 무관한 System.nanoTime을 사용하는 것이 더 정확하다.



### wait와 notify 메서드

새로운 코드라면 wait, notify가 아닌 언제나 동시성 유틸리티를 사용해야 한다. 하지만 사용할 수밖에 없는 상황이라면 반드시 동기화 영역 안에서 사용해야 한다.

* wait() 메소드를 사용하는 표준 방식

```java
synchronized (obj) {
    while (조건이 충족되지 않았다) {
        obj.wait(); // 락을 놓고 대기 상태로 돌아간다.
    }

    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

**wait() 메소드를 사용할 때는 반드시 대기 반복문 관용구를 사용해 한다. 반복문 밖에서는 절대로 호출해서는 안된다.** 반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다. 대기 전에 조건을 검사하여 조건이 충족되었으면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치이다. 만약 조건이 이미 충족되었는데 쓰레드가 notify 또는 notifyAll 메소드로 먼저 호출한 후 대기 상태로 빠지면, 그 쓰레드를 다시 깨우지 못할 수 있다.

한편, 대기 후에 조건을 검사하여 조건을 충족하지 않았을 때 다시 대기하게 하는 것은 잘못된 값을 계산하는 **안전 실패**를 막기 위한 조치다. 그런데 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 있다.

- `notify`를 호출하여 대기 중인 스레드가 깨어나는 사이에 다른 스레드가 락을 거는 경우
- 조건이 만족되지 않았지만 실수 혹은 악의적으로 `notify`를 호출하는 경우
- 대기 중인 스레드 중 일부만 조건을 충족해도 `notifyAll`로 모든 스레드를 깨우는 경우
- 대기 중인 스레드가 드물게 `notify` 없이 깨어나는 경우. 허위 각성(spurious wakeup)이라고 한다.

일반적으로 `notify`보다는 `notifyAll`을 사용하는 것이 안전하며, `wait`는 항상 `while`문 내부에서 호출하도록 해야 한다.