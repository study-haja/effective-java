#  Item39 : 명명 패턴보다 애너테이션을 사용하라

전통적으로  도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다. 예컨대 테스트 프레임워크인 JUnit은 버전 3까지 테스트 메소드 이름을 test로 시작하게끔 했다.

하지만 이 방법은 다음과 같이 단점이 많다.

1. 오타가 나면 안된다. ( 실수로 tset~~ 하면 Junit3에서는 해당 테스트 클래스를 무시하게 된다. )
2. 명명 패턴은 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다. ( 메소드가 아닌 클래스로 Test~~ 이라 지었을 때, Junit에 던지면 수행하지 않는다. )
3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다. ( 특정 예외를 던져야만 성공하는 테스트를 해야할 경우 방법이 없다. )



## 애너테이션

위 문제들은 JUnit4의 애너테이션을 도입함으로써 모든걸 해결해 주었다. 다음 애너테이션의 동작 방식을 보자.

* 마커 애너테이션 타입 선언

```java
import java.lang.annotation.*;

/**
	* 테스트 메서드임을 선언하는 애너테이션이다.
	* 매개변수 없는 정적 매소드 전용이다.
	*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

위에 @Test 애너테이션 타입 선언 자체에도 두 가지의 다른 애너테이션이 달려 있다. 위의 애너테이션을 살펴보자면 다음과 같다.

1.  @Retention(RetentionPolicy.RUNTIME) : 메타애너테이션은 @Test가 런타임에도 유지되어야 한다는 표시
2. @Target(ElementType.METHOD) : @Test가 반드시 메서드 선언에서만 사용돼야 한다고 알려주는 표시

> 번외 
>
> 1. 메타 애너테이션 : 애너테이션 선언하는 다른 애너테이션
> 2. 마커 애너테이션 : 아무 매개변수도 없이 단순히 대상에 마킹

즉 위의 @Test는 매개변수 없는 정적 메서드 전용이라 보면 된다. 만약 적절한 애너테이션 처리기 없이 인스턴스 메서드나 매개변수가있는 메서드에 달면  어떻게 될까? 아마 컴파일은 잘 되겠지만, 테스트 도구를 실행할 때 문제가 나타나게 된다.

다음 코드는 @Test 애너테이션을 실제 적용한 모습이다. 

* 마커 애너테이션을 사용한 프로그램 예

```java
public class Sample {
  @Test public static void m1() { } // 성공
  public static void m2() { }
  @Test public static void m3() { // 실패
  	throw new RuntimeException("실패");
  }
  public static void m4() { }
  @Test public void m5() { } // 잘못 사용한 예 : 정적 메소드가 아님
  public static void m6() { } 
  @Test public static void m7() { // 실패
  	throw new RuntimeException("실패");
  }
	public static void m8() { }
}
```

Sample 클래스를 보면 마킹 애너테이션이 붙은 메소드는 m1,  m3, m5,  m7 이다. m3, m7 메소드는 예외를 던지고 m1, m5는 그렇지 않다.그리고 m5는 인스턴스 메소드이므로 @Test를 잘못 사용했다. 

즉 총 4개의 테스트 메서드 중 1개는 성공, 2개는 실패, 1개는 잘못 사용했다. 그리고 나머지 메소드들은 테스트 도구가 무시할 것이다.

@Test 애너테이션이 Sample 클래스의 의미에 직접적인 영향을 주지는 않는다. 그저 어노테이션에 관심 있는 프로그램에게 추가 정보를 제공할 뿐이다. 더 넓게 이야기를 하자면, 대상 코드의 의미는 그대로 둔 채 그 애너테이션에 관심 있는 도구에서 특별히 처리할 기회를 준다. 다음 예를 보자.

* 마커 애너테이션을 처리하는 프로그램

```java
import java.lang.reflect.*;

public class RunTests {
  public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) { // @Test를 찾는 부분
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) { // 리플랙션은 InvocationTargetException을
                    Throwable exc = wrappedExc.getCause();			 // 감싸 다시 던짐
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m); // InvocationTargetException외의 예외는 잘못 사용
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```

이렇게 리플렉션을 사용하여, @Test 애너테이션이 달린 메소드를 찾고, 원래 예외에 담긴 실패 정보를 추출하여 출력한다.

이번에는 특정 예외를 던져야만 성공하는 테스트를 지원하도록 해보자. 다음 새로운 애너테이션 타입의 예이다.

* 매개변수 하나를 받는 애너테이션 타입

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable> value();
}
```

이 예제는 애너테이션은 "Throwable을 확장한 클래스의 Class 객체" 라는 뜻의 매개변수를 사용하라는 의미를 가지고 있다. 따라서 모든 예외 타입을 다 수용한다는 뜻이다. 이 예제를 통해 실제로 활용하는 코드를 보자.

* 매개변수 하나짜리 애너테이션을 사용한 프로그램

```java
public class Sample2 {
  
	@ExceptionTest(ArithmeticException.class)
  public static void m1() { // 성공
    int i = 0;
    i = i / i;
  }
  
  @ExceptionTest(ArithmeticException.class)
  public static void m2() { // 실패 (다른 예외 발생)
		int a = new int[0];
    int i = a[1];
  }
  
  @ExceptionTest(ArithmeticException.class)
  public static void m3() { // 실패 (예외 발생 x)
  }
}
```

