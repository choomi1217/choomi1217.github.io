# SpringBootJpa 개발 | 8 DAY
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

## Spring-Security Provided Login by default

Spring Security config

```java
http.formLogin()  
    .loginPage("/login")  
    .permitAll();  
  
http.logout()  
    .logoutSuccessUrl("/");
```

하지만 로그인 부분은 db 조회하는 과정이 있어야 하므로 유저정보를 조회하는 부분을 만들어야 하는데 `UserDetailsService` 를 implements해서 `loadUserByUsername`를 만듭니다.

