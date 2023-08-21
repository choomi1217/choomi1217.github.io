# Service에서 필요한 Entity의 Id

- 아래와 같이 서비스 로직에서 엔터티의 id가 필요한 일이 생겼다..
- 도메인과 엔터티를 분리하고 싶었는데 하는 수 없이 엔터티에 id를 넣을 수 밖에 없었다.
- 그러고선 한참을 구글링을 하고 찾아다닌 끝에 아래와 같은 내용을 챗gpt를 통해 들었다.
- Aggregate Root가 고유 식별자를 가져도 좋다

<aside>
💡 레이어의 책임과 역할에 따라, **`Domain`** 객체에서 **`id`**를 포함시키는 것도 고려해볼 수 있습니다. DDD(Domain-Driven Design)에서는 Aggregate Root가 고유 식별자를 가져도 좋다는 점을 강조합니다. 그래서 도메인 객체에서도 id를 가질 수 있으며, 이는 JPA의 **`Entity`**와의 변환 과정에서 유지될 수 있습니다.

</aside>

```java
// AccountService login method
public LoginResponse login(LoginRequest loginRequest) {
        Account account = accountRepository.findByEmail(loginRequest.email());
        if(!passwordEncoder.matches(loginRequest.password(), account.password())){
            // todo: 비밀번호 불일치 예외 처리
            throw new IllegalArgumentException("Password is not matched");
        }
        if(account.isEmailVerified()){
            userService.findUserByAccountId(account.id());
        }
        return new LoginResponse(
            account.email(), account.phoneNumber(), null
        );
    }

// Account Record
public record Account (Long accountId, String email, String password, String phoneNumber, boolean isEmailVerified, boolean isActive) {

    public Account(String email, String password, String phoneNumber) {
        this(null, email, password, phoneNumber, false,  true);
    }
}
```