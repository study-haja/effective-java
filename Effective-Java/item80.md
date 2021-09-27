# Item 80 : Executor, Task, Stream 을 Thread 보다 선호하라.

- Executor Framework

  - ```java.util.concurrent``` 패키지에 존재	

  - 기존의 ```Thread``` 클래스보다 더 편리한 사용성 제공

  - 객체생성

    ``` ExecutorService exec = Executors.newSingleThreadExecutor();```

  - 실행

    ``` exec.execute(runnable); ```

  - 종료

    ``` exec.shutdown();```

  - Thread pool 작업의 모든면을 설정하고 싶다면, ```Executors.ThreadPoolExecutor``` 사용

  - 작은 프로그램(local에서 동작하는 프로그램)에 대해서는 ```Executors.newCachedThreadPool``` 사용

  - 큰 프로그램(production용)에 대해서는 ```Executors.newFixedThreadPool``` 사용

  - 작업의 단위를 task로 추상화하며, 두가지 종류가 있다.

    - Runnable
    - Callable : Runnable과 비슷하지만 값을 리턴하고 임의의 예외를 던질수 있음.

  - Fork-join task를 지원

    - Fork-join task는 여러 subtask로 나누어지고 스레드들은 task를 처리할 뿐만 아니라 다른 스레드들로부터 task를 뺏어올 수 있다. 결과적으로 higher CPU utilization, higher throughput, lower latency를 제공함.

  - Parallel streams

    - Fork-join pools를 기반으로 작성된 더 쉬운 api

