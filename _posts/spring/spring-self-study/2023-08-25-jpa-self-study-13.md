---
title:  "[JPA Self Study] ThymeLeaf & SpringSecurity"

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

# SpringBootJpa 개발 | 6 DAY - 2
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```

## Thymeleaf-extras-springsecurity5

타임리프에서 사용자의 권한이나 로그인 상태에 따라 버튼을 다르게 보여주기 위한 의존성 추가
```xml
<dependency>  
 <groupId>org.thymeleaf.extras</groupId>  
 <artifactId>thymeleaf-extras-springsecurity5</artifactId>  
 <version>3.0.4.RELEASE</version>  
</dependency>
```
타임리프에서 사용하기 위해선 아래처럼 선언해줘야 합니다.
```html
<html lang="en"  
  xmlns:th="http://www.thymeleaf.org"  
  xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
```
```html
<li class="nav-item" sec:authorize="!isAuthenticated()">  
 <a class="nav-link" th:href="@{/login}">로그인</a>  
</li>
<a class="dropdown-item" th:href="@{'/profile/' + ${#authentication.name}}">프로필</a>
```
( org.thymeleaf.extras 참고 블로그 )[https://velog.io/@codren/thymeleaf-extras-springsecurity-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC]

### 프론트엔드 라이브러리 설정
써드파티 라이브러리는 직접 jar 라던가 js 라던가 하는 파일로 내려받아서 직접 static resource로 관리하는 것보다 의존성 관리 툴의 지원을 받아서 사용하는 걸 추천합니다.

1. WebJar를 사용하는 방법
  - pom.xml에 bootstrap 의존성을 추가하는 것

2. NPM을 사용하는 방법
  - npm install bootstrap을 통해 다운 받는 것
  - 이때 다른 빌드툴이 빌드 할 때 프론트 쪽 모듈도 같이 빌드를 해야합니다. ( NPM install을 실행한다는 의미 )
  - 빌드를 위한 플러그인

```xml
<plugin>
	<groupId>com.github.eirslett</groupId>
	<artifactId>frontend-maven-plugin</artifactId>
	<version>1.8.0</version>
	<configuration>
		<nodeVersion>v4.6.0</nodeVersion>
		<workingDirectory>src/main/resources/static</workingDirectory>
	</configuration>
	<executions>
		<execution>
			<id>install node and npm</id>
			<goals>
				<goal>install-node-and-npm</goal>
			</goals>
			<phase>generate-resources</phase>
		</execution>
		<execution>
			<id>npm install</id>
			<goals>
				<goal>npm</goal>
			</goals>
			<phase>generate-resources</phase>
			<configuration>
				<arguments>install</arguments>
			</configuration>
		</execution>
	</executions>
</plugin>
```

spring-self-studySecurity 설정도 바꿔줘야 합니다
```java
@Override  
public void configure(WebSecurity web) throws Exception {  
  web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());  
}
```
Spring Security 설정은 했었는데 왜 또 걸리냐면
StaticResourceLocation 클래스에 정의된 아래 경로의 파일이 아니면 걸립니다.

```java
CSS(new String[]{"/css/**"}),  
JAVA_SCRIPT(new String[]{"/js/**"}),  
IMAGES(new String[]{"/images/**"}),  
WEB_JARS(new String[]{"/webjars/**"}),  
FAVICON(new String[]{"/favicon.*", "/*/icon-*"});
```
