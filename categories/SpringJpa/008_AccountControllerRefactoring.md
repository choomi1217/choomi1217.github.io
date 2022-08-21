# SpringBootJpa 개발 | 5 DAY - 1
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

## 1. AccountEmailAthenticate

### - About Repository
repository를 도메인 계층으로 보느냐  ayer계층(controller,service,dao..)으로 보느냐에 따라 repository를 controller에서 쓰는 것에 대한 논의가 많은데 여기선 도메인게층으로 보고 controller에서 사용하겠습니다.

###  - Token is NPE ( Transaction Issue )
Email을 통해 Token을 인증하는 과정에서 DB에 저장된 내 정보에 Token이 없어서 문제가 생겼습니다.

문제가 생긴 로직입니다.

**1. processNewAccount**
```
public void processNewAccount(SignUpForm signUpForm) {
  Account newAccount = saveNewAccount(signUpForm);
  newAccount.generateEmailCheckToken();
  sendSignUpConfirmEmail(newAccount);
  }
```

**2. saveNewAccount**
```
private Account saveNewAccount(SignUpForm signUpForm) {
  Account account = Account.builder()
  .email(signUpForm.getEmail())
  .nickname(signUpForm.getNickname())
  .password(passwordEncoder.encode(signUpForm.getPassword()))
  .studyCreatedByWeb(true)
  .studyEnrollmentResultByWeb(true)
  .studyUpdatedByWeb(true)
  .build();

  return accountRepository.save(account);
  }
```

**3. sendSignUpConfirmEmail**
```
private void sendSignUpConfirmEmail(Account newAccount) {
  SimpleMailMessage mailMessage = new SimpleMailMessage();
  mailMessage.setTo(newAccount.getEmail());
  mailMessage.setSubject("스터디올래, 회원 가입 인증");
  mailMessage.setText("/check-email-token?token="+ newAccount.getEmailCheckToken()+"&email="+ newAccount.getEmail());
  javaMailSender.send(mailMessage);
  }
```

1. ``` saveNewAccount(signUpForm) ```으로 저장 후
2.  ```newAccount.generateEmailCheckToken();  ``` 으로 토큰을 생성
3. ``` sendSignUpConfirmEmail(newAccount); ``` 회원가입 인증 메일 전송

**결론 : 계정 저장하고 나서 토큰을 생성해서 저장된 계정엔 토큰이 없음**

이에 대해 강의에서는 **" Detached Object라 DB에 Synch되지 않았다. "** 라고 말했습니다.

###### ~~( 멋지고.. 하나도 못 알아들었다.. 🤗 Jpa 건너뛰고 들었더니.. Jpa에 대해 기본적으로 공부할 것이 있는것 같네요.. 😅 )~~


1번 processNewAccount에서 토큰을 생성했음에도 토큰이 저장이 되지 않는 이유는 1번에서의 **newAccount 객체는 Detached Object**라서 DB에 synch가 되지 않은 겁니다.
생성했음에도 null인 이유는
2번 saveNewAccount에서 이미 트랜잭션 처리를 했기 때문에 이 안에서 해당하는 entity가 JPA entity life cycle에 해당하는 Persistent 상태입니다.
2번을 나온 1번에선 Detached 상태입니다.

1번에서도 Persistent 상태를 유지하려면 **@Transactional** 을 붙이면 됩니다.

```
@Transactional
public void processNewAccount(SignUpForm signUpForm) {
    Account newAccount = saveNewAccount(signUpForm);
    newAccount.generateEmailCheckToken();
    sendSignUpConfirmEmail(newAccount);
}
```

[JPA의 영속성에 대해 참고한 블로그](https://kihoonkim.github.io/2017/01/27/JPA(Java%20ORM)/2.%20JPA-%EC%98%81%EC%86%8D%EC%84%B1%20%EA%B4%80%EB%A6%AC/)

이런 문제를 조기에 발견해서 다행이지만.. TestCode에서 미처 확인하지 못했습니다.

TestCode 보완해서 다시 테스트했습니다.
```
@DisplayName("회원 가입 처리 - 입력값 정상")
@Test
void signUpSubmit_with_correct_input() throws Exception {

  mockMvc.perform(post("/sign-up")
          .param("nickname","oomi")
          .param("email","whdudal1217@naver.com")
          .param("password","1234578910")
          .with(csrf()))
      .andExpect(status().is3xxRedirection())
      .andExpect(view().name("redirect:/"));

  Account account = accountRepository.findByEmail("whdudal1217@naver.com");

  assertThat(account).isNotNull();
  assertThat(account.getPassword()).isNotEqualTo("1234578910");
  assertThat(account.getEmailCheckToken()).isNotNull();
  then(javaMailSender).should().send(any(SimpleMailMessage.class));
}
```

Git Revision Number : 09933326161c696067b1040bdba985e11f472246

## 2. Account Email Athenticate Test and Refactoring



Git Revision Number : 7cfd0e4fc4293e7836a3aa29cc32e0b312c230ca
