# ITEM 18: 상속보다 구성을 선호하라

평범한 concrete 클래스를 상속하는것은 위험하다. 슈퍼 클래스의 구현이 계속 바뀔수 있고, 바뀌는 경우 서브 클래스가 고장날 수 있다. 단 슈퍼클래스의 작성자가 상속을 목적으로 정의한 경우라면 상속해도 좋다.

부적절한 상속의 예는 아래와 같다.

``` java
public class InstrumentedHashSet<E> extends HashSet<E> {

    private int addCount = 0;

    public InstrumentedHashSet(){}

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap,loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(Arrays.asList("Snap","Crackle","Pop"));
        System.out.println(s.getAddCount()); // 6
    }
}

```

위 경우에, ```s.getAddCount()``` 가 6을 리턴하는데, 그 이유는 ```add``` 함수를 기반으로 구현이 되어있기 때문이다. 위 코드는 모든 자바 버전에서 동작한다고 보장할 수 없다. 추후에 ```HashSet``` 구현이 변경되서, 예상치 못하게 동작할 가능성이 존재한다. 여기서 ```addAll``` 함수에서 ```add``` 함수를 호출하는 방식으로 구현하면 해결은 되자만 모든 문제가 해결되는 것은 아니다. 슈퍼 클래스의 함수를 호출할 수 없는 경우도 존재한다. 슈퍼 클래스의 함수를 호출하지 않는 함수를 새로 정의하는 경우에는 더 안전하다고 말할수는 있지만, 다음버전에 운이 나쁘게도 슈퍼클래스에 반환형은 다르고 동일한 메소드 시크니쳐를 가진 함수가 추가되면 더이상 코드가 컴파일 되지 않게 된다. 

이러한 문제점을 해결하는 방법이 바로 구성(Composition)이다. 새로운 클래스의 인스턴스 함수는 포함된 인스턴스에 대한 함수를 호출하는데, 이를 ```forwarding``` 이라 부른다. 

``` java
public class ForwardingSet<E> implements Set<E> {

    private final Set<E> s;

    public ForwardingSet(Set<E> s) { this.s = s; }

    @Override
    public int size() { return s.size(); }
    @Override
    public boolean isEmpty() { return s.isEmpty(); }
    @Override
    public boolean contains(Object o) { return s.contains(o); }
    @Override
    public Iterator<E> iterator() { return s.iterator(); }
    @Override
    public Object[] toArray() { return s.toArray(); }
    @Override
    public <T> T[] toArray(T[] a) { return s.toArray(a); }
    @Override
    public boolean add(E e) { return s.add(e); }
    @Override
    public boolean remove(Object o) { return s.remove(o); }
    @Override
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    @Override
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
    @Override
    public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
    @Override
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    @Override
    public void clear() { s.clear(); }
    @Override
    public boolean equals(Object o) { return s.equals(o); }
    @Override
    public int hashCode() { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```



``` java
public class InstrumentedSet<E> extends ForwardingSet<E> {

    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

```

위와 같이 wrapper class는 어떠한 ```Set``` 구현체에도 사용할 수 있다.

``` java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

상속은 오직 서브 클래스가 슈퍼 클래스의 하위타입인 경우에만 사용해야한다. 즉, 클래스 B는 클래스 A를 오직 "is-a" 관계가 성립할때만 상속해야 한다.

Java Platform 라이브러리에는 이 규칙을 어긴 경우가 많이 있는데, 예를 들면 ```Stack``` 은 ```Vector``` 의 일종이 아닌데도 불구하고 ```Stack``` 이 ```Vector``` 를 상속했다. 

구성이 적절한 곳에 상속을 하면, 쓸대없이 슈퍼 클래스의 세부구현을 노출시키게 되고, API가 상위 클래스의 구현에 묶이게 된다. 또한 사용자 또한 해깔릴수 있다. 예를들면, p가 ```Properties``` 인스턴스라고 할때, ```p.getProperty(key)``` 와 ```p.get(key)``` 두가지 선택지가 생기므로 사용자는 어떤것을 사용할지 해깔릴수 있다.

## 정리

상속은 유용하지만, 캡슐화를 위반하기 때문에 문제가 있다. 따라서 서브 클래스와 슈퍼 클래스 간에 진정한 서브타입 관계(is-a 관계) 가 있을때 사용해야 한다. 그외의 경우, 구성과 ```forwarding``` 을 사용해야 한다.