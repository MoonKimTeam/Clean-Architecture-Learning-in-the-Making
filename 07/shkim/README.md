# 아키텍처 요소 테스트하기

## 단위 테스트로 도메인 엔티티 테스트하기

Account의 상태를 과거 특정 시점의 잔고와 그 이후의 입출금 내역으로 구성되어 있다. <br>
withdraw() 메서드가 기대한대로 동작하는지 검증해보자

```java
class AccountTest {

    @Test
    void withdrawalSucceeds() {
        AccountId accountId = new AccountId(1L);
        Account account = defaultAccount()
                .withAccountId(accountId)
                .withBaselineBalance(Money.of(555L))
                .withActivityWindow(new ActivityWindow(
                        defaultActivity()
                                .withTargetAccount(accountId)
                                .withMoney(Money.of(999L)).build(),
                        defaultActivity()
                                .withTargetAccount(accountId)
                                .withMoney(Money.of(1L)).build()))
                .build();
        boolean success = account.withdraw(Money.of(555L), new AccountId(99L));
        assertThat(success).isTrue();
        assertThat(account.getActivityWindow().getActivitiesO).hasSize(3);
        assertThat(account.calculateBalance()).isEqualTo(Money.of(1000L));
    }
}
```

Account를 인스턴스화하고 withdraw() 메서드를 호출해서 출금을 성공했는지 검증하고, Account 객체의 상태에 대해 기대되는 부수효과들이 잘 일어났는지 확인하는 단순한 단위테스트다.

## 단위 테스트로 유스케이스 테스트하기

SendMoneyService의 테스트를 살펴보자. <br>
SendMoney 유스케이스는 출금 계좌의 잔고가 다른 트랜잭션에 의해 변경되지 않도록 락을 건다. <br>
출금 계좌에서 돈이 출금되고 나면 똑같이 입금 계좌에 락을 걸고 돈을 입금시킨다. 그러고 나서 두 계좌에서 모두 락을 해제한다.

```java
class SendMoneyServiceTest {

    @Test
    void transactionSucceeds() {
        Account sourceAccount = givenSourceAccount();
        Account targetAccount = givenTargetAccount();
        givenWithdrawalWiUSucceed(sourceAccount);
        givenDepositWiUSucceed(targetAccount);
        Money money = Money.of(500L);
        SendMoneyCommand command = new SendMoneyCommand(
                sourceAccount.getId(),
                targetAccount.getId(),
                money);
        boolean success = SendMoneyService.sendMoney(command);
        assertThat(success).isTrue();
        Accountld sourceAccountId = sourceAccount.getId();
        Accountld targetAccountId = targetAccount.getId();
        then(accountLock).should().lockAccount(eq(sourceAccountId));
        then(sourceAccount).should().withdraw(eq(money), eq(targetAccountId));
        then(accountLock).should().releaseAccount(eq(sourceAccountId));
        then(accountLock).should().lockAccount(eq(targetAccountId));
        then(targetAccount).should().deposit(eq(money), eq(sourceAccountId));
        then(accountLock).should().releaseAccount(eq(targetAccountId));
        thenAccountsHaveBeenUpdated(sourceAccountId, targetAccountId);
    }
}
```

테스트 중인 유스케이스 서비스는 상태가 없기때문에 then 섹션에서 특정 상태를 검증할 수 없다. <br>
대신 테스트는 서비스가 의존 대상의 특정 메서드와 상호작용했는지 여부를 검증한다.

## 통합 테스트로 웹 어댑터 테스트하기

웹 어댑터는 JSON 문자열 등의 형태로 HTTP를 통해 입력을 받고, 입력에 대한 유효성 검증을 하고, 유스케이스에서 사용할 수 있는 포맷으로 매핑하고, 유스케이스에 전달한다. <br>
그러고 나서 다시 유스케이스의 결과를 JSON으로 매핑하고 HTTP 응답을 통해 클라이언트에 반환했다.

```java
@WebMvcTest(controllers = SendMoneyController.class)
class SendMoneyControllerTest {
    
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private SendMoneyUseCase SendMoneyUseCase;

    @Test
    void testSendMoney() throws Exception {
        mockMvc.perform(
            post("accounts/send/{sourceAccountId}/{targetAccountId}/{amnount}",
                    41L, 42L, 500)
                    .header("Content-Type", "application/json"))
                .andExpect(status().isOk());
        
        then(sendMoneyUseCase).should()
            .sendMoney(eq(new SendMoneyCommand(
                new AccountId(41L),
                new AccountId(42L),
                Money.of(500L))));
    }
}
```

이 테스트에서는 하나의 웹 컨트롤러 클래스만 테스트한 것처럼 보이지만, 사실 보이지 않는 곳에서 더 많은 일들이 벌어지고 있다. <br>
@WebMvcTest 애너테이션은 스프링이 특정 요청 경로, 자바와 JSON 간의 매핑, HTTP 입력 검증 등에 필요한 전체 객체 네트워크를 인스턴스화하도록 만든다. <br>
그리고 테스트에서는 웹 컨트롤러가 이 네트워크의 일부로서 잘 동작하는지 검증한다.


## 통합 테스트로 영속성 어댑터 테스트하기

영속성 어댑터의 테스트에는 단위 테스트보다는 통합 테스트를 적용하는 것이 합리적이다. <br>
단순히 어댑터의 로직만 검증하고 싶은 게 아니라 데이터베이스 매핑도 검증하고 싶기 때문이다.




