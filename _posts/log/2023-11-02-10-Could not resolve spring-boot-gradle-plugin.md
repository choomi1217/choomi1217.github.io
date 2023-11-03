---
title:  "[개발 기록] Could not resolve spring-boot-gradle-plugin"

categories:
  - log
tags:
  - [log]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-11-02
last_modified_at: 2023-11-02
---

Spring Boot가 3 버전대로 올라가면서 Java 버전을 17 이상부터만 지원합니다. 
이에 따라 Gradle JVM 자바 버전 설정이 17로 설정이 되어있지 않아 발생했어요.

[Spring Boot Document](https://spring.io/blog/2022/11/24/spring-boot-3-0-goes-ga)

```text
스프링 부트 3.0 정식 출시

새 릴리스의 주요 내용은 다음과 같습니다.

Java 17 기준
실험적인 Spring 네이티브 프로젝트를 대체하는 GraalVM을 사용하여 네이티브 이미지 생성 지원
마이크로미터 및 마이크로미터 추적으로 관찰 가능성 향상
EE 9 기준을 사용하여 Jakarta EE 10 지원
```