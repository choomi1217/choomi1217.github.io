---
title:  "[자바 ORM 표준 JPA] ch11-1.settings (스프링 프레임워크와 스프링 부트 차이)"

categories:
  java-orm-jpa
tags:
  - [java-orm-jpa]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-11-02
last_modified_at: 2023-11-02
---

### 책과 다른 버전들을 사용했으므로 설정이 다릅니다.
설정에 대해서 다른점을 다루다보니 의도치않게 스프링프레임워크와 스프링부트의 차이점을 정리하게 됐습니다 😅

1. java version 17
2. springframework.boot version '3.1.5'
3. mySql 사용
4. gradle 사용

### 스프링 프레임워크와 스프링 부트의 트랜잭션
스프링 프레임워크에서 트랜잭션을 설정하는 아래와 같은 코드는 필요 없어졌습니다.
```text
<tx:annnotation-driven/>
```
대신 아래처럼 Java 코드를 통해 설정 할 수 있지만 스프링부트에선 트랜잭션 어노테이션이 자동입니다.
```java
@Configuration
@EnableTransactionManagement
public class TransactionConfig {
    // 추가적인 설정 및 빈들을 여기에 정의할 수 있습니다.
}
```

### 스프링 프레임워크와 스프링 부트의 component-scan 
SpringFramework는 `<context:component-scan base-package />`로 했습니다.
Spring Boot는 `@SpringBootApplication` 어노테이션이 있는 메인 애플리케이션 클래스에서 시작하여 해당 패키지와 하위 패키지의 모든 컴포넌트를 스캔합니다.

### 스프링 프레임워크와 스프링 부트의 데이터 액세스 예외 처리
SpringFramework는 `<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>`을 통해 
데이터 액세스 레이어에서 발생한 예외를 [Spring의 예외 계층 구조](../../log/2023-11-02-10-Could%20not%20resolve%20spring-boot-gradle-plugin.md)로 변환하여 처리하는 역할을 합니다.
`@Repository`가 붙은 스프링 빈에 예외 변환 AOP를 적용합니다.
Spring Boot는 자동으로 합니다.

### 스프링 프레임워크와 스프링 부트의 엔티티 탐색
`@SpringBootApplication` 어노테이션이 있는 메인 애플리케이션 클래스의 패키지와 하위 패키지
`@EntityScan` 어노테이션을 사용하여 명시적으로 지정한 패키지
