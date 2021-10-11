# Item 84 : Thread Scheduler에 의존하지마라

## 이유

운영체제마다 스레드 스케줄링 방식이 달라서, 모든 운영체제에서 같게 동작한다는 보장이 없다.

## 해결방법

1. runnable thread의 평균수를 프로세서 수 보다 너무 많게 설정하지 않기

   - busy waiting인 스레드를 줄이기

   ``` java
   public class SlowCountDownLatch {
     private int count;
     public SlowCountDownLatch(int count) { 
       if (count < 0)  throw new IllegalArgumentException(count + " < 0"); 	
       this.count = count;
     }
     public void await() {
       while (true) { 
         synchronized(this) {
         if (count == 0) return;
         }
       }
     }
     public synchronized void countDown() { 
       if (count != 0) count--; 
     }
   }
   
   ```

2. ```Thread.yield``` 또는 Thread priority 조정으로 문제를 고치려고 하지 말고 근본문제를 파악하고 해결하기

   - JVM 구현체에 따라 효과가 있을수도 있고 없을수도 있다.

   - 이미 잘 동작중인 프로그램에 대해서는 시도할수 있다. 