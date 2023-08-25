---
title:  "[JPA Self Study] SpringSecurity Login"

categories:
  - spring
tags:
  - [spring-self-study]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

# SpringBootJpa 개발 | 6 DAY - 1
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

## 1. Spring Security

- 방법 1 ( 백기선 강의에서 말하는 정석적인 루틴 )
```java
private final AuthenticationManager authenticationManager;

public void login(Account account) {  
    UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(  
        account.getNickname(), account.getPassword()));  
  
    Authentication authentication = authenticationManager.authenticate(token);  
    SecurityContext context = SecurityContextHolder.getContext();  
    context.setAuthentication(authentication);  
}
```
- 방법 2
  - 현재 개발 상태에선 암호화 된 패스워드에만 접근이 가능한 상태이므로 이 방법을 사용
  - 방법 1로 하려면 암호화 되지 않은 플레인 패스워드를 알고 있어야 함
``` java
UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(  
    account.getNickname()  
    , account.getPassword()  
    , List.of(new SimpleGrantedAuthority("ROLE USER"))  
);  
SecurityContext context = SecurityContextHolder.getContext();  
context.setAuthentication(token);
```
- Spring Security Test code
  이렇게 적용한 Spring Security가 정상작동 하는지 확인 하려면 테스트코드를 짜야합니다.
  mockMvc에 SpringSecurity를 지원하는 기능이 있습니다.
```java
mockMvc.perform(post("/sign-up") 
	  .param("nickname","**oomi")  
	  .with(csrf()))  
      .andExpect(unauthenticated());  
      .andExpect(authenticated().withUsername("**oomi"));
```

Git Revision Number : cc7304981c4bd771e2f7569c0b4b23b80b417b24
