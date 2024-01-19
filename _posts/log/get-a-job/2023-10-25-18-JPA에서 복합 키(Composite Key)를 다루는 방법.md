---
title:  "[개발자 취업] JPA에서 복합 키(Composite Key)를 다루는 방법"

categories:
  - get-a-job
tags:
  - [get-a-job]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-10-25
last_modified_at: 2023-10-25
---

**제 경험이 조금씩 첨가된 주관적인 답변이므로 참고만 하세요! 답이 아닙니다! 여러분의 의견을 존중합니다!**

### JPA에서 복합 키(Composite Key)를 다루는 방법

IdClass를 이용한 방법과 EmbeddedId를 이용한 방법이 있습니다.
IdClass를 사용하면 어노테이션에 복합 키 클래스를 맵핑해야하고
EmbeddedId를 사용하면 필드로 복합 키 클래스를 맵핑합니다. 클래스 내에 복합 키 클래스를 포함하면 가독성이 좋을 것 같습니다.
