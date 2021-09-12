## Item 74: 각각의 함수에서 던질수 있는 모든 exception을 문서화하라.

- checked exception 뿐 아니라, unchecked exception 또한 문서화 해야 한다.

- unchecked exception에 대해서는, 정상적으로 동작하기 위한 precondition(사전 조건)을 명시해주어야 한다.

- @throw 어노테이션을 사용해서 각각의 exception을 문서화 하되, runtime exception에 대해서는 throws 키워드를 사용하지마라. 즉 아래와 같이 사용해서는 않된다. 

  ``` java
  public void test() throws RuntimeException
  ```

  보통 @throw tag로 생성된 문서에서 ```throws``` 키워드가 있는 경우에는 checked exception으로 인지되고, ```throws``` 키워드가 없는 경우에는 unchecked exception으로 인식 되기 때문이다.

- 각각의 함수가 던질수 있는 모든 unchecked exception을 문서화하는 것이 이상적이나, 해당 함수가 호출하고 있는 모든 함수에서 던질수 있는 exception에 대해서 문서화 하는것은 비현실적이다. 왜냐하면 호출되는 함수가 언제 수정이 될지 알수가 없기 때문이다. 따라서 이런 경우에는 완벽히 문서화를 하지 않아도 된다.

- 클래스에있는 함수들이 같은 exception을 던질 수 있다면, 주석을 사용하는 것이 좋다.

  - 예를들어, ```이 클래스에 있는 모든함수에서 파라미터에 Null객체가 전달되면, NullPointerException을 던질수 있습니다.``` 라고 명시할 수 있다.

## 요약

작성한 모든 함수에서 발생할수 있는 checked/unchecked exception에 대해서 문서화하라. 만약 그렇게 하지못하면, 클라이언트들이 API를 효과적으로 사용할 수 없다.