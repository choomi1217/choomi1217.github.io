---
title:  "[개발자 취업] 영속성 컨텍스트의 생명주기와 JPA 엔티티 상태(Transient, Persistent, Detached, Removed)에 대해 설명해주세요."

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

### 영속성 컨텍스트의 생명주기와 JPA 엔티티 상태(Transient, Persistent, Detached, Removed)에 대해 설명해주세요.

비영속(일시적인)상태는 JPA와 관련 없는 그냥 객체를 생성한 상태입니다.
해당 객체를 persist하거나 DB로부터 읽어오면 persistent 상태로 엔티티 매니저가 관리하는 대상이 됩니다.
또한, 영속 엔티티는 영속성 컨텍스트에 저장되며, 이 컨텍스트 내에서 엔티티의 상태를 추적합니다.
준영속 상태는 영속성 컨텍스트와 더 이상 관련이 없는 상태입니다.
엔티티 매니저를 종료했거나, detach메소드를 실행하면 영속성 컨텍스트로부터 분리됩니다.
삭제는 엔티티에 Removed 표시를 해두고 엔티티 매니저 flush를 하면 쿼리가 실행되고 db에서 삭제됩니다.


객체를 생성하고 persist하거나 db로부터 find한 객체는 1차 캐시에 엔티티가 저장된 상태가 됩니다.
영속성 컨텍스트 내에서 엔티티를 찾거나 업데이트할 때 1차 캐시에서 해당 엔티티를 검색하게 됩니다.
이로써 JPA는 같은 데이터에 대해 같은 인스턴스를 제공하게 됩니다.

JPA는 트랜잭션을 커밋하기 전까진 쓰기지연 sql 저장소에 insert,update,delete등의 쿼리를 보관해두고 실행하지 않습니다.
트랜잭션이 커밋되면 엔티티 매니저가 영속성 컨텍스트를 FLUSH 하고 이때 저장소의 쿼리들이 db에서 실행됩니다.

