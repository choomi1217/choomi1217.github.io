---
title:  "[자바 ORM 표준 JPA] ch10.객체지향 쿼리 - 조인편"

categories:
  java-orm-jpa
tags:
  - [java-orm-jpa]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

# JPQL 조인

### 내부조인

```java
String query = "SELECT m FROM Member m JOIN m.team t WHERE t.name = :name";
List<Member> resultList = em.createQuery(query, Member.class)
        .setParameter("name", "teamA")
        .getResultList();
```
```text
select
        member0_.MEMBER_ID as member_i1_0_,
        member0_.city as city2_0_,
        member0_.street as street3_0_,
        member0_.zipcode as zipcode4_0_,
        member0_.age as age5_0_,
        member0_.name as name6_0_,
        member0_.TEAM_ID as team_id7_0_ 
    from
        Member member0_ 
    **inner join Team team1_ on member0_.TEAM_ID=team1_.TEAM_ID** 
    where
        team1_.name=?
```
