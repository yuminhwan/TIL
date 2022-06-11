## 조회 빈이 2개 이상 일 때 

```java
public interface DiscountPolicy {
	...
}

@Component
public class FixDiscountPolicy implements DiscountPolicy {
    ...
}

@Component
public class RateDiscountPolicy implements DiscountPolicy {
    ...
}
```

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
    ...
}
```

위와 같은 상황에서 실행하면 `NoUniqueBeanDefinitionException` 오류가 발생하게 된다. 

`@Autowired`는 타입으로 조회하기 때문이다. 

충돌난다고 하위타입으로 지정하여 DIP를 위배하고 유연성을 포기하면서까지 하고싶진 않다.

어떻게 해결할까? 



### 1. @Autowired 필드명 매핑 

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    
    @Autowired // final 키워드 없다고 가정 (필드 주입)
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy rateDiscountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = rateDiscountPolicy;
    }
    ...
}
```

`@Autowired`의 경우 타입 매칭을 시도 한 뒤, 여러 빈이 있다면 이름, 파라미터 이름으로 빈 이름을 추가 매칭하게 된다.



### 2. @Qualifier 사용 

```java
@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {
    ...
}

@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {
    ...
}

// orderServiceImpl.java
public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```

`@Qualifier`는 추가 구분자를 붙여주는 방식이다. 즉, 주입 시 추가적인 방법을 제공해주는 것일뿐 빈 이름을 변경하는 것은 아니다.

수정자 자동 주입에서도 사용이 가능하다.

만약 `@Qualifier`로 주입할 때 지정해준 이름으로 찾지 못하면 어떻게 될까? 

그렇게 된다면 해당 이름의 스프링 빈을 추가로 찾게 된다.



### 3. @Primary 사용 

```java
@Component
public class FixDiscountPolicy implements DiscountPolicy {
    ...
}

@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {
    ...
}

// orderServiceImpl.java
public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```

`@Primary` 는 우선순위를 정하는 방식이다. 주입 시 여러 빈이 매칭되게 되면 `@Primary`가 우선권을 가지게 된다.



## @Qualifier? @Primary?

그렇다면 둘 중 어느 것을 사용해야 더 좋을까? 

`@Qualifier`는 주입 받을 때 마다 `@Qualifier`로 지정해줘야하는 단점이 있다. 

그렇기 때문에 다음과 예처럼 사용해주는 것이 좋다. 

코드에서 자주 사용하는 메인 데이터베이스의 커넥션을 획득하는 스프링 빈과 특별한 기능으로 가끔 사용하는 서브 데이터베이스의 커넥션을 획득하는 스프링 빈이 있을 때, 메인 데이터베이스의 커넥션을 획득하는 스프링 빈을 `@Primary`로 지정하여 `@Qualifier` 지정없이 사용해주고, 서브 데이터베이스 커넥션 빈을 획득할 때는 `@Qualifier`로 지정하여 명시적으로 획득하는 방식을 사용해준다. 

물론 메인 데이터베이스의 스프링 빈도 `@Qualifier`로 지정해줘도 상관없다. 



### 우선순위 

스프링은 자동보다는 수동이, 넓은 범위의 선택권보다는 좁은 범위의 선택권이 우선순위가 높다. 

그렇기 때문에 `@Primary`보다 `@Qualifier`가 우선권이 높다.



## Reference 

- [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/dashboard)

