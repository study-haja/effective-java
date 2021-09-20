# Item 78 : Shared Mutable Data에 대한 접근을 동기화하라.

## Synchronization이 보장하는것

1. 함수에서 consistent 상태의 객체읽기
2. 스레드들이 서로가 변경한 사항들을 읽기

## 동기화 방법

### 1. synchronized 사용

``` java
public class StopThread {
  private static boolean stopRequested;
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> { 
      int i = 0;
    	while (!stopRequested) i++;
    }); 
    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true; 
  }
}
```

위 코드는 정상동작 하지 않는다. 왜냐면 vm이 코드를 다음처럼 변환하기 때문이다.

``` java
// before
while (!stopRequested) i++;
```



``` java
// after
if (!stopRequested) 
  while (true) i++;
```

위 프로그램은 다음처럼 수정하면 정상동작한다.

``` java
// Properly synchronized cooperative thread termination
public class StopThread {
private static boolean stopRequested;
private static synchronized void requestStop() { stopRequested = true;}
private static synchronized boolean stopRequested() { return stopRequested;}
public static void main(String[] args) throws InterruptedException {
Thread backgroundThread = new Thread(() -> {
    int i = 0;
    while (!stopRequested())
      i++; 
    });
    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    requestStop(); 
   }
}
```

여기서 중요한것은 read/write 작업에 모두 동기화를 해야한다는 점이다. 여기서 위의 작업들은 atomic하지만 스레드들간의 커뮤니케이션을 위해서 동기화를 한것이다.

### 2. volatile 사용

mutual exclusion없이 스레드들간의 커뮤니케이션만 가능하게 하는 방법으로 ``` volatile``` 을 이용한 방법이 있다.

```volatile``` 은 가장 최근에 변경된 값을 읽도록 허용한다. 

``` java
// Cooperative thread termination with a volatile field
public class StopThread {
private static volatile boolean stopRequested;
public static void main(String[] args) throws InterruptedException {
  Thread backgroundThread = new Thread(() -> { int i = 0;
    while (!stopRequested) i++;
    }); 
    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true; 
  }
}
```

위의 예제에서는 mutual exclusion은 필요없으므로, volatile 사용은 적절하다. 반면 다음과 같은 상황에서는 ```volatile``` 을 사용하면 안된다.

``` java
// Broken - requires synchronization!
private static volatile int nextSerialNumber = 0;
public static int generateSerialNumber() { 
  return nextSerialNumber++;
}
```

이유는 ```nextSerialNumber++``` 작업이 원자적이지 않기 때문이다. 즉, 값을 읽는과정과 더하는 과정이 독립적이기 때문에, 멀티스레드 환경에서 동기화 문제가 발생한다. 이를 ```safety failure(프로그램이 잘못된 결과를 계산)``` 라 한다.

이는 두가지 방법으로 수정 가능하다.

1. synchronized 사용

  ``` java
  private static int nextSerialNumber = 0;
  public static int synchronized generateSerialNumber() { 
    return nextSerialNumber++;
  }
  ```

2. AtomicLong 사용 : 1번 보다 성능이 우수하다.

  ``` java
  // Lock-free synchronization with java.util.concurrent.atomic
  private static final AtomicLong nextSerialNum = new AtomicLong();
  public static long generateSerialNumber() {
  	return nextSerialNum.getAndIncrement(); 
  }
  ```

## 요약

- 스레드들이 서로 mutable data를 공유한다면, read/write 작업은 동기화 되어야한다.    

- 만약 thread communication만 필요하고 mutual exclusion이 필요없는 경우라면 volatile 키워드를 사용하라.
