# Item 70 : Recoverable 조건에서는 checked exception 을 사용하고 프로그래밍 에러일때는 Runtime exception 을 사용하라.

## 세가지 유형의 throwables

1. checked exception

   - 사용자는 반드시 catch문으로 exception 을 헨들링 해야함.
   - 사용자는 exception에 대한 해결로직을 구현해야함.
   - Exception 클래스를 상속받음.

2. unchecked throwable

   - 절대 catch 하면 안됨.
   - 보통 exception 회복이 불가능하다고 판단 또는 계속되는 실행이 더 시스템에 악영향을 줌.
   - 결국 현재 스레드가 죽고, 에러를 출력
   - 주로 프로그래밍 에러를 나타내기 위해서 사용함.(예 : ArrayIndexOutOfBounds)
   - RuntimeException을 상속받음.

   1. runtime exception

   2. error

      - error는 보통 JVM이 사용하도록 예약되어 있다. 
      - 보통 리소스 부족, invariant failure(불변성 실패) 등을 나타낼때 사용된다.

      - Error 클래스를 상속해서 구현해서 만들수 있으나, 보통 새로 만들지 않는것이 관습.

## 요약

- recoverable한 상태이면 checked exception을 던지고, 프로그래밍 에러일때는 unchecked exception을 던져라. 
- 잘 모르겠으면, unchecked exception 을 던져라
- runtime exception, checked exception이 아닌 throwable 클래스는 정의하지마라.
-  checked exception의 recovery를 돕기위해서, 함수를 제공해라