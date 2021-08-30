# Item63 : 문자열 연결은 느리니 주의하라

문자열 연결 연산자는 +를 사용하여 문자열들을 연결시킬 수 있다. 

하지만 한 줄 자리 출력값 혹은 작고 크기가 고정된 객체의 문자열 표현을 만들 때라면 괜찮지만, 본격적으로사용하기 시작하면 성능 저하를 일으킨다.

다음 예를 보자.

* 문자열 연결을 잘못 사용한 예

```java
public String statement() {
	String result = "";
	for (int i = 0; i < numItems(); i++) {
		result += lineForItem(i);
	}
	return result;
}
```

위 예제에서 문자열 연셜 연산자를 사용했다. 하지만 이 메소드는 심각하게 느려지는 현상을 볼 수 있다. 

그 이유는 문자열 연결 연산자는 문자열 n개를 잇는 시간이 n^2에 비례하기 때문이다. 문자열은 불변이기 때문에 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 하므로 성능 저하는 피할 수 없는 결과이다.

따라서 성능을 요구한다면 문자열 연결 연산자 대신 StringBuilder를 사용하자.

* StringBuilder를 사용한 예제

```java
public String statement2() {
	StringBuilder sb = new StringBuilder(newItems() * LINE+WIDTH);
	for (int i = 0; i < numItems(); i++) {
		sb.append(lineForItem(i));
	}
	return sb.toString();
}
```

자바 6이후 문자열 연결 성능을 다방면으로 개선했지만, 여전히 두 메소드의 성능 차이는 크다.

이펙티브 자바에서는 실제로 테스트한 결과 품목 100개의 lineForItem의 길이가 80인 문자열을 반환한 결과 6.5배 정도 statement2()가 빨랐다.



### 정리

문자열 연산자는 수행시간이 품목 수의 제곱에 비례하고 StringBuilder는 선형으로 늘어나니 성능을 요구한다면 StringBuilder를 사용하자.

Statement2() 에서 StringBuilder를 전체 결과를 담기에 충분한 크기로 초기화 한점을 잊지 말자. 그렇다고 초기화를 안해도 StringBuilder가 훨씬 빠르다.