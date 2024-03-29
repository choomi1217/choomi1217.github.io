---
title:  "[개발 기록] Configuring the Annotation Processor"

categories:
  - log
tags:
  - [log]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

### 🤔 **Configuring the Annotation Processor**

```java
@Data
@Component
@ConfigurationProperties("app")
public class AppProperties {
    private String host;
}
```

위와 같은 코드를 작성해서 `properties` 파일의 값을 읽어오려고 했는데 아래와 같은 정보창이 떴습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/27bde9c5-dac0-44cc-81a6-b904c49ea9a3/Untitled.png)

아래와 같은 의존성 추가하고 다시 컴파일하면 정보창이 사라지고 IDE 내부에서 자동완성과 네비게이션이 됩니다.

```java
annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"
```

### 🗒️ 참고

[Configuration Metadata](https://docs.spring.io/spring-boot/docs/3.1.1/reference/html/configuration-metadata.html#appendix.configuration-metadata.annotation-processor)

[(Spring Boot) spring-boot-configuration-processor 활용하기](https://perfectacle.github.io/2021/11/21/spring-boot-configuration-processor/)