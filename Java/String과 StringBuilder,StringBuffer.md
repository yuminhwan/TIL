# String과 StringBuilder,StringBuffer

## String

String은 Java에서 기본 자료형으로 제공하지만, **Reference-type** 이다.

String은 Reference-type이지만 다른 클래스들과는 다르게 **String Constant Pool**을 사용하는 특이한 녀석이다.

## **String Constant Pool**

Java에서 String을 선언하는 방법은  두 가지가 있다.

```java
// 1. 리터럴 방식 
String a = "hello";

// 2. new 방식 ( Reference 방식 )
String b = new String("hello"); 
```

리터럴 방식의 경우 heap 메모리 영역의  String Constant Pool에서 할당되는 반면 new 방식은 메모리의  heap에 바로 할당된다.

![ Yh.jpg](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hvlqt52oj20u00v9dhd.jpg)

## 문자열 비교

```java
public class App {
    public static void main(String[] args) {
        String name1 = "hwan";
        String name2 = new String("hwan");

        System.out.println(name1.equals(name2)); // true
        System.out.println(name1 == name2); // false
    }
}
```

String의 eqauls 메서드의 경우 단순하게 문자열의 값을 비교하기 때문에 true를 반환하게 되고 `==`연산의 경우 객체의 주소값을 비교하기 때문에 false가 반환되게 된다. 

### 왜??

문자열 리터럴의 경우 내부적으로 String의 `intern`메서드가 호출하게 된다. 해당 메서드는 해당 문자열이 String Constant Pool에 이미 있는 경우에는 그 문자열의 주소값을 반환하고 없다면 새로 집어넣고 그 주소값을 반환하기 때문에 **new 연산자를 통해 Heap 영역에 생성된 String**과 **리터럴을 통해 String Constant Pool 영역에 위치한 String**의 주소값은 같을 수 없는 것이다.

## String은 불변이다.

```java
String c = "abc";
String d = "abc"; // c == d
```

String Constant Pool에 할당된 객체는 재활용되기 때문에 String은 **불변의 성질**을 가져야 한다. 만약 String이 가변일 경우 위의 예시에서 변수 c에 할당된 값을 “ABC”로 바꿀 경우 변수 d에 할당된 값도 변경될 수 있기 때문이다.

그렇기에 변수 c에 할당된 값을 “ABC”로 바꿀 경우 String Constant Pool에 “ABC”가 생성되어 지고 이를 참조하게 된다. 

또한, 불변하기 때문에 멀티 스레드 환경에서 공유하여 사용할 수 있게 된다.

## String 최적화

불변인 String을 가지고 `+`연산을 할 시 기존 문자열에 새롭게 문자열이 추가되는 것이 아닌, 새로운 문자열 객체를 만들고 그 객체를 참조하도록 한다. 

따라서 기존 문자열은 래퍼런스 참조가 사라져 가비지 컬렉터의 수집 대상이 되어버린다.

### StringBuilder와 StringBuffer

StringBuilder와 StringBuffer의 경우 String과 달리 가변의 속성을 가지고 있다는 점이 큰 차이이다.

이 둘은 문자열을 한번 만들고 연산이 필요할 때마다 크기를 변경해가며 문자열을 변경하기 때문에 새롭게 객체를 만드는 String보다 더 빠르다.

그럼 **둘의 차이는 무엇일까?** 

바로 thread-safe 즉, 동기화에 있다. StringBuilder의 경우 동기화를 보장하지 않아 thread-safe하지 않지만, StringBuffer의 경우 동기화를 보장해 thread-safe하다.

하지만 동기화를 보장하기 때문에 속도의 경우 StringBuilder에 비해 느리다는 단점이 있다. ( 그래도 String 보다는 빠르다 ! )

### StringBuilder를 통한 최적화

```java
String result = "";

for (int i = 0; i < 100; i++) {
    result += i;
}
```

해당 코드는 String이 `+`연산을 할 때마다 객체를 생성할 것 같지만 JDK 1.5 버전부터는 StringBuilder를 사용하도록 변경 되었다.  ( 11버전 부터는 StringConcatFactory.makeConcatWithConstants를 사용한다고 한다. )

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hvlrji5aj21t60u0n25.jpg)

**그렇다면 이런식으로 되는 건가..?**

```java
StringBuilder result = new StringBuilder();

for (int i = 0; i < 100; i++) {
    result.append(i);
}
```

JVM에서 위의 코드처럼 바뀌어서 동작할 것이라고 생각할 수 있겠지만 **아니다** 

**실제로는 ...**

```java
String result = "";

for (int i = 0; i < 100; i++) {
    result = new StringBuilder(text).append(i).toString();
}
```

반복을 진행할 때마다 매번 StringBuilder가 생성되기 때문에 상대적으로 느릴 수 밖에 없다.

그러니 반복문일 경우 직접 StringBuilder를 적용하는 것이 좋다.

### final을 이용한 최적화

또 다른 방법으로 final을 사용하여 최적화 하는 방법이 있다.

```java
final String a = "a";
final String b = "b";
final String c = "c";

String result = a + b + c;
```

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hvlvop62j21gs0u0aes.jpg)

final 키워드가 사용된 경우에는 StringBuilder를 이용하지 않고 컴파일 과정에서 하나의 문자열로 변경된다.

**이 경우엔?**

```java
final String a = "a";
String b = "b";
final String c = "c";

String result = a + b + c;
```

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hvltb0b7j21g60u0te3.jpg)

아까와는 다르게 컴파일 과정에서 StringBuilder을 사용하도록 변경된다.

**그럼 이건?**

```java
final String a = "a";
final String c = "c";

String result = a + "b" + c;
```

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hvlyn5loj21g40sm429.jpg)

변수가 아닌 리터럴 자체를 연산하게 되면 첫 번째와 마찬가지로 컴파일 과정에서 하나의 문자열로 변경된다.

## Reference

- [[우테코 강의] 문자와 문자열 ( String , StringBuilder , StringBuffer )](https://pparksean.tistory.com/115)
- [String과 StringBuilder](https://toneyparky.tistory.com/28)
- [자바의 String 객체와 String 리터럴](https://madplay.github.io/post/java-string-literal-vs-string-object)