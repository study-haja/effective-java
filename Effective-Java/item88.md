# Item 88 : 방어적으로 READOBJECT 함수를 사용하라.

## 예제코드

``` java
public final class Period { 
  private final Date start; 
  private final Date end; 
  /**
  * @param start the beginning of the period
  * @param end the end of the period; must not precede start * @throws IllegalArgumentException if start is after end
  * @throws NullPointerException if start or end is null
  */
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime()); 
    this.end = new Date(end.getTime()); 
    if (this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException( start + " after " + end);
  }
  public Date start () { 
    return new Date(start.getTime()); 
  }
  public Date end () { 
    return new Date(end.getTime()); 
  }
  public String toString() { 
    return start + " - " + end; 
  }
  ... // Remainder omitted 
}
```

## 직렬화를 이용해서 공격할 수 있는 방법

1. byte array 조작

   ``` java
   public class BogusPeriod {
     // Byte stream couldn't have come from a real Period instance!
     private static final byte[] serializedForm = {
     (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06, 0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8, 0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02, 0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f, 0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74, 0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75, 0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a, (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00, 0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf, 0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03, 0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22, 0x00, 0x78
     };
     
     public static void main(String[] args) {
     	Period p = (Period) deserialize(serializedForm); 
       System.out.println(p); /* Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984. */
     }
     // Returns the object with the specified serialized form 
     static Object deserialize(byte[] sf) {
       try {
         return new ObjectInputStream(
         new ByteArrayInputStream(sf)).readObject();
       } catch (IOException | ClassNotFoundException e) {
         throw new IllegalArgumentException(e); }
      } 
   }
   ```

   위의 문제는 `readObject` 함수를 아래와 같이 수정하면 해결가능하다.

   ``` java
   private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
   	s.defaultReadObject();
   	// Check that our invariants are satisfied if (start.compareTo(end) > 0)
   	throw new InvalidObjectException(start +" after "+ end); 
   }
   ```

   2. private date field를 byte stream에 추가

      ``` java
      public class MutablePeriod { 
          // A period instance 
          public final Period period;
          // period's start field, to which we shouldn't have access 
          public final Date start;
          // period's end field, to which we shouldn't have access public final Date end;
          public MutablePeriod() { 
            try {
              ByteArrayOutputStream bos = new ByteArrayOutputStream();
              ObjectOutputStream out = new ObjectOutputStream(bos);
              // Serialize a valid Period instance 
              out.writeObject(new Period(new Date(), new Date()));
              /*
              * Append rogue "previous object refs" for internal * Date fields in Period. For details, see "Java
              * Object Serialization Specification," Section 6.4. */
              byte[]ref={0x71,0,0x7e,0,5}; //Ref#5 
              bos.write(ref); // The start field
              ref[4] = 4; // Ref # 4
              bos.write(ref); // The end field
              // Deserialize Period and "stolen" Date references 
              ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray())); 			period = (Period) in.readObject();
             start = (Date) in.readObject();
             end = (Date) in.readObject();
            } 
            catch (IOException | ClassNotFoundException e) { 
              throw new AssertionError(e);
            } 
          }
      
          public static void main(String[] args) { 
            MutablePeriod mp = new MutablePeriod(); 
            Period p = mp.period;
            Date pEnd = mp.end;
            // Let's turn back the clock 
            pEnd.setYear(78); 
            System.out.println(p);
            // Bring back the 60s! 
            pEnd.setYear(69);
            System.out.println(p);
           	/*
           		Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978 
           		Wed Nov 22 00:21:29 PST 2017 - Sat Nov 22 00:21:29 PST 1969
           	*/
        	}
      }
      ```

      해결방법은 아래와 같이 `readObject` 함수를 정의한다.

      ``` java
      private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException { 
        s.defaultReadObject();
      	// Defensively copy our mutable components 
        start = new Date(start.getTime());
      	end = new Date(end.getTime());
      	// Check that our invariants are satisfied 
        if (start.compareTo(end) > 0)
      		throw new InvalidObjectException(start +" after "+ end); 
      }
      ```

      ## 요약

      `readObject` 함수를 작성할때는 다음 가이드라인을 따르라.

      - private으로 남아있어야하는 레퍼런스 필드들은 defensive copy하라.
      - 체크가 실패한다면 `InvalidObjectException` 을 던져라
      - 오버라이딩 메소드들을 직간접적으로 호출하지 마라.