# Item 66 : Native method를 현명하게 사용하라.

JNI(Java Native Interface)는 자바 프로그램에서 native method를 호출할 수 있게 해준다. 

Platform specific 기능들에 접근하기 위해서 native method를 사용하는것은 합법적이지만 대부분 그럴 필요가 없다.

Java에서 이미 같은 기능을 구현한 라이브러리가 있는 경우가 대부분이기 때문이다.

예전에는 성능 최적화를 위해서, native 라이브러리를 사용하기도 했지만, 이제는 Java 라이브러리가 성능이 개선이 되었기 때문에 사용할 필요가 없어졌다. 예를들어, ```BigInteger``` 클래스의 경우가 그렇다.

## Native method의 단점

1. 메모리 corruption error에 취약해진다. garbage collector가 관리하주지 못한다.
2. 이식성이 낮다. 
3. 디버깅 하기 어렵다.
4. native code를 실행하고, 결과를 받는 절차에 비용이 든다.
5. native code를 실행하기 위한 glue code를 구현하는데, 시간과 노력이 많이 든다.

## 요약

native method를 사용하는것을 신중하게 생각하라. 만약  low-level 리소스나 native 라이브러리를 꼭 써야한다면, 가능하면 적게 쓰도록 하고 테스트도 충분히 해라.