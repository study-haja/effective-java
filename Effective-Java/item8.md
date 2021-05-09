## Finalizer(소멸자)와 Cleaner 사용을 피하라.

### Finalizer란?

Object 클래스의 메소드 타입중 하나로써, Garbage Collection이 수행될때 호출되는 메소드. 주로 자원을 해제하기 위해서 사용된다. Java 9에서는 deprecated됨.

``` java
public class FinalizerTest extends Object {

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalizer is called!");
    }
}
```



### Cleaner란?

Java 9부터 생겨났고

수거대상이 되는 객체를 ```Cleaner``` 의 register함수로 등록한다.

아래와 같이 수거대상이 되는 ```State``` 객체에 대한 정리 작업이 ```run()```  함수에서 수행된다.

``` java
import java.lang.ref.Cleaner;

public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("Room Clean");
            numJunkPiles = 0;
        }
    }

    private final State state;
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this,state);
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}

```

run 함수가 호출되는 2가지 상황은 아래와 같다.

1. Room 클래스의 close 메소드를 명시적으로 호출하는 경우
2. Garbage Collector가 Room객체를 회수할때

### 피해야하는 이유

1. 객체가 unreachable 되는 시점에 메모리가 즉시 회수되지 않는다. 그러므로 finalizer나 cleaner에서 time-critical한 일들은 해서는 안된다. 예를들면, file close 작업이 finalizer나 cleaner에서 수행된다면 시스템 지연되어 file close가 되지않아서 더이상 파일을 열수가 없게된다.

2. JVM 구현체에 따라서, finalizer와 cleaner가 호출될지 안될지 확신할수 없고, 호출되더라도 언제 호출될지 알수없다.

   - finalizer의 경우 어느 스레드에 의해서 실행될지 알수 없다.
   - Cleaner의 경우 class 작성자가 cleaner thread를 통제한다는 점에서, finalizer보다 낫다. 하지만 cleaner 또한 garbage collector의 통제를 받기 때문에 즉각적으로 호출된다는 보장이 없다.

3. finalizer에서 발생한 uncaught exception은 무시된다. uncaught exception이란 사용자가 직접적으로 처리하지 않은 예외를 말한다. uncaught exception이 발생하면 finalization이 종료가 되고 다른 스레드가 해당 객체에 접근하게 되면 예상할 수 없는 결과가 발생할 수 있다.
   
   ``` java
   public class FinalizerThrowingErrorTest {
       @Override
       protected void finalize() throws Throwable {
           super.finalize();
         	cleanTask1(); // 만약 exception이 발생한다면, 아래 로직은 실행되지 않는다.
         	cleanTask2();
       }
   }
   ```
   
   ``` java
   //caught exception 예시
   try {
     throw new NullPointerException("Null pointer reference error!");
   }
   catch(NullPointerException e) {
     System.out.println("caught!");
   }
   
   //uncaught exception 예시
   int[] arr = new int[10]
   arr[12] = 10; // array index out of bound exception
   ```
   
4. Finalizer와 cleaner를 사용하는데는 심각한 퍼포먼스 약점이 있다. 다음과 같이 try-with-resources 방식으로 메모리를 해제하면 12ns가 소요된다.

   #### try-catch-finally 방식의 자원해제

   ```  java
    public static void main(String[] args) throws IOException {
           FileInputStream is = null;
           BufferedInputStream bis = null;
           try {
               is = new FileInputStream("file.txt");
               bis = new BufferedInputStream(is);
               int data = -1;
               while ((data = bis.read()) != -1) {
                   System.out.println((char)data);
               }
           } finally {
               if (is != null) is.close();
               if (bis != null) bis.close();
           }
       }
   ```

   #### try-with-resources 방식의 자원해제

   ```java
       public static void main(String[] args) {
           try (
                   FileInputStream is = new FileInputStream("file.txt");
                   BufferedInputStream bis = new BufferedInputStream(is);
           ) {
               int data = -1;
               while ((data = bis.read()) != -1) {
                   System.out.print((char) data);
               }
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   ```

   반면 finalizer를 사용해서 메모리를 해제하면 550ns가 소요된다. 50배정도 더 느려짐을 알수 있다. 이렇게 느려진 이유는 finalizer가 즉시 호출되지 않기 때문이다.

