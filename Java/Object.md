## Object 클래스

> 자바의 최상위 클래스

Object클래스는 모든 클래스의 최고 조상이기 때문에 Oject클래스의 멤버들은 모든 클래스에서 바로 사용 가능하다. 

<br>

## Object클래스의 멤버

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h0sfmvfrjlj211u0mc796.jpg)

Object클래스는 멤버변수는 없고 오직 11개의 메서드만 가지고 있다. 이 메서드들은 모든 인스턴스가 가져야 할 기본적인 것들이다.

<br>

## equals(Object obj)

> `==`연산 결과 반환 ( 주소 값 비교 ) 
> Overriding 목적 : 물리적으로 다른 메모리에 위치하는 객체여도 논리적으로 동일함을 구현하기 위해

```java
public class Number {
    private final int value;

    public Number(int value) {
        this.value = value;
    }

    public static void main(String[] args) {
        Number number1 = new Number(1);
        Number number2 = new Number(1);

        System.out.println(number1.equals(number2)); // false
    }
}
```

equals()의 기본 동작은 `==`연산이기 때문에 서로 다른 인스턴스를 가리키는 참조변수를 비교하면 false가 반환된다.

이런 경우 아래와 같이 equals()를 오버라이드하여 논리적인 동일성을 갖도록 할 수 있다.



<br>

```java
public class Number {
    private final int value;

    public Number(int value) {
        this.value = value;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof Number) {
            return this.value == ((Number)obj).value;
        }
        return false;
    }

    public static void main(String[] args) {
        Number number1 = new Number(1);
        Number number2 = new Number(1);

        System.out.println(number1.equals(number2)); // true
    }
}
```

만약 equals()의 반환 값이 true일 경우 두 객체가 같은 해시코드값을 가진다는 말이다.



## hashCode()

> 해싱(hashing)기법에 사용되는 해시함수(hash function)을 구현하여 해시코드(hash code)를 반환
>
> Overriding : 서로 다른 메모리에 위치한 객체여도 논리적으로 동일함을 구현하기 위해  

Object클래스에 정의된 hashCode메서드는 객체의 주소값으로 해시코드를 만들어 반환한다. 

32bit JVM에서는 서로 다른 두 객체는 결코 같은 해시코드를 가질 수 없었지만, 64bit JVM에서는 8byte 주소값으로 해시코드(4byte)를 만들기 때문에 해시코드가 중복될 수 있다. 

그래서 앞서 설명했듯이 클래스의 인스턴스변수 값으로 객체의 같고 다름을 판단할 경우 equals메서드 뿐 만 아니라 hashCode메서드도 오버라이딩해줘야 한다. 즉, 같은 객체라면 hashCode메서드를 호출했을 때 반환되는 해시코드도 같아야 한다.

```java
public class Number {
    private final int value;

    public Number(int value) {
        this.value = value;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof Number) {
            return this.value == ((Number)obj).value;
        }
        return false;
    }

    public static void main(String[] args) {
        Number number1 = new Number(1);
        Number number2 = new Number(1);

        System.out.println(number1.equals(number2)); // true
        System.out.println(number1.hashCode());      // 873415566
        System.out.println(number2.hashCode());      // 818403870
    }
}
```

위에서 살펴본 예제를 본다면 equals를 overriding해주어서 논리적으로 같은 객체이지만 hashCode는 다른 값은 반환하고 있다. 그런 경우 만약 해시기법을 사용하는 HashMap, HashSet과 같은 Collection에 저장할 경우 예상과 다르게 작동하는 문제가 발생할 수 있다.



<br>

```java
public static void main(String[] args) {
  	Number number1 = new Number(1);
  	Number number2 = new Number(1);

  	Set<Number> set = new HashSet<>();
  	set.add(number1);
  	set.add(number2);
  	System.out.println(set.size()); // 2
}
```

중복되지 않는 Set에 논리적으로 같은 Number객체를 넣어 Set의 크기가 1이 될거라 예상했지만 결과로는 2가 나오게 된다.

<br>

