## AnonymousAuthenticationFilter

> 익명 사용자의 요청에 대해 처리해주는 필터 

![스크린샷 2022-05-20 오후 9.58.37](https://tva1.sinaimg.cn/large/e6c9d24egy1h2f62jc0fyj21g80jsmzw.jpg)

인증을 하지 않은 요청인 경우 인증 객체를 익명 권한이 들어가 있는 객체를 만들어 SecurityContextHolder에 넣어주는 역할을 한다.



### 생성 

![image-20220520220139526](https://tva1.sinaimg.cn/large/e6c9d24egy1h2f65ik5aaj21hg0nejuz.jpg)

![스크린샷 2022-05-20 오후 10.01.12](https://tva1.sinaimg.cn/large/e6c9d24egy1h2f654h2uuj21b80g476z.jpg)

![스크린샷 2022-05-20 오후 10.03.41](https://tva1.sinaimg.cn/large/e6c9d24egy1h2f67pdbqxj20nm09it9a.jpg)

- `AnonymousConfigurer`에서 init 메서드가 호출되면서 생성된다. 

- 생성될 때 기본으로 세팅되는 값들이 존재한다. 

	- **key** : UUID random 값
	- **principal** : anonymousUser
	- **authorities** : ROLE_ANONYMOUS

	- 이는 HttpSecurity클래스를 통해 설정 해줄 수 있다. 

		```java
		http.anonymous()
		    .key("My_Key")
		    .principal("yuminhwan")
		    .authorities("ROLE_HWAN")
		```

		![image-20220520220835243](https://tva1.sinaimg.cn/large/e6c9d24egy1h2f6cq6uekj20to07q75z.jpg)

- Key는 `AnonymousAuthenticationFilter`로 생성된 token인지 확인을 하기 위한 용도로 사용된다.



### 역할 

SecurityContext에 Authentication이 null일 경우 `AnonymousAuthenticationToken`을 주입하는 역할을 한다. 

![image-20220520221626722](https://tva1.sinaimg.cn/large/e6c9d24egy1h2f6kx4xg6j216s0u0jwa.jpg)

![image-20220520221705264](https://tva1.sinaimg.cn/large/e6c9d24egy1h2f6lkkvu3j217k09edhj.jpg)

`AnonymousAuthenticationToken`의 경우 `createAuthentication` 메서드를 통해 생성된다.

이렇게 생성하여 사용하는 이유는 SercurityFilter에서 Authentication이 null인지 확인하는 부분을 없애고, null을 대변하는 객체인 `AnonymousAuthenticationToken`을 이용해서 null인지 확인한다. 

![스크린샷 2022-05-20 오후 10.25.12](https://tva1.sinaimg.cn/large/e6c9d24egy1h2f6u2h4s6j216o06sjsd.jpg)

![image-20220520222256527](https://tva1.sinaimg.cn/large/e6c9d24egy1h2f6ro5l4yj20xg0ao75l.jpg)

실제로 `AuthenticationTrustResolverImpl`의 `isAnonymous`메서드를 보게 되면 `Authentication`이 `AnonymousAuthenticationToken`인지 확인하는 부분이 존재한다. 

이를 **Null Object Pattern** 이라고 한다.



## Reference 

- https://ncucu.me/128?category=796566
- https://kangwoojin.github.io/programing/spring-security-basic-anonymous-authentication-filter/