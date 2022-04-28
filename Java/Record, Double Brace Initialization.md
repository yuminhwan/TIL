## Record

자바 14부터 지원하며 class, enum, interface와 같은 타입 선언이다.

record는 **불변 데이터 집합**이며 명목상 튜플이다.

```java
public record Point(int x, int y) {
}
```



record를 만들면 다음과 같은 내용이 처리된다. 

- private final 필드로 선언된다.
- public으로 읽기 접근 메서드가 같은 이름과 타입으로 만들어 진다. 
 - 즉, public int x(), public int y()
- 초기화하는 public 생성자도 만들어 진다.
- equals, hashCode, toString 의 구현도 만들어진다.

```java
public record Point(int x, int y) {

    public static void main(String[] args) {
        Point point = new Point(1, 2);
        System.out.println("x = " + point.x() + ", y = " + point.y());  
        // x = 1, y = 2
        System.out.println(point.equals(new Point(1, 2)));    // true 
        System.out.println(point.hashCode());    // 33
        System.out.println(point);              // Point[x=1, y=2]
    }
}
```



## Double Brace Initialization

```java
public class Main {
    public static void main(String[] args) {
        List<Integer> numbers1 = new ArrayList<>();
        numbers1.add(1);
        numbers1.add(2);

        List<Integer> numbers2 = new ArrayList<>() {{
            add(1);
            add(2);
        }};

        System.out.println(numbers1); // [1, 2]
        System.out.println(numbers2); // [1, 2]

        System.out.println(numbers1.getClass());  // class java.util.ArrayList
        System.out.println(numbers2.getClass());  // class org.prgrms.kdt.Main$1
    }
}
```

인스턴스 생성과 함께 초기화가 가능하지만 지양해야 할 안티 패턴이다. 

위의 결과를 보면 class가 사실 익명 클래스로 넘어온다는 것을 확인할 수 있다. 

이러한 까닭에 double brace initialization을 하면, JVM이 새로운 클래스 파일을 읽어야 하는 부담도 생기고, 클래스 파일이 늘어남에 따라 저장 공간에 부담이 생기게 된다. 또한, 바깥 인스턴스에 대한 숨겨진 참조를 가지기 때문에 메모리 누수를 일으킬 수 있다.