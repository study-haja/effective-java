# Item87 : 커스텀 직렬화 형태를 고려해보라

클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용하면 다음 릴리스 때 고치거나 버릴려 할 때 발목을 잡을 확률이 크다. 즉 기본 직렬화 형태를 버릴 수 없게 되버려 이와 같은 문제를 낳게 된다. 실제로 BigInteger 같은 일부 자바 클래스가 이 문제에 시달리고 있다.

**따라서 먼저 고민해보고 괜찮다고 판단 될 때만 기본 직렬화 형태를 사용하라.** 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다. 



### 직렬화에 적합한 예

기본 직렬화 형태는 객체가 포함한 데이터뿐만 아니라 그 객체를 시작으로 접근할 수 있는 모든 객체와 객체들의 연결된 정보까지 나타낸다. 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다. **객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태를 선택해도 무방하다.** 다음 예를 보자.

* 사람의 성명을 간략히 표현한 객체 - 기본 직렬화 형태에 적합

```java
public class Name implements Serializable {
    /**
     * 성. null이 아니어야 한다.
     * @serial
     */
    private final Stirng lastName;

    /**
     * 이름. null이 아니어야 한다.
     * @serial
     */
    private final String firstName;

    /**
     * 중간이름. 중간이름이 없다면 null
     * @serial
     */
    private final String middleName;

    ... // 나머지 코드는 생략
}
```

이름은 논리적으로 성, 이름, 중간 이름이라는 3개의 문자열로 구성하는데 위 클래스의 인스턴스 필드들은 이 논리적인 구성 요소를 정확하게 반영했다.

**기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메소드를 제공해야 할 때가 많다.** 앞의 Name 클래스의 경우에는 readObject 메소드가 lastName과 firstName 필드가 null이 아님을 보장해야 한다.



### 직렬화에 적합하지 않은 예

그렇다면 직렬화 형태에 적합하지 않은 예를 한번 보자.

* 직렬화에 적합하지 않는 코드

```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    // ... 생략
}
```

위 코드에서 논리적으로는 일련의 문자열을 표현했고 물리적으로는 문자열들을 이중 연결 리스트로 연결했다. 하지만 이 클래스에 기본 직렬화 형태를 사용하면 각 노드에 연결된 노드들까지 모두 표현할 것이다.

따라서 객체의 물리적 표현과 논리적 표현의 차이가 클 때는 아래와 같은 문제가 생긴다.

1. 공개 API 가 현재의내부 표현 방식에 영구히 묶인다. 
   1. 예를 들어, 앞에 코드에서 StringList.Entry가 공개 API가 되어 버린다면, 다음 릴리스에서 내부 표현 방식을 바꾸더라도 StringList 클래스는 여전히 연결 리스트로 표현된 입력도 처리할 수 있어야 한다. 즉 연결 리스트를 사용하지 않더라도 관련 코드는 절대 제거 할 수 없다.
2. 너무 많은 공간을 차지한다.
   1. 위 예의 직렬화 형태는 연결 리스트의 모든 엔트리와 연결 정보까지 기록했지만, 엔트리와 연결 정보는 내부 구현에 해당되니 직렬화 형태에 포함할 가치가 없다.
   2. 이처럼 직렬화 형태가너무 커져서 디스크에 저장하거나 네트워크로 전송하는 속도가 느려진다.
3. 시간이 너무 많이 걸릴 수 있다.
   1. 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없으니 그래프를 직접 순회해볼 수밖에 없다.
4. 스택 오버플로를 일으킬 수 있다.
   1. 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 이 작업은 중간 정도 크기의 객체 그래프에서도 자칫 스택 오버플로를 일으킬 수 있다.



###  합리적인 직렬화 형태

그렇다면 합리적인 직렬화 형태는 어떤 모습일까? 단순히 리스트가 포함한 문자열의 개수와 문자열들만 있으면 될 것이다. 물리적인 상세 표현은 배재한 채 논리적인 구성만을 담으면 된다. 위 코드를 통해 예를 한번 보자.

* 합리적인 커스텀 직렬화 형태를 갖춘 StringList

