# Item 86 : SERIALIZABLE을 구현할때는 주의해서 하라.

## 이유
Serializable을 구현시 다음과 같은 비용이 발생

1. 클래스 구현을 바꾸기 어려움
   - 구현을 변경하면 byte-stream encoding이 변경되고 클라이언트 코드가 영향을 받는다.

2. 버그와 보안적 결함을 가질 가능성 증가

3. 명시적으로 생성자를 호출하지 않기 때문에, 초기화과정을 잊거나 공격자 코드가 호출되도록 둘수 있음.

4. 테스트 비용증가
   - 버전들 사이에 serialize, deserialize 과정의 호환성이 테스트 되어야 한다.

## 추가적으로 주의할것
1. 상속할 수 있으면서 Serializable을 구현하는 클래스를 정의할때는 더 주의하기
   - 클라이언트 코드에서 ```finalize``` 를 정의하여 finalizer attack을 수행할 수 있다.

2. Inner class들은 Serializable을 구현하면 안됨
   - 필드들이 어떻게 클래스 정의에 대응하는지가 정의되지 않음