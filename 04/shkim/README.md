# 유스케이스 구현하기

## 유스케이스 둘러보기

일반적으로 유스케이스는 다음과 같은 단계를 따른다.

1. 입력을 받는다.
2. 비즈니스 규칙을 검증한다.
3. 모델 상태를 조작한다.
4. 출력을 반환한다.

유스케이스는 인커밍 어댑터로부터 입력을 받는다. <br>
유스케이스 코드는 도메인 로직에만 신경 써야 하고 입력 유효성 검증으로 오염되면 안 된다. <br>
유스케이스는 비즈니스 규칙을 검증할 책임이 있다. 그리고 도메인 엔티티와 이 책임을 공유한다. <br>
비즈니스 규칙을 충족하면 유스케이스는 입력을 기반으로 어떤 방법으로든 모델의 상태를 변경한다. <br>
일반적으로 도메인 객체의 상태를 바꾸고 영속성 어댑터를 통해 구현된 포트로 이 상태를 전달해서 저장될 수 있게 한다. 유스케이스는 또 다른 아웃고잉 어댑터를 호출할 수도 있다.

<img width="662" alt="스크린샷 2024-11-10 오후 3 07 56" src="https://github.com/user-attachments/assets/1df1946c-9daf-4bfe-b3c2-1ea02ac4d462">

## 입력 유효성 검증

애플리케이션 계층에서 입력 유효성을 검증해야 하는 이유는, 그렇게 하지 않을 경우 애플리케이션 코어의 바깥쪽으로부터 유효하지 않은 입력값을 받게 되고, 모델의 상태를 해칠 수 있기 때문이다. <br>
**유스케이스 클래스가 아니라면 도대체 어디에서 입력 유효성을 검증해야 할까 ?** <br>
입력 모델이 이 문제를 다루도록 해보자.

```java
@Getter
public class SendMoneyCommand {
    private final Accountld sourceAccountld;
    private final Accountld targetAccountld;
    private final Money money;

    public SendMoneyCommand(
            Accountld sourceAccountld,
            Accountld targetAccountld,
            Money money) {
        this.sourceAccountld = sourceAccountld;
        this.targetAccountld = targetAccountld;
        this.money = money;
        requireNonNull(sourceAccountld);
        requireNonNull(targetAccountld);
        requireNonNull(money);
        requireGreaterThan(money, 0);
    }
}
```

SendMoneyCommand의 필드에 final을 지정해 불변 필드로 만들었다. <br>
따라서 일단 생성에 성공하고 나면 상태는 유효하고 이후에 잘못된 상태로 변경할 수 없다는 사실을 보장할 수 있다.

## 유스케이스마다 다른 입력 모델

각기 다른 유스케이스에 동일한 입력 모델을 사용하고 싶은 생각이 들 때가 있다. <br>
**계좌 등록하기**와 **계좌 정보 업데이트하기**라는 두 가지 유스케이스는 거의 똑같은 계좌 상세 정보가 필요할 것이다. <br>
차이점은 ‘계좌 정보 업데이트하기’ 유스케이스는 업데이트할 계좌를 특정하기 위해 계좌 ID 정보를 필요로 하고, ‘계좌 등록하기’ 유스케이스는 계좌를 귀속시킬 소유자의 ID 정보를 필요로 한다는 것이다. <br>
두 유스케이스에서 같은 입력 모델을 공유할 경우 ‘계좌 정보 업데이트하기’에서는 소유자 ID에, ‘계좌 등록하기’에서는 계좌 id에 null 값을 허용해야 한다.

불변 커맨드 객체의 필드에 대해서 null을 유효한 상태로 받아들이는 것은 그 자체로 코드 냄새다. <br>
하지만 더 문제가 되는 부분은 이제 입력 유효성을 어떻게 검증하느냐다. 등록 유스케이스와 업데이트 유스케이스는 서로 다른 유효성 검증 로직이 필요하다.

**각 유스케이스 전용 입력 모델은 유스케이스를 훨씬 명확하게 만들고 다른 유스케이스와의 결합도 제거해서 불필요한 부수효과가 발생하지 않게 한다.**

## 유스케이스마다 다른 출력 모델

입력과 비슷하게 출력도 가능하면 각 유스케이스에 맞게 구체적일수록 좋다. <br>
유스케이스들 간에 같은 출력 모델을 공유하게 되면 유스케이스들도 강하게 결합된다.






