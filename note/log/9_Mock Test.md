### MockBean

메일이 제대로 보내졌는지 테스트하는 코드를 짜는 과정에서 MockBean을 사용했습니다.

```
@MockBean
JavaMailSender javaMailSender;

... 생략

@DisplayName("메일 처리")
@Test
void mailTest() throws Exception {

  then(javaMailSender).should().send(any(SimpleMailMessage.class));

}
```

### BDD Mokito : Behavior Driven Development

메일에 대한 테스트를 진행 하며 BDDMokito를 사용

[BDD_Baeldung_도큐 참고](https://www.baeldung.com/bdd-mockito)