---
title:  "[JPA Self Study] ProfileAnnotation"

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

# SpringBootJpa 개발 | 3 DAY - 2
```
하다가 막히는 부분이나 알아야 할 사항, 혹은 버전의 문제 등을 적기 위한 간단한 노트이며

이는 공부용 소스이기 때문에 소스에 주석을 달아가며 설명을 적어뒀습니다.
따라서 해당 날짜의 소스를 보며 노트를 참고하는 식으로 봐야합니다.

소스는 깃헙에 올려놓았습니다.
```


## @Profile

application.properties
```properties
spring.profiles.active=local
```

Java Class File
```java
@Profile("local")
```

프로퍼티 파일에 " local " 을 해두고 자바 클래스의``` @Profile ``` 프로필 어노테이션으로 " local " 을 적어두면
어플리케이션의 런타임 환경을 관리 할 수 있습니다.
