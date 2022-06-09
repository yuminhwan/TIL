## @Configuration 

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
    
}
```

해당 설정정보를 스프링 컨테이너에 올리게 된다면 memberRepository() 메서드가 3번 호출되는 것이 아닐까?



```java
@DisplayName("@Configuration 싱글톤")
@Test
void ConfigurationTest() {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
    OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
    MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

    MemberRepository memberRepository1 = memberService.getMemberRepository();
    MemberRepository memberRepository2 = orderService.getMemberRepository();

    System.out.println("memberService -> memberRepository = " + memberRepository1);
    System.out.println("orderService -> memberRepository = " + memberRepository2);
    System.out.println("memberRepository = " + memberRepository);

    assertThat(memberRepository1).isSameAs(memberRepository);
    assertThat(memberRepository2).isSameAs(memberRepository);
}
```

![image-20220609193225083](https://tva1.sinaimg.cn/large/e6c9d24egy1h3268fuy35j211e05hwfs.jpg)

테스트 해본 결과 모두 같은 MemberRepository가 반환되었다. 어떻게 된 일 일까?



## CGLIB 

> 클래스의 바이트코드를 조작하여 Proxy 객체를 생성해주는 라이브러리
> https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html

```java
@DisplayName("@Configuration 바이트 코드")
@Test
void configurationDeep() {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean.getClass());
    // bean = class com.example.core.AppConfig$$EnhancerBySpringCGLIB$$3f4dfc3a
}
```

![image-20220609193955901](../../../../../Library/Application%20Support/typora-user-images/image-20220609193955901.png)

> 사실 AnnotationConfigApplicationContext에 파라미터로 넘긴 값은 스프링 빈으로 등록된다. 
>
> 그렇기 때문에 AppConfig도 스프링 빈이 된다.

우리가 구현한 AppConfig 클래스 이름이 나오는 것이 아니라 CGLIB가 붙어서 출력되는 것을 확인할 수 있다. 

이것은 내가 만든 클래스가 아니라 스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, **그 다른 클래스를 스프링 빈으로 등록한 것이다.**



![스크린샷 2022-06-09 오후 7.46.46](https://tva1.sinaimg.cn/large/e6c9d24egy1h326ndf7avj21kl0u0tas.jpg)

@Bean이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어지는 것이다. 

그렇기 때문에 싱글톤을 보장할 수 있게 되는 것이다. 

MemberRepository를 기준으로 설명하자면 MemberRepository가 스프링 컨테이너에 없다면 기존 로직을 호출(MemberRepository를 new로 생성)하여 스프링 컨테이너에 등록하고 반환한다.

 스프링 컨테이너에 존재한다면 스프링 컨테이너에서 찾아서 반환해주게 된다.



## @Configuration이 없다면 ?

@Configuration이 없이 @Bean만 적용했을 땐 어떻게 될까? 

![image-20220609195114519](https://tva1.sinaimg.cn/large/e6c9d24egy1h326rz7qhij20hs03k0t1.jpg)

AppConfig가 CGLIB 기술 없이 순수한 AppConfig 즉, 우리가 구현한 클래스가 스프링 빈에 등록되게 된다. 



![image-20220609195231317](https://tva1.sinaimg.cn/large/e6c9d24egy1h326tbky0gj211l0fyjvf.jpg)하지만 해당 코드를 보면 각 다른 MemberRepository 인스턴스를 반환되게 된다. 

즉, `@Bean`만 사용해도 스프링 빈으로 등록은 되지만 **싱글톤을 보장하지 못한다.** memberRepository() 처럼 의존관계 주입이 필요해서 메서드를 직접호출할 때 싱글톤을 보장하지 못하는 것이다.



## Reference 

- [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/dashboard)