---
title:  "[개발 기록] MockMvc CSRF Test"

categories:
  - log
tags:
  - [log, spring-log]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

CSRF ( Cross-site Request Forgery )

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

```java
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

제가 스프링-시큐리티 설정을 위와 같이 sign-up일 땐 허용한다고 했음에도 CSRF Token에 막혀 불가능합니다.

즉, 제가 CSRF Token은 Form을 통한 요청이 아니라 불가능하다는 겁니다.

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

위 처럼 `.with(csrf())` 하시면 됩니다.

###