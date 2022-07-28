# SpringJpa 개발 | DAY 3
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

### Fail - Fast란
JavaScript를 이용한 validation-check는 사실 비정상적인 루트를 통해 들어오는 요청엔 소용이 없고 결국 백엔드에서 걸러야 하는데 그럼 왜 귀찮게 굳이 Front에서 걸러서 들어오냐면 

> **Fail-Fast 에 의미가 있다.**

공항까지 와서 여권이 없다는 걸 알아채고 " 아, 여권 없네 " 하고 집으로 돌아가는 것
그리고
집을 나오며 " 아, 여권 없네 " 하고 가지고 나오는 것의 차이이다.


### @Profile

application.properties
```properties
spring.profiles.active=local
```

Java Class File
```java
@Profile("local")
```

프로퍼티 파일에 " local " 을 해두고 자바 클래스의``` @Profile ``` 프로필 어노테이션으로 " local " 을 적어두면 
어플리케이션의 런타임 환경을 관리 할 수 있습니다.

### CSRF ( Cross-site Request Forgery )
사이트 간 요청 위조

MockMvc를 이용한 테스트 도중에 에러가 났습니다.
에러가 난 콘솔창입니다.
``` 
MockHttpServletRequest:
      HTTP Method = POST
      Request URI = /sign-up
       Parameters = {nickname=[oomi], email=[whdudal1217.naver.com], password=[12345]}
          Headers = []
             Body = null
    Session Attrs = {org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository.CSRF_TOKEN=org.springframework.security.web.csrf.DefaultCsrfToken@2f006edf}

Handler:
             Type = null

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 403
    Error message = Forbidden
          Headers = [X-Content-Type-Options:"nosniff", X-XSS-Protection:"1; mode=block", Cache-Control:"no-cache, no-store, max-age=0, must-revalidate", Pragma:"no-cache", Expires:"0", X-Frame-Options:"DENY"]
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

java.lang.AssertionError: Status expected:<200> but was:<403>
Expected :200
Actual   :403
<Click to see difference>

... 생략

2022-07-28 14:59:43.771  INFO 27860 --- [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2022-07-28 14:59:43.772  INFO 27860 --- [ionShutdownHook] .SchemaDropperImpl$DelayedDropActionImpl : HHH000477: Starting delayed evictData of schema as part of SessionFactory shut-down'
2022-07-28 14:59:43.783  INFO 27860 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2022-07-28 14:59:43.790  INFO 27860 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```



타 사이트에서 제 사이트로 폼 데이터를 보내는 것 이런걸 방지하는 기술로 CSRF 토큰이라는 걸 사용합니다.

현재 개발 환경에서 CSRF Token이란 것을 기본적으로 발급하게 됩니다.
저는 넣지 않았음에도 불구하고 말이죠..

![where is my CSRF Token](/images/2022/07/28/CSRFToken.png)

위 사진 속 숨겨진 인풋박스가 CSRF Token입니다.

제가 스프링-시큐리티 설정을
```
@Override  
protected void configure(HttpSecurity http) throws Exception {  
  http.authorizeRequests()  
      .mvcMatchers(  
          "/"  
  ,"/login"  
  ,"/sign-up"  
  ,"/check-email"  
  ,"/check-email-token"  
  ,"/email-login"  
  ,"/check-email-login"  
  ,"/login-link"  
  ,"/profile/*"  
  ).permitAll()  
      .mvcMatchers(HttpMethod.GET,"/profile/*").permitAll()  
      .anyRequest().authenticated();  
}
```
위와 같이 sign-up일 땐 허용한다고 했음에도 CSRF Token에 막혀 불가능합니다.

즉, 제가 CSRF Token이 존재하는 Form을 통한 요청이 아니라 제가 테스트를 위해 만든 요청이라 불가능하다는 겁니다.

이럴 땐
```
static import org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.csrf

... 생략

	mockMvc.perform(post("/sign-up")  
    .param("nickname","oomi")  
    .param("email","whdudal1217.naver.com")  
    .param("password","12345")  
    .with(csrf()))  
    .andExpect(status().isOk())  
    .andExpect(view().name("account/sign-up"));
```
위 처럼 ``` .with(csrf()) ``` 하시면 됩니다.



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
메일에 대한 테스트를 진행 하다가 BDDMokito를 사용

[BDD_Baeldung_도큐 참고](https://www.baeldung.com/bdd-mockito)

then
should
any








