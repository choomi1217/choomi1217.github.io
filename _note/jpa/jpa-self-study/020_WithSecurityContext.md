# SpringBootJpa 개발 | 12 DAY - 2
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

## WithSercurityContext

### How to  Authenticatio User Test

1. 사용자 가입
  -  ``` passwordEncoder.encode(signUpForm.getPassword()) ``` 패스워드가 인코딩 되어 DB에 저장됨.
  - ``` this.emailCheckToken = UUID.randomUUID().toString() ``` 이메일 체크 토큰이 uuid로 만들어져서 DB에 저장됨.

2. 사용자 로그인
  -  ``` UsernamePasswordAuthenticationToken ```을 만들어서 **SecurityContextHolder에 context에 Authentication으로 저장되게 됨.**

3. 사용자 정보 수정
  - **즉, form에 csrf만 넣어서 보낸다고 될 일이 아님.**

### 아래와 같이 mockMvc에 Form을 생성해서 만드는 test 코드로는 작동할 수 없음.

```java
mockMvc.perform(post(SettingsController.SETTINGS_PROFILE_URL)  
  .param("bio",testBio)  
  .with(csrf()))  
  .andExpect(status().is3xxRedirection())  
  .andExpect(redirectedUrl(SettingsController.SETTINGS_PROFILE_URL))  
  .andExpect(flash().attributeExists("message"));  
  
Account account = accountRepository.findAllByNickname("oomi");  
  
assertThat(testBio).isEqualTo(account.getBio());
```

### 그럼 BeforeEach를 사용해서 테스트하면 되는 것이 아닌가?

**아래와 같은 어노테이션 없이는 테스트코드 작동 후에 BeforeEach가 작동 됩니다.**
( 강의에서는 해당 어노테이션 버그로 인해 사용하지 않습니다. )

```java
@WithUserDetails(value = "oomi" , setupBefore = TestExecutionEvent.TEST_EXECUTION)  
```

```java
  
@SpringBootTest  
@AutoConfigureMockMvc  
public class SettingsControllerTest {  
  
  @Autowired  
  MockMvc mockMvc;  
    @Autowired  
  AccountService accountService;  
    @Autowired  
  AccountRepository accountRepository;  
  
    @BeforeEach  
  void addAccount(){  
  SignUpForm signUpForm = new SignUpForm();  
        signUpForm.setNickname("oomi");  
        signUpForm.setEmail("whdudal1217@naver.com");  
        signUpForm.setPassword("123456789");  
        accountService.processNewAccount(signUpForm);  
    }  
  
  
  @WithUserDetails(value = "oomi" , setupBefore = TestExecutionEvent.TEST_EXECUTION)  
  @DisplayName("프로필 수정하기 - 입력값 정상")  
  @Test  
  void updateProfile() throws Exception{  
    }  
  
}
```

## @WithSecurityContext

> Test Code
```java

@WithAccount("oomi")  
@DisplayName("프로필 수정 폼을 보여줍니다.")  
@Test  
void updateProfileForm() throws Exception {  
    mockMvc.perform(get(SettingsController.SETTINGS_PROFILE_URL))  
        .andExpect(status().isOk())  
        .andExpect(model().attributeExists("account"))  
        .andExpect(model().attributeExists("profile"));  
}
```
- @WithAccount라는 어노테이션에 String 값 입력

> Custom Annotation
```java
@Retention(RetentionPolicy.RUNTIME)  
@WithSecurityContext(factory = WithAccountSecurityContextFactory.class)  
public @interface WithAccount {  
    String value();  
}
```

- WithAccount 어노테이션에 value라는 값을 받습니다.

> WithSecurityContextFactory 구현

```java
@RequiredArgsConstructor  
public class WithAccountSecurityContextFactory implements WithSecurityContextFactory<WithAccount> {  
  
    private final AccountService accountService;  
  
    @Override  
  public SecurityContext createSecurityContext(WithAccount withAccount) {  
        String nickName = withAccount.value();  
  
        SignUpForm signUpForm = new SignUpForm();  
        signUpForm.setNickname(nickName);  
        signUpForm.setPassword("123456789");  
        signUpForm.setEmail("oomi@naver.com");  
        accountService.processNewAccount(signUpForm);  
  
        UserDetails principle = accountService.loadUserByUsername(nickName);  
        Authentication authentication = new UsernamePasswordAuthenticationToken(  
            principle, principle.getPassword(), principle.getAuthorities()  
        );  
        SecurityContext context = SecurityContextHolder.createEmptyContext();  
        context.setAuthentication(authentication);  
  
        return context;  
    }  
}
```
- @WithAccount가 받은 value의 값을 꺼내옵니다.
  - `public SecurityContext createSecurityContext(WithAccount withAccount)`
  - `withAccount.value();`
- `SignUpForm`을 만들고 회원가입 로직을 수행합니다.
- `UserNamePasswordAuthenticationToken`을 만듭니다.
- `SecurityContext`에 만들어 둔 `UserNamePasswordAuthenticationToken`을 `setAuthentication`으로 넣어줍니다.
- 그렇게 만든 `SecurityContext` 를 돌려줍니다.
