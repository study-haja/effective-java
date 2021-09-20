# Item 76 : Failure Atomicity(실패 원자성)을 보장하려고 노력하라.

## 실패원자성

실행도중 실패한 함수가 실패하더라도, 원본 객체는 변함없는 성질

## 실패원자성 보장 방법

1. immutable 객체 생성

2. 함수 실행전 파라미터를 체크

    ``` java
    public Object pop() {
    	if (size == 0)
      throw new EmptyStackException();
      Object result = elements[--size];
      elements[size] = null; // Eliminate obsolete reference return result;
    }
    ```

3. 실패할만한 로직을 함수의 앞부분에 배치

4. 원본객체를 임시 객체로 복사한뒤, 작업이 성공하면 원본객체를 대체

5. 함수 실행도중 발생한 실패를 캐치하여 원래 상태로 롤백

   - 단, 멀티스레딩 환경에서 적절한 동기화 없이 객체를 변경한 경우, 불가능

## 요약

실패원자성을 보장하려고 노력하되, 보장할 수 없으면 문서에 명시하여야 한다.