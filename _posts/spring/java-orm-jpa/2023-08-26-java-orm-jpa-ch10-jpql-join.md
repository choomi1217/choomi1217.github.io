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
last_modified_at: 2023-08-26
---

# JPQL 조인
JPQL 조인은 연관 필드를 사용합니다.
아래처럼 `Member m`의 조인으로 `m.team`이라는 본인의 필드를 사용해 조인합니다.
`Member m JOIN m.team t`

### 내부조인
`JOIN` 에 아무것도 붙이지 않으면 자연스럽게 내부조인이 됩니다.

< 코드 >
```java
String query = "SELECT m.name, t.name FROM Member m JOIN m.team t WHERE t.name = :name";
List<Object[]> resultList = em.createQuery(query).setParameter("name", "teamA").getResultList();
```

< 결과 >
```text
select
        member0_.name as col_0_0_,
        team1_.name as col_1_0_  
    from
        Member member0_ 
    inner join Team team1_ on member0_.TEAM_ID=team1_.TEAM_ID 
    where
        team1_.name=?
```

### 외부조인
`LEFT JOIN`, `LEFT OUTER JOIN` 을 사용하면 외부조인이 됩니다.
`OUTER`는 생략 가능합니다.

< 코드 >
```java
String query = "SELECT m.name, t.name FROM Member m LEFT OUTER JOIN m.team t WHERE m.name = :name";
List<Object[]> resultList = em.createQuery(query).setParameter("name", "kim").getResultList();

resultList.forEach(o -> {
  System.out.println("--------------------");
  System.out.println("member name | " + o[0]);
  System.out.println("team name | " + o[1]);
  System.out.println("--------------------");
});
```

< 결과 >
```text
Hibernate: 
    select
        member0_.name as col_0_0_,
        team1_.name as col_1_0_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.TEAM_ID=team1_.TEAM_ID 
    where
        member0_.name=?
--------------------
member name | kim
team name | teamB
--------------------
--------------------
member name | kim
team name | null
--------------------
```

### 컬렉션 조인
일대다, 다대다 관계처럼 컬렉션을 사용하는 곳에 조인하는 것을 컬렉션 조인이라고 합니다. 
그냥 일반 조인처럼 사용하면 됩니다.
```java
String query = "SELECT t, m FROM Team t JOIN t.members m WHERE t.name = :name";
```

### 세타 조인
연관관계가 아닌 필드를 이용해 조인을 할 수 있습니다.
이땐 `WHERE` 절을 이용해 조인합니다.

< 코드 >
```java
String query = "SELECT m.team, m.id, m.name, t.name FROM Member m, Team t WHERE m.name = t.name";

List<Object[]> resultList = em.createQuery(query).getResultList();

resultList.forEach(o -> {
  Team team = (Team) o[0];
  System.out.println("--------------------");
  System.out.println("member id: " + o[1]);
  System.out.println("member name: " + o[2]);
  System.out.println("member's team name: " + team.getName());
  System.out.println("joined team name: " + o[3]);
  System.out.println("--------------------");
});
```

< 결과 >
```text
Hibernate: 
    select
        member0_.TEAM_ID as col_0_0_,
        member0_.MEMBER_ID as col_1_0_,
        member0_.name as col_2_0_,
        team1_.name as col_3_0_,
        team2_.TEAM_ID as team_id1_4_,
        team2_.name as name2_4_ 
    from
        Member member0_ cross 
    join
        Team team1_ 
    inner join
        Team team2_ 
            on member0_.TEAM_ID=team2_.TEAM_ID 
    where
        member0_.name=team1_.name
--------------------
member id: 9
member name: teamA
member's team name: teamD
joined team name: teamA
--------------------
```