```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;

    // 이번에는 직렬화 하지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    // 지정된 문자열을 이 리스트에 추가한다.
    public final void add(String s) { ... }

    /**
     * StringList 인스턴스를 직렬화한다.
     */
    private void writeObject(ObjectOutputStream stream)
            throws IOException {
        stream.defaultWriteObject();
        stream.writeInt(size);

        // 모든 원소를 올바른 순서대로 기록한다.
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }

    private void readObject(ObjectInputStream stream)
            throws IOException, ClassNotFoundException {
        stream.defaultReadObject();
        int numElements = stream.readInt();

      	// 모든 원소를 읽어 이 리스트에 삽입한다.
        for (int i = 0; i < numElements; i++) {
            add((String) stream.readObject());
        }
    }
    // ... 생략
}
```

`transient` 키워드가 붙은 필드는 기본 직렬화 형태에 포함되지 않는다. StringList의 필드가 모두 `transient`더라도`writeObject`와 `readObject`는 각각 가장 먼저 `defaultWriteObject`와 `defaultReadObject` 메서드를 호출한다. 

클래스의 인스턴스 필드 모두가 `transient`면 `defaultWriteObject`와 `defaultReadObject`를 호출하지 않아도 된다고 들었을 수도 있지만, 직렬화 명세는 이 작업을 무조건 하라고 요구한다. 이렇게 해야 향후 릴리즈에서 `transient`가 아닌 필드가 추가되더라도 상위와 하위 모두 호환이 가능하기 때문이다.

신버전의 인스턴스를 직렬화한 후에 구버전으로 역직렬화하면 새로 추가된 필드는 무시될 것이다. 그리고 구버전 `readObject` 메서드에서 `defaultReadObject`를 호출하지 않는다면 역직렬화 과정에서 `StreamCorruptedException`이 발생할 것이다.

그리고 성능 문제에 있어서 문자열들의 길이가 평균 10이라면, 개선 버전의 StringList는 원래버전에서 절반 정도의 공간을 차지하며, 스택 오버플로가 전혀 발생하지 않아 실질적으로 직렬화 할 수 있는 크기 제한이 사라졌다.



### 결론

기본 직렬화 여부에 관계없이 `defaultWriteObject` 메서드를 호출하면 `transient`로 선언하지 않은 모든 필드는 직렬화된다. 따라서 `transient` 로 선언해도 되는 인스턴스 필드에는 모두 `transient`키워드를 선언해야 한다. **해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 `transient` 한정자를 생략해야 한다.**

기본 직렬화를 사용한다면 역직렬화할 때 `transient` 필드는 기본값으로 초기화된다. 기본값을 변경해야 하는 경우 `readObject` 메서드에서 `defaultReadObject` 메서드를 호출한 다음 원하는 값으로 복원하면 된다. 혹은 그 값을 처음 사용할 때 초기화하는 방법도 있다.



### 직렬화와 동기화

**기본 직렬화 사용 여부와 상관없이 직렬화에도 동기화 메커니즘을 적용해야 한다.** 예컨데 모든 메서드를 `synchronized`로 선언하여 스레드 안전하게 만든 객체에 기본 직렬화를 사용하려면, `writeObject`도 아래처럼 수정해야 한다.

* 기본직렬화를 사용하는 동기화된 클래스를 위한 writeObject 메소드

```java
private synchronized void writeObject(ObjectOutputStream stream) // synchronized 사용
        throws IOException {
    stream.defaultWriteObject();
}
```

`writeObject` 메소드 안에서 동기화 하고 싶다면 클래스의 다른 부분에서 사용하는 락 순서를 똑같이 따라야 한다. 안그러면 자원 순서 교착상태에 빠질 수 있다.



### 직렬 버전 UID

또한 어떤 직렬화 형태를 선택하더라도 직렬화가 가능한 클래스에는 SerialVersionUID(이하 SUID)를 명시적으로 선언해야 한다. 이렇게 하면 잠재적인 호환성 문제가 사라진다. 그리고 선언하지 않아도 자동 생성되지만 런타임에 이 값을 생성하느라 복잡한 연산을 수행해야 한다. 직렬 버전 UID 선언은 다음과 같이 해주면 된다.

* SerialVersion UID를 만드는 코드

```java
// 무작위로 고른 long 값
private static final long serialVersionUID = <무작위로 고른 long 값>;
```

새로 작성하는 클래스에서는 어떤 long 값을 선택하든 상관없다. 클래스 일련번호를 생성해주는 serialver 유틸리티를 사용해도 되며, 아무 값이나 넣어도 된다.

그리고 SUID가 꼭 유니크할 필요는 없다. 다만 이 값이 변경되면 기존 버전 클래스와의 호환을 끊게 되는 것이다. 따라서 호환성을 끊는 경우가 아니라면 SUID 값을 변경해서는 안 된다.