 

# Item9 : try-finally 보다는 try-with-resources를 사용하라

자바에서는 resource를 사용하고 close를 사용해 닫아야 하는 메소드들이 많다. 대표적인 예로 InputStream, OutputStream, dbConnection을 들수 있다.

만약 이 자원들을 닫지 않으면 시스템은 성능 문제를 야기 시킬 수 있다. 그렇기 때문에 전통적으로 자바에서는 finalizer를 활용해 자원을 닫아주는 것을 보장 했었다. 그렇지만 finalizer는 item8에서 봤듯이 언제 JVM에서 호출을 해줄지 모르기 때문에 별로 믿을만하지 못하다. 

또한 다음과 같은 단점도 보인다. 예를 한번 보자.

* try-finally를 이용해 자원을 회수 하는 코드

  ```java
  static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
      return br.readLine();
    } finally {
      br.close();
    }
  }
  ```

다음 코드는 전혀 문제 될 게 없어 보인다. 코드가 깔끔하게 보이고 finally 구문에서 BufferedReader의 자원이 잘 닫히는 것을 확인할 수 있다. 하지만 자원이 하나 더 추가되면 어떻게 될지 다음 예제를 통해 살펴 보자.

* try-finally를 이용해 2개의 자원을 회수하는 코드

  ```java
  static void copy(String src,  String dst) throws IOException {
    InputStream in = new InputStream(src);
    try {
      OutputStream out = new FileOutputStream(dst);
      try {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while((n = in.read(buf)) >= 0) 
          out.write(buf, 0, n);
      } finally {
        out.close();
      }
    } finally {
      in.close();
    }
  }
  ```

다음 코드를 봤을때 벌써부터 복잡해 보인다. InputStream을 선언해주어서 try 블록으로 감싸 구현하고, 또 그안에 OutputStream을 선언해서 또 try 블록으로 감싸서 구현하는 구조로 되어 있어 프로그래머가 해당 코드를 한눈에 파악하기 어렵고 유지보수 하기가 힘들다.

그리고 다음과 같은 문제가 또 발생할 수 있는데 try 블록과 finally 블록 모두 예외가 발생하게 된다면 둘 중 어느 한가지의 예외가 씹혀버릴 수가 있다. 예를 들어 기기의 물리적인 문제로 firstLineOfFile 메소드 안의 readLine 메소드가 예외를 던지고, 같은 이유로 close 메소드도 실패를 하게 되었을 때, 이런 상황에서 close 메소드의 대한 예외가 readLine의 예외를 집어 삼켜 버린다. `즉 스택 추적에서 첫번째 예외를 완전히 삼켜버렸기 때문에 첫 번째 예외에 관한 정보는 남지 않게 되어, 실제 디버깅을 어렵게 만든다.` 물론 두 번째 예외 대신 첫 번째 예외를 기록하도록 코드를 수정할 수 있지만, 코드가 너무 지저분해지고 실제로 그렇게 까지 안한다고 한다.

따라서 문제점을 정리하자면 다음과 같다.

> 1. Complexity Code - 코드 복잡성
> 2. Difficulty Stack Trace - 스택 추적의 난관

이러한 문제들을 자바 7에서 나온 try-with-resources를 통해 모두 해결되었다. 다음 예를 보자

* try-with-resources를 통해 한개의 자원을 회수하는 코드

  ```java
  static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
      return br.readLine();
    }
  }
  ```

* try-with-resources를 통해 복수개의 자원을 회수하는 코드

  ```java
  static void copy(String src,  String dst) throws IOException {\
    try (InputStream in = new InputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while((n = in.read(buf)) >= 0) 
          out.write(buf, 0, n);
    }
  }
  ```

해당 코드를 통해 확실히 코드가 짧아진 것을 볼 수 있다. 또한 문제를 진단하기도 훨씬 좋다. readLine과 close 호출 양쪽에서 예외가 발생하면, close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다. 즉 프로그래머에게 보여줄 예외 하나만 보여주고 나머지는 Stack Trace에 Suppressed라는 꼬리표를 달고 예외를 출력한다. 또한 Throwable에 추가된 `getSuppressed`메소드를 이용해 프로그램 코드에서 가져 올 수 있다.

 그리고 `try-catch-resource` 구조에서 catch 절을 사용할수 있다. 이 catch 절을 통해 try 문을  더 중첩하지 않고도 다수의 예외를 처리 할 수 있다. 다음 예를 보자.

* catch절을 사용해 예외에 대한 후 처리 하는 코드

   ```java
   static String firstLineOfFile(String path, String defaultVal) {
     try (BufferedReader br = new BufferedReader(new FileReader(path))) {
       return br.readLine();
     } catch (IOException e) {
     	return defaultVal;
     }
   }
   ```

readLine에서 예외가 발생하면 다음과 같이 catch 구문에서 기본값을 반환하도록 예외를 처리하는 모습을 볼 수 있다.

`try-with-resources` 구조를 사용하기 위해서는 `AutoCloseable` 인터페이스를 사용해서 구현해야 한다. 자바 라이브러리는 서드파티 라이브러리에서도 `AutoCloseable`을 구현하거나 확장해뒀다. 그렇기 때문에 만약 닫아야 하는 작성을 구현해야 한다면 `AutoCloseable`을 반드시 구현해야 한다.

따라서 정리하면 다음과 같다.

1. 꼭 회수해야 하는 자원을 다룰 때는 `try-finally`를 사용하지 말고 `try-with-resources`를 사용하자.
2. 그 이유는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하기 때문이다.
3. 그리고 어떠한 예외도 없이  `try-with-resources`로  정확하고 쉽게 자원을 회수 할 수 있기 때문에 꼭 사용하자!