![img](https://tva1.sinaimg.cn/large/e6c9d24egy1h0suuso7m8j20hm06kglt.jpg)

위와 같은 문제가 발생하는 이유는 hashCode()의 리턴값이 우선 일치하고 equals()의 리턴값이 true여야 논리적으로 같은 객체라고 판단하기 때문이다. 앞의 예제에서는 HashSet에 Number객체를 추가할때도 위의 과정을 거치게 되는 데 hashCode()가 overriding되어 있지 않아 Object클래스의 hashCode()가 사용되어 다른 객체라고 인식하게 되는 것이다.

<br>

```java
public class Number {
    private final int value;

    public Number(int value) {
        this.value = value;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof Number) {
            return this.value == ((Number)obj).value;
        }
        return false;
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }

    public static void main(String[] args) {
        Number number1 = new Number(1);
        Number number2 = new Number(1);

        System.out.println(number1.equals(number2)); // true
        System.out.println(number1.hashCode());      // 32
        System.out.println(number2.hashCode());      // 32
        System.out.println(System.identityHashCode(number1)); // 873415566
        System.out.println(System.identityHashCode(number2)); // 818403870

        Set<Number> set = new HashSet<>();
        set.add(number1);
        set.add(number2);
        System.out.println(set.size());   // 1
    }
}
```

그렇기 때문에 equals를 overriding할때는 hashCode도 overriding해줘야 하는 것이다. hashCode는 `Objects.hash`메서드를 사용하여 간편하게 만들 수 있지만 내부적으로 AutoBoxing이 일어나 성능이 떨어지기 때문에 성능에 민감한 경우에는 직접 재정의해서 사용하는 것이 좋다.

여기서 `System.identityHashCode()`는 Object클래스의 hashCode메서드처럼 객체의 주소값으로 해시코드를 생성한다.



<br>

## toString()

> 인스턴스에 대한 정보를 문자열로 반환

Object클래스의 toString메서드는 기본적으로 아래와 같은 형식을 가지고 있다. 

```java
public String toString() {
  	return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

그대로 사용할 경우 클래스이름에 16진수의 해시코드를 얻게 된다. 그렇기 때문에 인스턴스에 대한 정보를 문자열로 표현해야할 땐 toString을 overriding해서 쓸모 있는 정보를 제공할 수 있다.

```java
public class Number {
    private final int value;

    public Number(int value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Number{" +
            "value=" + value +
            '}';
    }

    public static void main(String[] args) {
        Number number1 = new Number(1);
        Number number2 = new Number(2);

        System.out.println(number1);  // Number{value=1}
        System.out.println(number2);  // Number{value=2}
    }
}
```



<br>

## clone()

> 자신을 복제하여 새로운 인스턴스를 생성 
>
> 복제할 클래스가 Cloneable인터페이스를 구현해야 함.

```java
class Point implements Cloneable {
    int x;
    int y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public String toString() {
        return "x=" + x + ", y=" + y;
    }

    @Override
    public Object clone() {
        try {
            Object clone = super.clone();
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

class PointMain {
    public static void main(String[] args) {
        Point original = new Point(1, 2);
        Point copy = (Point)original.clone();
        System.out.println(original);  // x=1, y=2
        System.out.println(copy);      // x=1, y=2
    }
}
```

Object클래스에 정의된 clone메서드는 단순히 인스턴스변수의 값만 복사하기 때문에 참조타입의 인스턴스 변수가 있는 클래스는 완전한 인스턴스 복제가 이루어지지 않는다. 예를 들어 배열의 경우, 복제된 인스턴스도 같은 배열의 주소를 갖기 때문에 복제된 인스턴스의 작업이 원래의 인스턴스에 영향을 미치게 된다. 이런 경우 clone메서드를 오버라이딩해서 새로운 배열을 생성하고 배열의 내용을 복사하도록 해야 한다.



### 공변 반환타입

```java
@Override
public Point clone() {
  	try {
    	Object clone = super.clone();
    	return (Point)clone;
  	} catch (CloneNotSupportedException e) {
  	  throw new AssertionError();
 	 }
}
```

JDK 1.5부터 **공변 반환타입**이 추가되어 오버라이딩할 시 조상 메서드의 반환타입을 자손 클래스의 타입으로 변경을 허용되었다. 위의 코드에선 clone()의 반환타입을 Object -> Point로 변경한 것이다. return문에서도 Point타입으로 형변환시켜준다.

이를 통해 배열 뿐만 아니라 Vector, ArrayList, HashSet 등 여러 클래스들이 복제가 가능하다. 

```java
ArrayList list = new ArrayList();
ArrayList list2 = (ArrayList)list.clone();
```

<br>

### 얕은 복사와 깊은 복사 

```java
class Point {
    int x;
    int y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public String toString() {
        return "(" + x + ", " + y + ")";
    }
}

class Circle implements Cloneable {
    Point p;
    double r;

    public Circle(Point p, double r) {
        this.p = p;
        this.r = r;
    }

    public Circle shallowCopy() {
        try {
            Circle clone = (Circle)super.clone();
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

    public Circle deepCopy() {
        try {
            Circle clone = (Circle)super.clone();
            clone.p = new Point(this.p.x, this.p.y);
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

    @Override
    public String toString() {
        return "[p=" + p + ", r=" + r + "]";
    }
}

class PointMain {
    public static void main(String[] args) {
        Circle c1 = new Circle(new Point(1, 1), 2.0);
        Circle c2 = c1.shallowCopy();
        Circle c3 = c1.deepCopy();

        System.out.println(c1);  // [p=(1, 1), r=2.0]
        System.out.println(c2);  // [p=(1, 1), r=2.0]
        System.out.println(c3);  // [p=(1, 1), r=2.0]

        c1.p.x = 10;
        c1.p.y = 7;
        System.out.println(c1);  // [p=(10, 7), r=2.0]
        System.out.println(c2);  // [p=(10, 7), r=2.0]
        System.out.println(c3);  // [p=(1, 1), r=2.0]
    }
}
```

![ㅂ](https://tva1.sinaimg.cn/large/e6c9d24egy1h0t089sbdej20up0u0dhb.jpg)얕은 복사는 원본을 변경하면 복사본도 영향을 받는다. 그에 반해 깊은 복사는 원본과 복사본이 서로 다른 객체를 참조하기 때문에 원본의 변경이 복사본에 영향을 미치지 않는다.

위 코드에서 `shallowCopy()`가 Object클래스의 clone메서드를 그대로 호출한 얕은 복사이다. `deepCopy()`는 복제된 객체가 새로운 Point인스턴스를 참조하도록한 깊은 복사이다.



<br>

## getClass()

> 자신이 속한 클래스의 Class객체를 반환

```java
public final class Class implements ... {  // Class 클래스 
		... 
}
```

Class객체는 클래스의 모든 정보를 담고 있으며, 클래스 당 1개만 존재한다. 그리고 클래스 로더(ClassLoader)에 의해서 메모리에 올라갈 때, 자동으로 생성된다.

<br>

### 클래스로더(ClassLoader)

클래스 로더는 실행 시에 필여한 클래스를 **동적**으로 메모리에 로드하는 역할을 한다. 먼저 기존에 생성된 클래스 객체가 메모리(JVM의 Method Area)에 존재하는지 확인하고 없다면 클래스 패스(classpath)에 지정된 경로를 따라서 클래스 파일을 찾는다. (클래스 로더의 순서는 `BootStrap → Extension → Application ClassLoader` 순이다. ) 만약 `Application ClassLoader`가 확인했을 때 찾지 못하면 `ClassNotFoundException`이 발생하고, 찾으면 해당 클래스 파일을 읽어서 Class객체로 변환한다.

정리하자면 파일 형태로 저장되어 있는 클래스(바이트코드)를 읽어서 Class클래스에 정의된 형식으로 변환하는 것이다. 즉, 클래스 파일을 읽어서 사용하기 편한 형태로 저장해 놓은 것이 Class객체이다.



### Class객체 얻는 법

```java
Class cObj = new Card().getClass();  // 생성된 객체로 부터 얻는 방법
Class cObj = Card.class;             // 클래스 리터럴(*.class)로 부터 얻는 방법
Class cObj = Class.forName("Card")   // 클래스 이름으로부터 얻는 방법 
```

위와 같이 Class객체에 대한 참조를 얻을 수 있다. Class객체를 이용하면 클래스에 정의된 멤버의 이름이나 개수 등, 클래스에 대한 모든 정보를 얻을 수 있기 때문에 Class객체를 통해서 객체를 생성하고 메서드를 호출하는 등 보다 동적인 코드를 작성할 수 있다.

```java
Card c = new Card();                // new연산자를 이용해서 객체 생성 
Card c = Card.class.newInstance();  // Class객체를 이용해서 객체 생성
```



## Reference 

- 남궁성. Java의 정석 3판. 도우출판, 2016.
- [JVM에 관하여 - Part 2, ClassLoader](https://tecoble.techcourse.co.kr/post/2021-07-15-jvm-classloader/)

 