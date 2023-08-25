---
title:  "[자바 ORM 표준 JPA] ch7.고급매핑 - 조인 테이블"

categories:
  - spring
tags:
  - [spring-orm-jpa]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

# 조인 테이블

[6장 다양한 연관관계 매핑](https://www.notion.so/6-4e81f87c1c914a1885065067bfaf9c51?pvs=21)

1. 조인 컬럼 사용 ( 외래 키 )
2. 조인 테이블 사용 ( 관계 테이블 )

### 조인 컬럼 사용

- `member` 가 `locker` 를 등록하기 전엔 둘 사이에 관계가 없으므로
  `member` 의 `locker_id` 가 `null` 이 됩니다.
  이렇게 외래 키에 null 을 허용하는 관계를 [선택적 비식별 관계](2023-08-25-ch7-4-identity_composekey.md)라고 합니다.

![Untitled](../image/7/7_14.png)

### 조인 테이블 사용
![Untitled](../image/7/7_15.png)