---
title:  "[JPA Self Study] AuthenticationPrinciple"

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


# SpringBootJpa 개발 | 7 DAY - 2
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

## @AuthenticationPrinciple

기본 루트로 접근하는 사람의
```
@GetMapping("/")
public String home(@CurrentUser Account account, Model model){  
    if(account != null){  
        model.addAttribute(account);  
    }  
    return "index";  
}
```

AccountController 에서
`@PostMapping("/sign-up")` 매핑으로 들어온 요청에 대해
`accountService.login(loginAccount);`를 실행하고
```java
UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(  
    new UserAccount(account)  
    ,account.getPassword()  
    , List.of(new SimpleGrantedAuthority("ROLE USER"))  
);
```
이렇게 `UsernamePasswordAuthenticationToken`을 만들어줄 때 token 생성자에 `new UserAccount(account)`를 넣어주고


### Error in @AuthenticationPrinciple
anonymousUser 오타를 내면
```
@AuthenticationPrincipal(expression = "#this == 'anonymousUser' ? null : account")
```
해당 에러가 남
```
org.springframework.expression.spel.SpelEvaluationException: EL1008E: Property or field 'account' cannot be found on object of type 'java.lang.String' - maybe not public or not valid?
```

### Error

```
2022-08-08 11:14:26.431 ERROR 7952 --- [nio-8080-exec-5] org.thymeleaf.TemplateEngine             : [THYMELEAF][http-nio-8080-exec-5] Exception processing template "index": An error happened during template parsing (template: "class path resource [templates/index.html]")

org.thymeleaf.exceptions.TemplateInputException: An error happened during template parsing (template: "class path resource [templates/index.html]")
```
...? 한참을 이리저리 살펴보다가... 한 30분 태웠는데..
아래 코드에서 첫번째는 위 에러를 뱉고 두번째는 정상작동합니다..
```html
<div class="alert alert-warnning" role="alert" th:if="${account!=null && !acoount.emailVerified}">
```
```html
<div class="alert alert-warnning" role="alert" th:if="${account != null && !account.emailVerified}">
```


---



## 어노테이션 정리
> 에러를 해결하고 나서 어노테이션 정리

### @Retention

> 커스텀 어노테이션을 생성할 때 주로 사용하는 어노테이션
> 왜 커스텀 어노테이션을 생성할 때 사용하는데..?

RetentionPolicy를 파라미터로 받음
```
@Documented  
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.ANNOTATION_TYPE)  
public @interface Retention {  
    RetentionPolicy value();  
}
```
RetentionPolicy 는 enum의 형태로 3가지 값 제공
```java
public enum RetentionPolicy {  
	SOURCE, CLASS, RUNTIME  
}
```

@Reention을 붙인 해당 객체가 메모리에 존재할 수 있는 기간을 정의
SOURCE 는 컴파일러가 컴파일하면서 버립니다
CLASS 는 클래스가 살아있는 동안
RUNTIME 은 애플리케이션이 활동하는 동안

### @AuthenticationPrincipal

---
```html
<a th:href="@{/check-email}" class="alert-link">계정 인증 이메일을 확인</a>하세요.
```
여기서 /check-email로 넘어오는 과정에
form 데이터를 보낸다던가 한 것이 아닌데도 데이터가 잘 오가는걸 보면
```java
@GetMapping("/check-email")  
public String checkEmail(@CurrentUser Account account, Model model){  
  model.addAttribute("email", account.getEmail());  
  return "account/check-email";  
}
```
아래 코드에 뭔가 동작하는게 따로 있구나 싶었습니다.
`@CurrentUser Account account`

https://shirohoo.github.io/spring/spring-security/2022-03-31-mvcMatchers/
https://github.com/shirohoo/sample-spring-security-login
https://shirohoo.github.io/spring/spring-security/2021-08-30-http-login/