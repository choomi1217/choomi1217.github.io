---
title:  "[자바 ORM 표준 JPA] ch6.다양한 연관관계 매핑 - 일대다"

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

# 일대다

**차라리 다대일 양방향 매핑을 사용하심이 좋습니다.**

`Team` 이 `Member` 를 조회하고 관리합니다.

`@JoinColumn` 을 사용해야 합니다. 그렇지 않으면 관계 테이블을 생성해 매핑합니다.

### ⬅️단방향

아래 코드 결과를 보시면 `Insert` 이후 `Update` 를 이용해 `Member` 테이블의 FK를 바꿔주는 모습입니다.

**차라리 다대일 양방향 매핑을 사용하심이 좋습니다.**

대상

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;
}
```

주인( DB에선 FK를 가진 Member가 주인 )

```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany
    @JoinColumn(name = "team_id")
    private List<Member> members = new ArrayList<Member>();
}
```

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

tx.begin();
Member member1 = new Member("member1");
Member member2 = new Member("member2");

Team team1 = new Team("team1");
team1.getMembers().add(member1);
team1.getMembers().add(member2);

em.persist(member1);
em.persist(member2);
em.persist(team1);

tx.commit();
emf.close()
```

```java
Hibernate: 
    select
        nextval ('hibernate_sequence')
Hibernate: 
    select
        nextval ('hibernate_sequence')
Hibernate: 
    select
        nextval ('hibernate_sequence')
Hibernate: 
    insert 
    into
        Member
        (name, member_id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        Member
        (name, member_id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        Team
        (name, id) 
    values
        (?, ?)
Hibernate: 
    update
        Member 
    set
        team_id=? 
    where
        member_id=?
Hibernate: 
    update
        Member 
    set
        team_id=? 
    where
        member_id=?
```

### ↔️ 양방향

`Member` 가 주인이고 `Team` 이 종속된 객체라면 `Team` 에 `mappedBy` 속성을 적어 주인을 분류하는데

이땐 `Team` 에 `JoinColumn` 을 통해 매핑하고, `Member` 는 읽기전용 `Team` 을 가지게 됩니다.

**차라리 다대일 양방향 매핑을 사용하심이 좋습니다.**

대상

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;
    
    @ManyToOne
    @JoinColumn(name = "team_id", 
		insertable = false, 
		updatable = false)
    private Team team;
    
}
```

주인