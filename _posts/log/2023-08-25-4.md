---
title:  "[개발 기록] Spring Security - Deprecated WebSecurityConfigurerAdapter"

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

### Deprecated WebSecurityConfigurerAdapter

스프링 시큐리티 5.7.x 부터 WebSecurityConfigurerAdapter 는 Deprecated 되었습니다.

찾아보니 상황에 따라 SecurityFilterChain 과 WebSecurityCustomizer 를 빈으로 등록해 사용하는 방식을 권장하는 것 같습니다.

Spring Security 5.7.x 부터 WebSecurityConfigurerAdapter 는 Deprecated.

- > SecurityFilterChain, WebSecurityCustomizer 를 상황에 따라 빈으로 등록해 사용한다.

```
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final AccountService accountService;
    private final DataSource dataSource;

 @Bean
 public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http.authorizeRequests()
                .mvcMatchers("/", "/login", "/sign-up", "/check-email", "/check-email-token",
  "/email-login", "/check-email-login", "login-link", "/profile/*").permitAll()
                .mvcMatchers(HttpMethod.GET, "/profile/*").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage("/login").permitAll()
                .and()
                .logout().logoutSuccessUrl("/")
                .and()
                .rememberMe().userDetailsService(accountService).tokenRepository(tokenRepository())
                .and().build();
  }

  @Bean
  public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring()
                .mvcMatchers("/node_modules/**")
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations());
  }
}
```

[docs.spring.io_참고](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configuration/WebSecurityConfigurerAdapter.html)