---
title:  "[JPA Self Study] AccountBeanValidation"

categories:
  spring-self-study
tags:
  - [spring-self-study]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

# SpringBootJpa 개발 | 2 DAY
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

Hibernate 공식 문서와 Baeldung를 보며 validate를 어떻게 어노테이션만으로 가능하게 했는지 원리를 좀 살펴보았습니다.

이와 같이 객체에 대한 유효성검사를 bean Validate라고 하더군요!

[hibernate_공식도큐_참고](https://docs.jboss.org/hibernate/validator/7.0/reference/en-US/html_single/#preface)
[Baeldung_도큐_참고)](https://www.baeldung.com/javax-validation)


---
### 그러나... validation 처리가 되지 않음
```
@Data  
public class SignUpForm {  
  
  @NotBlank  
 @Length(min = 3, max = 20)  
  @Pattern(regexp = "^[ㄱ-ㅎ가-힣a-z0-9_-]{3,20}$")  
  private String nickname;  
  
  @NotBlank  
 @Email  private String email;  
  
  @NotBlank  
 @Length(min = 8, max = 50)  
  private String password;  
  
}
```

윗 코드 ```   @Pattern(regexp = "^[ㄱ-ㅎ가-힣a-z0-9_-]{3,20}$")  ``` 이 부분에서 정규표현식을 사용한 유효성검사를 한다고 했음에도 불구하고 검사를 하지 않음

여러가지 글을 찾아본 결과 의존성이 충돌한 것 같다는 글을 봤습니다.

SpringFramework의 validation
```
<dependency>  
 <groupId>org.springframework.boot</groupId>  
 <artifactId>spring-boot-starter-validation</artifactId>  
</dependency>
```
제가 추가한 hibernate-validator
```
<dependency>  
 <groupId>org.hibernate.validator</groupId> 
 <artifactId>hibernate-validator</artifactId> 
 <version>7.0.4.Final</version>
</dependency>
```
위 두개의 validator가 충돌이 나서 그랬습니다.
아래 Hibernate를 주석처리하고 테스트 한 결과 잘 작동합니다! ☺
