## 뭐가 다른거지? 

우리는 주로 외부와 내부에서 주소값을 공유하는 인스턴스의 관계를 끊어주기 위해서 방어적 복사를 사용한다. (불변 유지)

또한, 외부로 값을 노출시킬 때도 관계를 끊어주기 위해서 복사본을 반환해준다. 

이 때 사용되는 것이 주로 `Unmodifiable Collection`와 `Collection.copyOf`이다. 

주로 `Unmodifiable Collection`을 사용했는 데 인터넷을 보다보면 `Collection.copyOf`을 사용하는 글도 많이 볼 수 있었다. 

과연 무엇이 다르고 언제 써야하는 것일까? 한번 `List`를 기준으로 복사에 대해 처음부터 알아보고자 한다.



## 1. = 

```java
List<Number> numbers = new ArrayList<>();

List<Number> copyNumbers = numbers;
```

`=`의 경우 그저 참조변수를 하나 만들어 주소를 넣어주기 때문에 완전히 동일한 컬렉션이다. 

즉, 컬렉션 주소와 안의 요소 주소가 모두 동일하기 때문에 원본 컬렉션인 `numbers`의 변경에도 영향을 받고 요소인 `Number`의 변경에도 영향을 받게 된다.



```java
List<Number> numbers = new ArrayList<>();
numbers.add(new Number(1));
numbers.add(new Number(2));
numbers.add(new Number(3));

List<Number> copyNumbers = numbers;

numbers.add(new Number(4));
numbers.get(0).setValue(10);

assertThat(copyNumbers).hasSize(4);
assertThat(copyNumbers.get(0).getValue()).isEqualTo(10);
```

