# 애플리케이션 조립하기

## 왜 조립까지 신경써야 할까 ?

코드 의존성이 올바른 방향을 가리키게 하기 위해서 조립을 신경써야 한다. <br>
모든 의존성은 안쪽으로, 애플리케이션의 도메인 코드 방향으로 향해야 도메인 코드가 바깥 계층의 변경으로부터 안전하다는 점을 기억하자.

<img width="709" alt="스크린샷 2024-11-17 오후 8 34 26" src="https://github.com/user-attachments/assets/bef9a674-d06b-4355-8b65-21ad47ddf72c">

## 스프링 클래스패스 스캐닝으로 조립하기

스프링 프레임워크를 이용해서 애플리케이션을 조립한 결과물을 애플리케이션 컨텍스트라고 한다. <br>
애플리케이션 컨텍스트는 애플리케이션을 구성하는 모든 객체(자바 용어로는 빈)를 포함한다.

```java
@RequiredArgsConstructor
@Component
class AccountPersistenceAdapter implements LoadAccountPort, UpdateAccountStatePort {
    private final AccountRepository accountRepository;
    private final ActivityRepository activityRepository;
    private final AccountMapper accountMapper;

    @Override
    public Account loadAccount(Accountld accountId, LocalDateTime baselineDate) {
        // ...
    }

    @Override
    public void updateActivities(Account account) {
        // ...
    }
}
```

클래스패스 스캐닝 방식을 이용하면 아주 편리하게 애플리케이션을 조립할 수 있다. 적절한 곳에 ©Component 애너테이션을 붙이고 생성자만 잘 만들어두면 된다.


