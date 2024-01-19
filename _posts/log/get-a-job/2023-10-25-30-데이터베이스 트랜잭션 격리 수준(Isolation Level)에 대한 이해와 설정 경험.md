---
title:  "[개발자 취업] 데이터베이스 트랜잭션 격리 수준(Isolation Level)에 대한 이해와 설정 경험"

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

### 데이터베이스 트랜잭션 격리 수준(Isolation Level)에 대한 이해와 설정 경험

REPEATABLE READ : 트랜잭션이 읽은 데이터에 대한 읽기 락을 설정하여, 다른 트랜잭션이 해당 데이터를 수정하지 못하도록 합니다.
이로써 데이터의 일관성을 유지하며, 다른 트랜잭션과의 충돌을 방지합니다. 다른 트랜잭션에서의 동시 수정을 방지하므로 데드락(deadlock) 상황이 발생할 수 있습니다.

[트랜잭션 격리 수준 참고](https://velog.io/@yujiniii/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B2%A9%EB%A6%AC-%EC%88%98%EC%A4%80)
