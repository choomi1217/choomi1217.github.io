# SpringBootJpa 개발 | 9 DAY
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

## Remember Me

Remember Me를 사용하기 전엔 JSession이라는 것을 사용해서 로그인 정보를 저장하곤 했습니다.

### 👿 JSession의 세션 유지시간이 정해져있다.
> 🔑 해결법
- ``` server.servlet.session.timeout=30m ``` 세션 유지 시간은 기본 30분입니다. ( 너무 길게 늘리면 메모리 낭비가 있을 수 있음 )
- 따라서 일정 시간 마다 로그인이 풀리게 되므로 쿠키를 하나 더 사용합니다. 그게 `Remeber Me` 입니다.
### 👿 Remember Me 쿠키가 탈취 당할 수 있음. 취약
> 🔑 해결법
- 이런 경우를 대비해서 매번 쿠키를 만들 때마다 쿠키에 랜덤한 문자열을 넣어서 토큰을 만들어서 저장합니다.
### 👿 해커가 먼저 쿠키로 인증을 시도하면 사용자의 쿠키는 유효해지지 않음. 고로 이것도 취약.
> 🔑 해결법
- 랜덤한 토큰값 + 시리즈
- 해커가 토큰으로 먼저 로그인하면 유저의 토큰은 유효하지 않은 토큰으로 로그인을 시도 하게 됨.
- 즉 한 계정에 유효한 토큰, 유효하지 않은 토큰, 2개의 토큰 생성. 그러면 나는 두개의 토큰을 다 삭제해버림.
- 그럼 다시 id, pwd 입력을 통한 로그인을 하게 됨.

### How to use
spring-sercurity 설정 파일에 아래와 같이 `http.rememberMe`를 설정하면 되는데..
```java
@Override
protected void configure(HttpSecurity http) throws Exception{
	http.rememberMe()  
	    .userDetailsService(accountService)  
	    .tokenRepository(tokenRepository());
}

@Bean  
public PersistentTokenRepository tokenRepository(){  
  JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();  
  jdbcTokenRepository.setDataSource(dataSource);  
  return jdbcTokenRepository;  
}
```
`tokenRepository`에서 사용되는 `JdbcTokenRepositoryImpl` 를 열어보면 아래와 같은 DDL이 있습니다.
```sql
create table persistent_logins 
(
	username varchar(64) not null
	, series varchar(64) primary key
	, token varchar(64) not null
	, last_used timestamp not null
)
```
현재 JPA를 설정없이 인메모리 디비를 사용중이므로 엔티티 정보를 보고 테이블을 자동생성 하므로 위 테이블에 해당하는 엔티티를 추가합니다.

RememberMe 사용시 꼭 필요한 form 데이터입니다!
```html
<div class="form-group form-check">  
  <input type="checkbox" class="form-check-input" id="remeberMe" name="remeber-me" checked>  
  <label class="form-check-label" for="remeberMe" aria-describedby="rememberMeHelp">로그인유지</label>  
</div>
```





