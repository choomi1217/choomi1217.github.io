---
title:  "[자바 ORM 표준 JPA] ch1.JPA 소개"

categories:
  - java-orm-jpa
tags:
  - [java-orm-jpa]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

# 1장 JPA 소개

Object와 Relation은 서로 지향하는 패러다임이 너무 다릅니다.

이 차이를 극복하고자 개발자가 많은 시간과 코드를 소비하고 객체 모델링을 포기하고 데이터 중심의 모델로 변합니다.

이 문제를 해결하고자 JPA를 사용합니다.

### JPA란?

자바 ORM 기술 표준입니다.

### 왜 사용해야 할까?

- 자바 컬렉션에 객체를 저장하듯 JPA 전달하면 지루한 쿼리들을 JPA가 대신 처리합니다.
- SQL을 직접 다루면 SQL도 유지보수 해야하지만 JPA는 유지보수 해야하는 코드 수를 줄여줍니다.
- 아래와 같은 패러다임 문제들을 해결합니다.
- 한번 조회한 객체를 재사용 할 수 있습니다.
- DB는 종류에 따라 사용되는 방법이 다릅니다. 즉, DB 벤더에 독립적입니다.

### ORM프레임워크란?

SQL을 개발자 대신 생성해서 DB에 전달, 아래 문제들도 해결해줍니다.

객체는 객체 모델링을 관계형 DB는 관계형 DB 모델링을 하고 이 둘의 맵핑 방법만을 ORM 프레임워크에게 전달해주면 됩니다.

### SQL을 직접 다룰 때 발생하는 문제점

- 객체를 데이터베이스에 CRUD 하려면 너무 많은 SQL과 JDBC API를 코드로 작성 해야 한다.
- 계층 분할이 어려움
    - 컬럼 추가시 CRUD 전체가 다 수정 되어야 한다. ( 엔티티와 SQL의 강한 의존 관계 )
- 엔티티를 신뢰 할 수가 없음
- SQL에 의존적인 코드
    - SQL을 숨기려 해도 어쩔 수 없이 SQL에 문제가 없는지 직접 열어서 확인 해야 한다.

### 패러다임의 불일치

- DB와 객체지향의 차이
    - 비즈니스 요구사항을 정의한 도메인 모델로 객체를 모델링하고 싶지만 **상속, 다형성, 추상화**는 DB에 반영 할 수 없다.

### JPA와 상속

- JPA는 상속이 관련된 객체를 DB에서 이용 할 수 있도록 해준다.

### JPA와 연관관계

- 객체는 참조를 이용해 다른 객체와 연관관계를 가지는데 테이블은 FK를 이용해 조인합니다.
    - **객체는 참조만 필요하고 DB는 외래키만 있으면 됩니다.**
- JPA는 객체에 참조 필드를 이용해 FK 조인을 시도합니다.
- 객체는 참조가 있는 방향으로만 조회가 가능합니다.
    - Member 객체에 Team 객체를 참조했다면 Member에서 Team 조회가 가능하고 반대는 불가능합니다

### 객체를 테이블에 맞추어 모델링 👎

- 아래 코드처럼 테이블에 맞춘 객체를 만들면 `teamId` 부분에서 `teamId` 를 이용해 다시 조회 해야합니다.

```java
class Member {
	String id;
	Long teamId;     // team id fk 
	String username;
}

class Team {
	Long id;
	String name;
}
```

### 객체지향 모델링 👍

- 그냥 `Team team` 이렇게 참조를 사용하는게 JPA에선 가능합니다

```java
class Member {
	String id;
	Team team;     // team id fk 
	String username;
}

class Team {
	Long id;
	String name;
}
```

### 객체 그래프 탐색이란

객체에서 참조를 사용해서 연관객체를 찾는 것.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5a1960f8-fa30-4a05-9231-13d34103b63a/Untitled.png)

위와 같은 설계에서 객체 그래프를 탐색하면 아래와 같이 자유롭게 객체 그래프를 탐색 할 수 있어야 합니다.

객체라면 충분히 가능해야 합니다.

```java
member.getOrder().getOrderItem().getItem().getCategory(); 
```

**그런데 아래와 같은 쿼리가 실행 됐다면 그것이 가능할까요?**

```sql
SELECT M.*, T.* FROM MEMBER M JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
```

SQL을 직접 다루면 객체 그래프를 다룰 수 있습니다. 

하지만, 어디까지 사용할지 모르는데 **어디서 끊어질지 모르는 객체 그래프를 사용할 순 없습니다.**

결국 어디까지 사용 가능한 객체 그래프인지 SQL문을 열어야 개발자는 확인 할 수 있습니다.

### JPA의 객체 그래프 ( 지연 로딩 )

JPA는 연관된 객체를 사용하는 시점에서 적절한 SELECT SQL을 실행합니다.

실제 객체를 사용하는 시점까지 DB 조회를 미룬다고 해서 **지연 로딩** 이라고 합니다.

### 데이터 비교 방법

- DB는 PK로 각 row를 비교 합니다.
- 객체는 객체 내부 값 비교(동등성), 객체 주소 값 비교(동일성)가 있습니다.

만약 아래 구문이 DB였다면 같은 row, 같은 PK라서 true 였을텐데 객체는 false를 반환합니다. 

하지만 JPA**는 같은 트랜잭션일 때 같은 객체임을 보장**합니다.

```java
// 타 orm
String memberId = "100";
Member member1 = memberDao.getMember(memberId);
Member member2 = memberDao.getMember(memberId);

member1 == member2; //false

// jpa orm
String memberId = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2; //true
```

---

이번 장에선 JPA 상속에 대해선 서론에서 자세히 다루지 않으므로 궁금하다면 밑에 참고