![스크린샷 2022-04-27 오후 10.20.04](https://tva1.sinaimg.cn/large/e6c9d24egy1h1olfpd11aj20ct024748.jpg)



## 2. = new ArrayList<>()

```java
List<Number> numbers = new ArrayList<>();

List<Number> newNubmers = new ArrayList<>(numbers);
```

앞써 말했던 방어적 복사이다. 

방어적 복사를 해줌으로써 컬렉션의 주소값이 달라져 원본 컬렉션의 변경에 영향을 받지 않게 된다. 

하지만 중요한 점은 요소의 주소값은 동일하기 때문에 요소의 변경에는 영향을 받게 된다.



```java
List<Number> numbers = new ArrayList<>();
numbers.add(new Number(1));
numbers.add(new Number(2));
numbers.add(new Number(3));

List<Number> copyNumbers = new ArrayList<>(numbers);

numbers.add(new Number(4));
numbers.get(0).setValue(10);

assertThat(copyNumbers).hasSize(3);
assertThat(copyNumbers.get(0).getValue()).isEqualTo(10);
```

![스크린샷 2022-04-27 오후 10.25.59](https://tva1.sinaimg.cn/large/e6c9d24egy1h1ollseuqfj20gh028t8p.jpg)



## 3. .addAll()

```java
List<Number> numbers = new ArrayList<>();

List<Number> copyNumbers = new ArrayList<>();
copyNumbers.addAll(numbers);
```

new ArrayList<>()와 동일하기 때문에 컬렉션의 주소는 달라지지만 요소의 주소는 같다.

그렇기 때문에 인텔리제이에선 친절히 대체할 수 있다고 알려준다!

![스크린샷 2022-04-27 오후 10.28.56](https://tva1.sinaimg.cn/large/e6c9d24egy1h1olova4fzj20ex01p0su.jpg)

주로 복사 보단 List끼리 합쳐주기주기 위해서 사용된다.



## 4. .stream().collect(Collectiors.toList())

```java
List<Number> numbers = new ArrayList<>();

List<Number> copyNumbers = numbers.stream().collect(Collectors.toList());
```

이 또한 `new ArrayList<>()`와 동일하기 때문에 컬렉션의 주소는 달라지지만 요소의 주소는 같다.

마찬가지로 인텔리제이에서 알려주게 된다!



## 5. Collections.unmodifiableList()

```java
List<Number> numbers = new ArrayList<>();

List<Number> copyNumbers = Collections.unmodifiableList(numbers);
```

드디어 나오게 되었다!!

기본적인 내용의 경우 new ArrayList<>()와 동일하기 때문에 컬렉션의 주소는 달라지지만 요소의 주소는 같다.

하지만 이름에서 보아도 불변인 List를 반환할 것 같다. 

실제로도 `copyNumbers`에 add나 remove를 통해 변경하려고 하면 `UnsupportedOperationException`이 발생하게 된다.

```java
List<Number> numbers = new ArrayList<>();
numbers.add(new Number(1));
numbers.add(new Number(2));
numbers.add(new Number(3));

List<Number> copyNumbers = Collections.unmodifiableList(numbers);

copyNumbers.add(new Number(4)); // copyNumbers.remove(1); 
```

![스크린샷 2022-04-27 오후 10.41.40](https://tva1.sinaimg.cn/large/e6c9d24egy1h1om240shwj20ih03twf8.jpg)



### 그럼 이것만 쓰면 되나? 

사실 `unmodifiableList`는 이름과 달리 불변이라고 부를 수가 없다. 

컬렉션을 변경하려고 하면 예외가 발생하여 막혀있지만 원본 컬렉션의 변경이 일어나면 복사한 컬렉션에도 그대로 적용이 되기 때문이다.

```java
List<Number> numbers = new ArrayList<>();
numbers.add(new Number(1));
numbers.add(new Number(2));
numbers.add(new Number(3));

List<Number> copyNumbers = Collections.unmodifiableList(numbers);

numbers.add(new Number(4));
assertThat(copyNumbers).hasSize(3);
```

![스크린샷 2022-04-27 오후 10.46.16](https://tva1.sinaimg.cn/large/e6c9d24egy1h1om71dog6j209l04egls.jpg)

이에 대해 자바 문서에서는 다음과 말합니다.

> **unmodifiableList**
>
> Returns an [unmodifiable view](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collection.html#unmodview) of the specified list. Query operations on the returned list "read through" to the specified list, and attempts to modify the returned list, whether direct or via its iterator, result in an `UnsupportedOperationException`.
>
> The returned list will be serializable if the specified list is serializable. Similarly, the returned list will implement [`RandomAccess`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/RandomAccess.html) if the specified list does.

> **Unmodifiable View Collections**
>
> Note that changes to the backing collection might still be possible, and if they occur, they are visible through the unmodifiable view.



즉, 마치 특정 파일에 대해 읽기 전용 권한으로 공유를 받았다고 생각하면 된다.

 만약 변경 권한이 있는 Collection 즉, 원본 컬렉션이 변경할 경우, 주소가 다르다고 해도 그 변경된 내용을 보게 되는 것이다. 



정리하자면 기본적인 것들은 생성자를 통한 복사와 같지만 복사된 컬렉션에 변경을 하게 되면 `UnsupportedOperationException`이 발생되기 때문에 불가하고 **원본 컬렉션에 대한 변경이 일어난 경우, 영향을 받게 된다.**



## 6. List.copyOf()

```java
List<Number> numbers = new ArrayList<>();

List<Number> copyNumbers = List.copyOf(numbers);
```

그럼 대체 `List.copyOf()`는 무엇일까? 

`List.copyOf()`는 `unmodifiableList`와 비슷하지만 원본 컬렉션의 변경에도 영향을 받지 않는다. 

즉,  컬렉션의 주소는 다르고 요소의 주소는 같으며 복사된 컬렉션에 대한 변경 시도시 `UnsupportedOperationException` 발생하며 원본 컬렉션의 변경에도 영향을 받지 않는다는 것이다.

이로써 복사된 컬렉션의 수정을 막으면서 원본 컬렉션 변경에 대한 영향을 받지 않을 수 있게 되었다.

```java
List<Number> numbers = new ArrayList<>();
numbers.add(new Number(1));
numbers.add(new Number(2));
numbers.add(new Number(3));

List<Number> copyNumbers = List.copyOf(numbers);

numbers.add(new Number(4));
assertThat(copyNumbers).hasSize(3);
```

![스크린샷 2022-04-27 오후 11.10.32](https://tva1.sinaimg.cn/large/e6c9d24egy1h1omw68ipmj20hz028749.jpg)









## 그럼 copyOf()만 사용하면 완전한 불변이겠네?? 

이에 대한 대답은 `No`이다

앞써 계속해서 말했던 것이 있다. 바로 ` 요소의 주소는 같다`이다.

이는 결국 요소의 변경이 일어나면 모두 영향을 받게 된다는 것이다. 

```java
List<Number> numbers = new ArrayList<>();
numbers.add(new Number(1));
numbers.add(new Number(2));
numbers.add(new Number(3));

List<Number> copyNumbers = List.copyOf(numbers);

numbers.get(0).setValue(10);
assertThat(copyNumbers.get(0).getValue()).isEqualTo(10);
```

![스크린샷 2022-04-27 오후 11.12.02](https://tva1.sinaimg.cn/large/e6c9d24egy1h1omxp8848j20ht02cdfr.jpg)



그렇기 때문에 **불변 객체, VO**가 중요하다는 것이다. 

만약 컬렉션의 요소가 불변임을 보장할 수 있게 된다면 copyOf를 통해 완전한 불변이 가능할테니 말이다!

항상 불변을 지키려는 자세를 가지고 개발하도록 하자



## Reference 

- [JavaDocs](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#unmodifiableList(java.util.List))
- [방어적 복사와 Unmodifiable Collection](https://tecoble.techcourse.co.kr/post/2021-04-26-defensive-copy-vs-unmodifiable/)

- [컬렉션의 복사 방법을 정리해봅시다! (unmodifiable view / list)](https://creampuffy.tistory.com/148?category=986887#%EB%A-%--%EC%B-%--%--%ED%-A%B-%EC%A-%--%--%ED%-C%-C%EC%-D%BC%EC%--%--%--%EB%-C%--%ED%--%B-%--%EC%-D%BD%EA%B-%B-%--%EC%A-%--%EC%-A%A-%--%EA%B-%-C%ED%--%-C%EC%-C%BC%EB%A-%-C%--%EA%B-%B-%EC%-C%A-%EB%A-%BC%--%EB%B-%-B%EC%--%--%EB%-B%A-%EA%B-%A-%--%EC%--%-D%EA%B-%--%ED%--%--%EB%A-%B-%--%EB%--%A-%EB%-B%--%EB%-B%A--%EC%--%--%EC%A-%--%--%EA%B-%-C%ED%--%-C%EC%-D%B-%--%EC%-E%--%EB%-A%--%--%EC%--%AC%EB%-E%-C%EC%-D%B-%--%EC%--%--%EC%A-%--%ED%--%A-%--%EA%B-%BD%EC%-A%B-%-C%--%EA%B-%B-%--%EC%--%--%EC%A-%--%EB%--%-C%--%EB%--%B-%EC%-A%A-%EC%-D%--%--%EB%B-%B-%EA%B-%-C%--%EB%--%--%EB%-A%--%--%EA%B-%B-%EC%A-%A--%EA%B-%B-%EB%-F%AC%EB%--%--%--%EB%--%B-%EA%B-%--%--%EC%--%--%EC%A-%--%ED%--%--%EB%A-%A-%--%ED%--%A-%--%EB%--%--%--%EA%B-%-C%ED%--%-C%EC%-D%B-%--%EC%--%--%EC%--%B-%EC%--%-C%--%EC%-B%A-%ED%-C%A-%ED%--%A-%EB%-B%--%EB%-B%A-%---%-E%--UOE%--%EB%B-%-C%EC%--%-D-)