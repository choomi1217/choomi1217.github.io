# SpringBootJpa 개발 | 3 DAY - 4
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

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