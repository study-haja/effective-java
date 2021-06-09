# ITEM 24 : NONSTATIC 보다 STATIC 클래스를 선호하라.

Nested class(이하 중첩클래스)는 오직 Enclosing class(이하 동봉 클래스)를 돕기 위해서만 사용되어야 한다. 만약 그렇지 않으면 top level 클래스로 정의해야 한다.

4가지 종류의 중첩 클래스가 존재한다.

> 1. static member class
> 2. nonstatic member class
> 3. anonymous class
> 4. local class
>
> 여기서 2,3,4 번을 inner 클래스라고도 부른다.

## 1. Static member class

가장 흔한 사용은 오직 동봉클래스와 연계해서 사용되는 public helper class이다.

``` java
public class Calculator {

    // ...

    public static enum Operation {
        PLUS,MINUS;
    }
}
```

private static member class의 흔한 사용은  동봉 클래스에 의해서 사용되는 객체의 컴포넌트를 나타낼때 이다. 예를 들어, ```Map``` 인스턴스는 key-value pair에 대해서  ```Entry``` 객체를 가지고 있는데, 각각의 ```Entry```에서 ```Map``` 에 접근할 필요는 없다. 물론 static을 생략해도 동작은 하겠지만, 메모리 공간과 시간이 낭비된다.

``` java
public class ExportedAPIClass {

    public class PublicNestedClass {
    }

    protected class ProtectedNestedClass {
    }
}

```

만약 위 처럼 외부로 노출된 API 클래스에 중첩 클래스를 정의했다면, static으로 선언하느냐 non static으로 선언하느냐는 더욱 중요해진다. 왜냐하면 한번 선언한후 변경하게 되면, 클라이언트 코드가 더이상 지원하지 않을 수 있기 때문이다.

## 2. Non Static Member class

동봉 클래스의 함수와 동봉 객체에 대한 레퍼런스를 얻을 수 있다. 오직 동봉 클래스가 객체화 되어야, 객체화 될수 있다.

``` java
public class EnclosingClass {

    private int member1;
    public NestedClass funcA() {
        return new NestedClass();
    }
    class NestedClass {

        public void funcA() {
            this.funcA(); // this로 동봉 객체 레퍼런스 획득 가능
            funcA(); // 동봉 객체의 함수 호출 가능
            int a = member1;
        }
    }
  
     public static void main(String[] args) {
        EnclosingClass ec = new EnclosingClass();
        NestedClass nc = ec.new NestedClass();
        // NestedClass nc2 = new NestedClass(); -> 컴파일 에러
    }
}
```

만약 동봉 클래스의 인스턴스와 독립적으로 존재할수 있다면, static member class로 정의해야 한다. 

다음은 전형적인 non static member class의 예시이다.

``` java
public class MySet<E> extends AbstractSet<E> {
    
    // ...
    
    @Override public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
			// ...
    }
}

```

만약 멤버 클래스가 동봉 인스턴스에게 접근할 필요가 없다면, 선언에 static modifier를 추가해야한다.

만약 그렇지 않으면, 동봉 인스턴스에 대한 레퍼런스를 가지게 되고, 이 레퍼런스를 저장하는데는 시간과 메모리 공간이 소모된다. 그리고 동봉 인스턴스가 GC가 되지 않게 된다. 이런경우, 메모리 릭이 발생할 수 있고 디버깅하기가 매우 까다롭다.

## 3. Anonymous class

익명 클래스는 다음의 특징이 있다.

- 선언과 동시에 객체를 생성해야 한다.
- ```instanceof``` 테스트를 할수 없고, 클래스 이름이 필요한 작업은 수행할 수 없다.
- 여러 인터페이스를 구현할 수 없고, 인터페이스와 추상클래스를 동시에 상속할 수 없다.
- 익명클래스의 클라이언트는 익명 클래스가 상속한 상위 클래스의 멤버들에만 호출할 수 있다.
- 10 줄 미만으로 작성해야 하며, 그렇지 않을 경우 가독성이 매우 안좋아진다.

``` java
public class AnonymousClassExample {
    private double x;
    private double y;

    public AnonymousClassExample(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double operate() {
        Operator operator = new Operator() {
            @Override
            public double plus() {
                return x+y;
            }

            @Override
            public double minus() {
                return x-y;
            }
        };
        return operator.plus();
    }

    public static void main(String[] args) {
        AnonymousClassExample anonymousClassExample = new AnonymousClassExample(1.0,2.0);
        System.out.println(anonymousClassExample.operate());
    }
}

interface Operator {
    double plus();
    double minus();
}
```

## 4. Local class

``` java
public class LocalClassExample {

    private int number;

    public LocalClassExample(int number) {
        this.number = number;
    }

    public void foo() {
        class LocalClass {
            private String name;

            public LocalClass(String name) {
                this.name = name;
            }

            public void print() {
                System.out.println(number + name);
            }
        }
        LocalClass localClass = new LocalClass("local");
        localClass.print();
    }

    public static void main(String[] args) {
        new LocalClassExample(10).foo();
    }
}
```

## 정리

중첩 클래스에는 4가지 타입이 있으며, 각자 사용되는 상황이 다르다. 만약 클래스가 단일 메소드 밖에서 접근 가능 해야 하거나, 단일 메소드에서 정의하기에 코드가 너무 긴 경우, 멤버 클래스를 사용해야 한다. 만약 동봉 인스턴에 대해서 접근가능해야 한다면 non static으로 정의해야 한다. 그렇지 않으면 static으로 정의하라. 만약 클래스가 메소드에 속하면서 오직 한 장소에서만 객체를 생성해야 하며, 클래스의 특징에 맞는 타입이 존재한다면 익명 클래스로 정의하라. 그렇지 않으면 로컬 클래스를 사용하라.



