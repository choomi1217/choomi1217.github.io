---
title:  "[개발 기록] validation dependency 충돌"

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

Hibernate 공식 문서와 Baeldung 블로그를를 보며 validate를 어떻게 어노테이션만으로 가능하게 했는지 궁금해졌습니다.

이와 같이 객체에 대한 유효성검사를 `bean Validate`라고 합니다.

[hibernate_공식도큐_참고](https://docs.jboss.org/hibernate/validator/7.0/reference/en-US/html_single/#preface)

[Baeldung_도큐_참고](https://www.baeldung.com/javax-validation)

---

### 🤔 문제

validation 처리가 되지 않습니다.

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

위 코드 `@Pattern(regexp = "^[ㄱ-ㅎ가-힣a-z0-9_-]{3,20}$")` 이 부분에서 정규표현식을 사용한 유효성검사를 한다고 했음에도 불구하고 검사를 하지 않습니다.

의존성이 충돌한 것 같았습니다.

SpringFramework의 validation

```
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

hibernate-validator

```
<dependency>
 <groupId>org.hibernate.validator</groupId>
 <artifactId>hibernate-validator</artifactId>
 <version>7.0.4.Final</version>
</dependency>
```

위 두개의 validator는 충돌이 납니다. 🥲