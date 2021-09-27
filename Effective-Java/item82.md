# Item 82 : Thread Safety를 문서화하라.

## thread safety level를 명시할것

- Immutable 

  - 인스턴스가 변하지 않는 경우, 예를들어 ```String```,```Long```,```BigInteger```
  - 아무런 동기화과정이 필요없음.

- Unconditionally thread-safe 

  - 인스턴스는 mutable이지만 외부적 동기화 없이 병렬로 사용 될수 있는 경우

  - 예를들어 ```AtomicLong```, ```ConcurrentHashMap```

  - synchronized method vs private lock

    - Synchronized method

      ``` java
      public synchronized foo() {
      	...  
      }
      ```

      다른 사용자가 고의로 또는 실수로 ```foo``` 함수를 실행해서, 정작 필요한 함수가 ```foo```를 실행하는 기회를 잃어 버릴수 있다.

    - private lock

      ``` java
      private final Object lock = new Object();
      public void foo() { 
        synchronized(lock) {
      		... 
        }
      }
      ```

      일단 lock 객체를 클래스 외부에서 접근하지 못해서 위 문제가 발생하지 않는다.

- Conditionally thread-safe

  - 몇몇 함수에만 외부적 동기화가 필요한 경우
  - 예를들어 ```Collections.synchronized``` wrapper에 의해 리턴되는 컬렉션
  - 정확히 어떤 함수가 외부적 동기화를 필요로 하는지 명시해야 됨.

- Not thread-safe

  - 인스턴스가 mutable이며, 모든 함수에서 외부적 동기화가 필요한 경우

- Thread-hostile

  - 함수에 외부적 동기화를 처리하더라도 unsafe한 경우
  - 보통 동기화없이 static data를 수정해서 발생

## 요약

- 모든 클래스는 분명하게 thread safety에 대해서 문서화할것
- Conditionally thread-safe 클래스들은 어느 함수를 호출이 동기화가 필요하며 어떤 락을 획득해야하는지 명시해야함
- Unconditionally thread-safe 클래스들을 작성할때는 synchronized 함수대신 private lock 객체를 사용하는것을 고려하라