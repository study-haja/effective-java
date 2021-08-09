# ITEM 52 : 오버로딩을 현명하게 사용하라.

``` java
public class CollectionClassifier {

  public static String classify(Set<?> s) {
		return "Set"; 
  }

  public static String classify(List<?> lst) { 
    return "List";
	}

  public static String classify(Collection<?> c) { 
    return "Unknown Collection";
	}
  
  public static void main(String[] args) { 
      Collection<?>[] collections = {
      new HashSet<String>(),
      new ArrayList<BigInteger>(),
      new HashMap<String, String>().values()
      };
      for (Collection<?> c : collections) System.out.println(classify(c));
    } 
}
```

먼저 위 함수의 결과는 ```Set```, ```List```, ```Map``` 이 차례대로 출력될것 같지만, 실제로는 ```Unknown Collection``` 이 3번 출력이 된다. 왜냐하면 **어느 overloading 함수가 호출될지는 compile 시간에 결정이 되기 때문이다.** 컴파일 시간에 객체들은 모두 ```Collection``` 타입이다. 

**반면,Overriding 된 경우에는 런타임에 해당 객체 타입에 있는 함수가 호출된다.**

다음의 코드를 보자.

``` java
class Wine {
	String name() { return "wine"; }
}

class SparklingWine extends Wine {
	@Override String name() { return "sparkling wine"; }
}

class Champagne extends SparklingWine { 
  @Override String name() { return "champagne"; }
}

public class Overriding {
	public static void main(String[] args) {
		List<Wine> wineList = List.of( new Wine(), new SparklingWine(), new Champagne());
		for (Wine wine : wineList) System.out.println(wine.name()); // 각자의 name 메소드가 호출된다.
	} 
}
```

## Overloading 문제 해결하는 방법

### 1. instanceof를 사용

CollectionClassifier 수정하기위해서, 우리가 기대하는 결과(Set,List,Unknown Collection을 구분해서 출력)를 얻으러면, 다음과 같이 명시적으로 ```instanceof``` 체크를 해야한다.

``` java
public static String classify(Collection<?> c) { 	
  return c instanceof Set ? "Set" : c instanceof List ? "List" : "Unknown Collection"; 
}
```

### 2. 같은 파라미터 개수를 가진 오버로딩을 절대 클라이언트에게 공개하지 않기 

그리고 가변인자를 받는 함수가 있는 경우에는 전혀 오버로딩을 하지 않는것이다. 이를 실현하는 방법은 오버로딩 대신에, 새로운 이름을 가진 함수를 정의하는 것이다.

-  대표적인 예로, ```ObjectOutputStream``` 클래스는 오버로딩 대신 ```writeBoolean(boolean)```, ```writeInt(int)```,```writeLong(long)``` 함수가 구현되어 있다.
-  생성자의 경우, 새로운 이름을 정의할수는 없다. 하지만 정적 팩토리 함수를 사용할 수 있다.

## 오버로딩을 해깔리게 만든 요인

### 1. 제너릭의 도입

``` java
public class SetList {
  public static void main(String[] args) {
    Set<Integer> set = new TreeSet<>(); List<Integer> list = new ArrayList<>();
    for (int i = -3; i < 3; i++) { 
        set.add(i);
        list.add(i); 
    }
    for (int i = 0; i < 3; i++) { 
      set.remove(i); 
      list.remove(i);
    }
    System.out.println(set + " " + list); 
  }
}
```

위 코드의 예상 결과는 set과 list에서 -3~2값이 추가되고 0~2를 가진 값이 삭제 되는 것이다. 

set의 경우는  ```set.remove(i)``` 에서 ```remove(E)``` 함수를 선택하고 여기서 E는 Integer 타입으로 치환된다. 그리고 int 값은 Integer로 boxing이 이루어진다. 결과적으로, 우리가 예상한대로 ```[-3,-2,-1]``` 가 출력된다.

반면 list의 경우에는 예상과 다르게 동작을 한다. ```[-2, 0, 2] ``` 라는 결과가 뜬다. 이유는 ```remove(int i)``` 함수가 호출되어, 0번째, 1번째, 2번째 아이템이 삭제가 되기 때문이다. 이를 해결하기 위해서는 인자의 타입을 Integer로 캐스팅하여 ```remove(E)``` 가 호출되도록 하면된다.

``` java
for (int i = 0; i < 3; i++) {
	set.remove(i);
	list.remove((Integer) i); // or remove(Integer.valueOf(i))
}
```

### 2. 람다의 도입으로, 오버로딩의 혼란이 더욱더 가중됨.

``` java
new Thread(System.out::println).start();
```

``` java
ExecutorService exec = Executors.newCachedThreadPool(); 
exec.submit(System.out::println);
```

첫번째 코드는 컴파일이 되지만, 두번째 코드는 컴파일이 되지 않는다. 이유는 ```submit``` 함수는 ```Callable<T>```(T 타입을 리턴하는 함수) 를 인자로 받는데, ```println``` 함수가 오버로딩 되있어서, 예상과 다르게 컴파일이 되지 않는다.

## 요약

같은 수의 파라미터를 가진 메소드 시그니처를 가진 함수를 오버로딩 하는 것은 지양하는 것이 좋다.