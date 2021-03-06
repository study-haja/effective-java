# Item59 : 라이브러리를 익히고 사용하라

* 무작위 정수 하나를 생성하는 문제가 심각한 코드

```java
static Random rnd = new Random();

static int random(int n) {
	return Math.abs(rnd.nextInt()) % n;
}

// 무작위 수 100만개를 생성한 다음, 그중 중간 값보다 작은 게 몇 개인지를 출력하는 코드
public static void main(String[] args) {
	int n = 2 * (Integer.MAX_VALUE / 3);
	int low = 0;
	for (int i = 0; i < 1000000; i++) {
		if (random(n) < n / 2)
			low++;
	System.out.println(low);
	}
}
```

위 코드에서는 3가지 문제가 있다.

1. n이 그리 크지 않은 2의 제곱수라면 얼마 지나지않아 같은 수열이 반복된다.
2. n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환한다.
   1. n 값이 크면 이 현상은 더 두드러진다.
3. 지정한 범위 바깥의 수가 종종 튀어나올 수 있다.
   1. random 메소드가 이상적으로 동작한다면 약 50만 개가 출력돼야 하지만, 실제로 돌려보면 666666개의 가까운 값을 얻는다. 즉 무작위로 생성된 수 중에서 2/3 가량이 중간값보다 낮은 쪽으로 쏠린 것이다.
   2. 그 이유는 rnd.nextInt()가 반환한 값을 Math.abs를 이용해 음수가 아닌 정수로 매핑하기 때문이다.
   3. 따라서 rnd.nextInt()값이 Integer.MIN_VALUE가 나오면 정수로 매핑이 되어 random()값이 음수가 나올 수 있다.

위 문제를 해결하려면 의사난수, 정수론, 2의 보수 계산 등을 알아야 하지만 우리는 그럴 필요가 없다. 이미 Random.nextInt()가 해결했기 때문이다. 또한 이미 검증된 메소드이기 때문에 자세한 동작방식은 몰라도 된다.



### 표준 라이브러리 사용 이점

표준 라이브러리를 사용하면 다음과 같음 이점이 있다.

1. 표준 라이브러리를 사용하면 그 코드를 작성한 전문가의 지식과 여러분보다 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다.
2. 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간을 허비하지 않아도 된다.
3. 따로 노력하지 않아도 성능이 지속해서 개선된다.
4. 기능이 점점 많아진다.
5. 우리가 작성한 코드가 많은 사람에게 낯익은 코드가 된다는 것이다. (유지보수, 재활용 코드)

이렇게 많은 이점이 있는데 사용자들은 이를 몰라 직접 만들어서 사용한다. 그렇기 때문에 메이저 릴리스마다 주목할 만한 수많은 기능이 라이브러리에 추가되는데 한번쯤 읽어보길 권장한다. 예를 들어 자바 9의 추가된 transferTo 메소드를 통해 지정한 url 의 내용을 쉽게 가져올수 있게 되었다.

* transferTo 메소드를 이용해 URL의 내용 가져오는 코드

```java
public static void main(String[] args) throws IOException {
	try (InputStream in = new URL(args[0]).openStream()) {
		in.transferTo(System.out);
	}
}
```

그리고 적어도 자바 개발자라면 java.util, java.lang, java.io와 그 하위 패키지들에는 익숙해지길 바란다. 그렇다고 라이브러리들은 매년 빠르게 성장하고 있으니 모든 기능을 알고 있으라고 하는건 아니다.

또 라이브러리를 사용하려고 시도해보자. 어떤 영역의 기능을 제공하는지 살펴보고 우리가 원하는 기능이 아니라 판단되면 그때 대안을 사용하자. 서드파티 라이브러리까지 찾아보고 그마저 적절한 기능을 찾지 못했다면 다른 선택이 없으니 직접 구현하자.





