## ITEM 48 : STREAM을 병렬로 만들때는 주의하라.

``` java
public static void main(String[] args) {
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
  return Stream.iterate(TWO,BigInteger::nextProbablePrime);
}
```

위 작업에 parallel() 함수를 추가한다고 해서, 성능이 개선되지는 않는다. 실제로 parallel() 함수를 추가하면 2배정도 시간이 소요된다.

>pipeline을 병렬화 하는 작업은 ```Stream.iterate``` 나 ```limit``` 이 사용된 경우에는 성능이 거의 좋아지지 않는다. stream 라이브러리가 병렬적으로 실행하는 방법을 모르기 때문이다

스트림 파이프라인을 무조건 병렬화 시켜서는 않된다. 잘못하면 퍼포먼스가 굉장히 않좋아진다.

> parallelism으로 성능이 향상되는 경우는 ArrayList, HashSet,ConcurrentHashMap 인스턴스; 배열; int ranges, long ranges 이다.

위 자료구조의 공통점은 2가지가 있다.

1. 특정 사이즈의 하위 범위로 정확하고, 효울적으로 쪼개질수 있다.

    결과적으로 스레드들이 병렬적으로 작업을 나누어 처리하기 쉬워진다. 이 작업을 실행하기 위해 streams 라이브러리에서 추상화된것이 ```spliterator``` 이며 ```Stream```, ```Iterable``` 에 대한 ```spliterator``` 함수에 의해서 리턴된다.

2. ```locality of reference``` (참조의 지역화)

   순차적으로 처리될때, 순차적인 원소는 메모리에서 함께 저장이 된다. 참조의 지역화는 bulk 작업을 병렬화 할때 매우 중요하다. 메모리에 있는 데이터를 processor 캐시로 저장하는 과정에서 스레드의 대기시간을 줄이는데 중요한 역할을 한다.

## stream을 병렬화 할때 주의할점

1. 전용 ```Stream```, ```Iterable```, ```Collection``` 을 구현할때, parallel peformance를 증가시키기 위해서는 양질의 ```spliterator``` 를 구현하는 것이 중요하다.
2. terminal operation이 간단해야하고, function object들이 서로 영향을 주어서는 않된다.
3. 스트림에 있는 원소수 * 실행되는 코드 라인 수 값이 최소 100,000(10만)은 될때 사용 해야한다.

## parallel 호출했을때 성능이 향상되는 예시

대표적으로 머신러닝, 데이터 프로세싱과 같은 특정 도메인에서는 이러한 속도향상을 누릴수 있다.

``` java
static long pi(long n) {
  return LongStream.rangeClosed(2,n)
    .mapToObj(BigInteger::valueOf)
    .filter(i->i.isProbablePrime(50))
    .count();
}
```

위 코드는 parallel 호출을 통해서 속도 향상을 얻을 수 있다.

``` java
static long pi(long n) {
  return LongStream.rangeClosed(2,n)
    .parallel()
    .mapToObj(BigInteger::valueOf)
    .filter(i->i.isProbablePrime(50))
    .count();
}
```

## 요약

계산의 정확성과 속도 향상에 대한 확실한 근거 없이는 스트림을 병렬화 하지 마라. 부적절하게 병렬화 할경우 프로그램이 죽거나 속도가 저하될 것이다. 만약 확실한 근거가 있다면, 현실적인 조건하에서 성능을 측정하라. 만약 성능이 향상되고 프로그램이 정확하게 동작한다면, 그때 프로덕션 코드에 적용하라.