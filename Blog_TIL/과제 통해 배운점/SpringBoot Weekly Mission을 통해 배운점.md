## 1. GitHub 저장소 

- [GitHub](https://github.com/yuminhwan/springboot-basic)
- [1주차 Pull Request](https://github.com/prgrms-be-devcourse/springboot-basic/pull/186)
- 2주차 Pull Request 
- 3주차 Pull Request 

## 1주차 미션 



### 1-1. @CsvSource에서 Enum 사용하기 

커맨드 라인 애플리케이션이다 보니 Enum을 통해 커맨드들을 정의하였는 데 해당 Enum을 @CsvSource에서 사용할 수 있다는 것을 알게되어 적용해보았다. 

```java
@DisplayName("해당하는 메인 메뉴를 반환한다.")
@ParameterizedTest
@CsvSource(value = {"create,CREATE", "read,READ", "update,UPDATE", "delete,DELETE", "exit,EXIT"})
void from_MenuCommand_ReturnMenuType(String command, CustomerMenuType customerMenuType) {
    assertThat(CustomerMenuType.fromMainMenu(command)).isEqualTo(customerMenuType);
}
```



### 1-2. null이 아닌 빈 컬렉션이나 배열을 반환하자 

블랙 리스트에 대한 정보를 Map을 통해 저장하고 있었는 데 이를 반환하는 메서드에서 `return new ArrayList<>(storage.values())`와 같이 새롭게 할당하여 반환하도록 하였다. 

하지만 [null을 반환하지 말고 빈 객체, 또는 Optional을 반환하자](https://kth990303.tistory.com/279) 해당 글을 읽고 나서 굳이 새롭게 할당할 필요없이 바로 빈 컬렉션을 반환하는 것이 더 좋다고 판단하여 밑과 같이 리팩터링해주었다. 

```java
// Before 
public List<User> findBlackUsers() {
    return new ArrayList<>(storage.values());
}

// After 
public List<User> findBlackCustomers() {
    if (storage.isEmpty()) {
        return Collections.emptyList();
    }

    return new ArrayList<>(storage.values());
}
```



### 1-3. Mockito 

```java
class VoucherServiceTest {

    VoucherService voucherService = new VoucherService(new VoucherRepository() {
        @Override
        public Voucher save(Voucher voucher) {
            return voucher;
        }

        @Override
        public List<Voucher> findAll() {
            try {
                return Arrays.asList(new FixedAmountVoucher(UUID.randomUUID(), 10L),
                    new PercentDiscountVoucher(UUID.randomUUID(), 20L));
            } catch (Exception ignored) {
            }
            return null;
        }
    });

		// 테스트 코드 
}
```

`VoucherService`의 단위 테스트를 위해서 직접 `VoucherRepository`를 익명 클래스를 사용하여 테스트를 진행했다. 알고보니 이 방식은 Stub방식이였다. 

`VoucherRepository`를 인터페이스로 정의함으로써 Mock프레임워크를 사용하지 않고도 테스트를 할 수 있다는 것을 알 수 있게 되었다. 

그래도 멘토님이 말씀하신 것처럼 Mock 프레임워크를 사용하는 것이 좋아보여 Mockito를 사용해 리팩터링해주었다. 

```java
@ExtendWith(MockitoExtension.class)
class VoucherServiceTest {

    @Mock
    VoucherRepository voucherRepository;

    @InjectMocks
    VoucherService voucherService;
    
    @DisplayName("FixedAmountVoucerType을 주면 FixedAmountVoucher를 반환한다.")
    @Test
    void create_FixedAmountVoucherType_ReturnFixedAmountVoucher() throws
        WrongDiscountPercentException,
        WrongDiscountAmountException {

        Voucher mockVoucher = new FixedAmountVoucher(UUID.randomUUID(), 10L);
        given(voucherRepository.save(any(FixedAmountVoucher.class))).willReturn(mockVoucher);

        Voucher voucher = voucherService.create(VoucherType.FIXED_AMOUNT, 10L);

        assertThat(voucher).isInstanceOf(FixedAmountVoucher.class);
        assertThat(voucher).extracting("discountAmount")
            .isEqualTo(10L);
        then(voucherRepository).should(times(1)).save(any(Voucher.class));
    }
}
```

하지만 "현재 상황에서 Service가 하는 것이라곤 Repository의 메서드들을 호출하는 것이 전부인 데 무작정 사용하는 것이 맞을까?" 라는 의문점이 들기 시작하였다. 

- [Stub 을 이용한 Service 계층 단위 테스트 하기](https://jojoldu.tistory.com/637?category=1036934)

- [Mockito, 이대로 괜찮은가?](https://tecoble.techcourse.co.kr/post/2020-10-16-is-ok-mockito/)

위 두글을 읽고 **무분별하게 테스트 더블을 활용하지 말고 실체 객체를 사용하되 어려운 경우 테스트 더블을 사용하는 것이 좋다** 라는 결론이 나오게 되었다. 

때에 따라 기준에 맞춰 사용해야 겠다. 



### 1-4. Logback filter

이번 미션에 에러는 파일로 기록하라는 요구사항이 있어 처음에는 Logback에 root에는 콘솔에 찍고 별도의 logger을 설정하여 파일을 생성하면 해결될 것이라고 생각했지만 적용해보니 root는 무시되어 파일로만 기록이 되었다. 

이에 고민하다 Logback의 filter를 사용하면 error 로그만 따로 파일로 기록할 수 있음을 알게 되었고 적용해보았다. 

```xml
<appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>ERROR</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>logs/voucherError-%d{yyyy-MM-dd}.log</fileNamePattern>
    </rollingPolicy>
    <encoder>
        <pattern>${FILE_LOG_PATTERN}</pattern>
    </encoder>
</appender>
```

- [Logback filter (slf4j)](https://mycup.tistory.com/276)



### 1-5. Checked Exception 





























## 끝없는 3주간의 과제

3주간 Spring 주를 마치며 Weekly Mission도 막을 내리게 되었다. 

이전과 달리 한번 구현하고 끝! 이 아니라 점차적으로 확장하다보니 리팩터링해야 할 것이 많아지게 되어 힘들었다. 역시 설계를 처음부터 잘해야 된다! 

요구사항이 크게 어렵지는 않지만 배운 것을 직접 적용하려 보니 시간이 오래걸렸던 것 같다. 

그러다보니 해당 주차에 PR을 날리지 못하고 항상 밀려서 제출했다. 

이에 대해 스펜서의 쓴소리도 들을 수 있게 되었고 다시 한번 다짐을 하게 되는 계기가 되었던 것 같다.

TMI이지만 조급하게 하다보니 생각을 하기 보단 구현을 먼저 한 것과 기록을 세세히 하지 못해 많이 아쉽다. 

다음부터는 구현하기 전에 "왜?"라는 생각을 하고 배운 점은 바로바로 기록하는 습관을 가질 수 있도록 노력해야겠다.