* 테스트 러너 수정 코드

```java
import java.lang.reflect.*;

public class RunTests {
  public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedExc) { 
                    Throwable exc = wrappedExc.getCause();
                  	Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
                  	if(excType.isInstance(exc)) {
                      passed++;
                    } elase {
	                    System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                        m, excType.getName(), exc);
                    }
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }
    }
}
```

이 코드는 애너테이션 매개변수의 값을 추출하여 테스트 메소드가 올바른 예외를 던지는지 확인하는 데 사용하는 코드이다. 형변환 코드가 없으니 ClassCastException 걱정은 없다. 즉 테스트 프로그램이 문제없이 컴파일되면 애너테이션 매개변수가 가리키는 예외가 올바른 타입이라는 뜻이다.

단, 해당 예외의 클래스 파일이 컴파일타임에는 존재했으나 런타임에는 존재하지 않을 수는 있다. 이런 경우라면 테스트 러너가 TypeNotPresentException을 던질 것이다.

이 예외에 한걸음 더 나아가 여러개의 예외를 명시하고 그중 하나라도 예외에 들면 성공하게 만드는 테스트 애너테이션을 만들어 보자.

* 배열 매개변수를 받는 애너테이션 타입

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable>[] value();
}
```

배열 매개변수를 받는 애너테이션은 앞에 배열만 붙히면 끝이다. 그리고 앞서 사용한 @ExceptionTest들도 수정할 필요가 없어 아주 유연하다. 다음 예를 보자.

* 배열 매개변수를 받는 애너테이션을 사용하는 코드

```java
@ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
public static void doublyBad() {
	List<String> list = new ArrayList<>();
  list.addAll(5, null);
}
```

* 테스트 러너를 수정 코드

```java
import java.lang.reflect.*;

public class RunTests {
  public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedExc) { 
                    Throwable exc = wrappedExc.getCause();
                  	int oldPassed = passed; 
                  	Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
                  	for (Class<? extends Throwable> excType : excTypes) {
                      if(excType.isInstance(exc)) { // 검증
                      	passed++;
                        break;
                    	}
                    }
                  	if (passed == oldPassed) { // 이전과 이후가 다르면
                      System.out.printf("테스트 %s 실패 : %s, %n", m, exc);
                    }
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }
    }
}
```



## @Repeatable

자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다. 바로 배열 매개변수를 사용하는 대신 @Repeatable 매타 애너테이션을 사용하는 방법이다. 

@Repeatable을 단 애너테이션은 하나의 프로그램 요소에 여러 번 달 수 있다. 하지만 주의점이 있다.

1. @Repeatable을 단 애너테이션을 반환하는 컨테이너 애너테이션을 하나 더 정의 -> @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달.
2. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메소드를 정의
3. 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)를 명시

만약 이 주의점을 지키지 않으면 컴파일이 되지 않을 것이다. 다음 예를 보자

* @Repeatable를 사용한 애너테이션 코드

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  	Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}
```

* 반복 가능 애너테이션을 두번  단 코드

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { ... }
```

반복 가능 애너테이션은 처리할 때도 주의를 요한다. 반복가능 애너테이션을 여러개  달면 하나만 달았을  때와 구분하기 위해 해당 컨테이너 애너테이션타입이  적용된다.

즉 getAnnotationByType 메소드는 이  둘을 구분하지 않아서 반복 가능 애너테이션과 그 컨테이너 애너테이션을 모두 가지고 오지만, isAnnotationPresent 매소드는 둘을 명확히 구분한다. 따라서 다음과 같이 코드를 구성해야한다.

* 반복 가능 애너테이션 다루기

```java
if (m.isAnnotationPresent(ExceptionTest.class) 
    || m.isAnnotationPresent(ExceptionTestContainer.class)) { // 둘을 명확히 구분하기 때문에 이렇게 따로 확인해야 함
  tests++;
}
try {
	m.invoke(null);
	System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
} catch (Throwable wrappedExc) { 
	Throwable exc = wrappedExc.getCause();
	int oldPassed = passed; 
	ExceptionTest[] excTests = m.getAnnotationByType(ExceptionTest.class); // 구분하지 않아 이렇게 사용 됌
	for (ExceptionTest excTest : excTests) {
		if(excTest.isInstance(exc)) {
			passed++;
			break;
    }
  }
  if (passed == oldPassed) {
    System.out.printf("테스트 %s 실패 : %s, %n", m, exc);
	}
}
```

이렇게 반복 가능 애너테이션을 사용해 하나의 프로그램 요소에 같은 애너테이션을 여러 번 달 때의 코드 가독성을 높여 보았다. 하지만 이 방법은 애너테이션을 선언하는 부분에서 코드 양이 늘어나며, 특히 처리 코드가 복잡해져 오류가 날 가능성이 커짐을 명심해야한다.



## 정리

이번 아이템의 테스트 프레임워크는 아주 간단하지만 애너테이션이 명명 패턴보다 낫다는 점은 확실히 보여준다. 물론 테스트는 애너테이션으로 할수 있는 일이 극히 일부일 뿐이지만, 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.

즉 도구 제작자를 제외하고는, 일반 프로그래머가 애너테이션 타입을 직접 정의할 일은 거의 없다. 하지만 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다. 이러면 해당 도구가 제공하는 진단 정보의 품질을 높여줄 것이다.