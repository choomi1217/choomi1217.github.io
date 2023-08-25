---
title:  "[자바 ORM 표준 JPA] ch6.다양한 연관관계 매핑 - 다대일"

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

# 다대일 

외래 키는 항상 다에 있습니다. 그러므로 연관관계의 주인은 항상 다입니다.

### ➡️단방향

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;
    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;

		//getter&setter
}
```

```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;
    private String name;

		//getter&setter
}
```

### ↔️ 양방향

외래키가 있는 `Member` 가 연관관계의 주인입니다.

**양방향은 서로 참조 할 수 있어야 합니다.**

**서로 참조하려면 연관관계 편의 메소드를 추가 하는 것이 좋습니다.**

`Member` 에는 `setTeam` , `Team` 에는 `addMember` 가 편의 메소드입니다.

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;

		//연관관계 편의 메소드
		public void setTeam(Team team) {
        this.team = team;
        if(!team.getMembers().contains(this)) {
            team.getMembers().add(this);
        }
    }
}
```

```java
@Entity
public class Team {

    @Id 
		@GeneratedValue
    @Column(name = "team_id")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();

		//연관관계 편의 메소드
    public void addMember(Member member) {
        this.members.add(member);
        if(member.getTeam() != this) {
            member.setTeam(this);
        }
    }
}
```