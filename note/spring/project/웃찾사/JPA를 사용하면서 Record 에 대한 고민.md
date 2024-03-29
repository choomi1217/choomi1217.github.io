### 🤔Record는 불변, 그럼 사용자의 데이터 업데이트는?

이메일 인증시 `account` 의 `emailVerificationStatus` 를 변경해줘야 하는 로직이 있습니다.

`Account`를 record로 했더니 변경이 안됩니다.

> record는 불변 객체로 abstract로 선언할 수 없으며 암시적으로 final로 선언됩니다. 한 번 값이 정해지면 setter를 통해 값을 변경할 수 없으며 상속을 할 수 없습니다.
> 

```java
public void emailVerification(String token, String email) {
    Account account = accountRepository.findByEmail(email);
    if (account.emailVerificationStatus()== AccountEmailVerificationStatus.VERIFIED) {
        throw new IllegalArgumentException("Already Verified");
    }
    if(token.equals(account.emailVerificationToken())) {
        account.verifyEmail();
    }else{
        throw new IllegalArgumentException("Invalid Token");
    }
}
```

```java
public record Account (Long accountId, String email, String password, String emailVerificationToken , AccountEmailVerificationStatus emailVerificationStatus, AccountActiveStatus activeStatus) {

    public Account(String email, String password) {
        this(null,
                email,
                password,
                generateEmailVerificationToken(),
                AccountEmailVerificationStatus.NON_VERIFIED, AccountActiveStatus.ACTIVE);
    }

    private static String generateEmailVerificationToken() {
        return UUID.randomUUID().toString();
    }

    public boolean verifyEmail(String token) {
        return token.equals(emailVerificationToken);
    }

}
```

---

### 🔧새로운 객체

```java
public AccountResponse emailVerification(String token, String email) {
        Account account = accountRepository.findByEmail(email);
        if (account.emailVerificationStatus()== AccountEmailVerificationStatus.VERIFIED) {
            throw new IllegalArgumentException("Already Verified");
        }
        if(token.equals(account.emailVerificationToken())) {
            Account verified = account.verifyEmail();
            accountRepository.update(verified);
            return new AccountResponse(verified.email(), verified.emailVerificationStatus(), verified.activeStatus());
        }else{
            throw new IllegalArgumentException("Invalid Token");
        }
    }
```

```java
public Account verifyEmail() {
    return new Account(
            accountId,
            email,
            password,
            emailVerificationToken,
            AccountEmailVerificationStatus.VERIFIED,
            activeStatus
    );
}
```

-----

# rec

citizen 저장하는 기능을 만들던 도중 `record` 의 역할에 대해 생각하게 되었습니다.

```java
public record CitizenResponse(
        String nickname,
        String phoneNumber,
        AddressResponse address,
        HobbyResponse hobby
) {
    public CitizenResponse(Citizen saved) {
        this(saved.nickname(), saved.phoneNumber(), new AddressResponse(saved.address()), new HobbyResponse(saved.hobby()));
    }
}
```

책임 분리(Separation of Responsibility)와 단일 책임 원칙(Single Responsibility Principle)을 따르면 아래와 같은 Factory 클래스를 하나 더 만드는게 맞다고 생각 중입니다만.. 🤔

레코드마다 Factory 클래스가 하나씩 더 생기고 이 클래스들도 관리 할 것을 생각하면.. 잘 모르겠습니다.

```java
public class CitizenResponseFactory {
    public static CitizenResponse from(Citizen saved) {
        return new CitizenResponse(
            saved.nickname(),
            saved.phoneNumber(),
            new AddressResponse(saved.address()),
            new HobbyResponse(saved.hobby())
        );
    }
}
```