5. finalizer attack에 노출된다. 아래의 예시는 finalize에서 생성되지 말아야할 객체가 만들어진 예시이다. 

   ``` java
   class Vulnerable {
       Integer value = 0;
   
       Vulnerable(int value) {
           if (value <= 0) {
               throw new IllegalArgumentException("Vulnerable value must be positive");
           }
           this.value = value;
       }
   
       @Override
       public String toString() {
           return (value.toString());
       }
   }
   public class AttackVulnerable extends Vulnerable {
       static Vulnerable vulnerable;
   
       public AttackVulnerable(int value) {
           super(value);
       }
   
       public void finalize() {
           vulnerable = this; // 제거되야할 객체를 vulnerable 변수가 참조하게 되므로, GC되지 않는다!
       }
   
       public static void main(String[] args) {
           try {
               new AttackVulnerable(-1);
           } catch (Exception e) {
               System.out.println(e);
           }
           System.gc();
           System.runFinalization();
           if (vulnerable != null) {
               System.out.println("Vulnerable object " + vulnerable + " created!");
           }
       }
   }
   ```

   이를 방지하기 위해서, 상위 클래스를 final로 정의하여 상속을 막을 수 있다. non final class로 선언해야 하는 경우는 final ```finalize```를 정의하여 재정의를 막을수 있다. 
   
   ``` java
   class Vulnerable {
       Integer value = 0;
   
       Vulnerable(int value) {
           if (value <= 0) {
               throw new IllegalArgumentException("Vulnerable value must be positive");
           }
           this.value = value;
       }
   		
     	@Override
       protected final void finalize() throws Throwable { // 상속을 막음!
           super.finalize();
       }
   }
   ```
   
   

### 문제가 발생하는 구체적인 사례

1. 장기간 동작하던 GUI application이 OOM을 만나서 죽는 상황이 발생할 수 있다. 이유는 여러 Graphical 객체들을 회수하는  finalizer가 실행되지 않아서 발생하였다. 
2. database와 같은 공유된 자원에 persistent lock을 해제하는 로직을 finalizer나 cleaner에 두었을때, 결국 lock을 해제되지 않아 분산 시스템이 동작을 멈출수도 있다.

### ```System.gc```, ```System.runFinalization``` 함수

finalizer나 cleaner가 실행될 가능성은 높여줄수 있지만, 보장해주지 못한다.

### ```System.runFinalizersOnExit``` , ```Runtime.runFinalizersOnExit``` 

이 함수들또한 결점이 있고 몇십년동안 deprecated 되었다.

### 해결방안

자원을 해제해야 하는 경우, finalizer와 cleaner대신에 무엇을 사용해야 할까? 이 경우에는 ```Autocloseable``` 인터페이스를 구현하는 클래스를 만들면 된다. 그리고 클래스 사용자가 ```close()``` 함수를 호출하도록 구현하면 된다. 이 경우에는 클래스에서 해당 자원이 해제되었다는 사실을 기록해두어야 하고, 자원이 해제된다음 접근할때 ```IlleagalStateException``` 을 던지도록 구현해야한다.

### Cleaner와 Finalizer는 언제 사용하는가?

1. 자원 사용자가 ```close``` 함수 호출하는것을 잊어버렸을 경우, 대신 자원을 해제하는 용도로 사용할 수 있다. 하지만 즉시 자원을 해제 한다는 보장이 없지만, 자원을 해제 하지 않는것보다 늦게라도 해제하는것이 좋다. 이때 finalizer와 cleaner가 가진 부작용에 염두해두어야한다. 예를들어 ```FileInputStream```, ```FileOutputStream```, ```ThreadPoolExecutor```, ```java.sql.Connection``` 은 이러한 안전장치가 finalizer에 구현되어있다.
2. Native peer를 가진 객체의 경우또한 합법적으로 사용할 수 있다. native peer란 Java 객체가 native method를 통해서 호출하는  native 객체(예를들면 C로 구현된 객체)이다. native peer는 garbage collector에 의해서 회수되지 않는다. 퍼포먼스가 괜찮고 native peer가 중요한 자원(필요없을시 즉시 해제되어야하는 자원)을 소유하고 있지 않으면, cleaner나 finalizer는 이를 대신수행할수 있다. 만약 native peer가 즉시 회수되어야 하는 자원을 갖고 있다면 ```close``` 함수를 대신 사용해야 한다.

### 정리

안전장치 용도나 중요하지 않은 자원들을 회수하는 경우를 제외하고, cleaner나 finalizer를 사용하지 말라. 만약 사용하게 된다면 일관적이지 않은 동작 (indeterminacy)와 퍼포먼스 문제를 고려하라.

