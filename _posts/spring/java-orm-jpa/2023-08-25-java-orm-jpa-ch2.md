---
title:  "[자바 ORM 표준 JPA] ch2.JPA 시작"

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


# 2장 JPA시작

### JAP 설정

```yaml
spring:
  jpa:
    database: postgresql
    show-sql: true
    hibernate:
      ddl-auto: update
  datasource:
    username: postgres
    password: 
    url: jdbc:postgresql://localhost:5432/postgres
    hikari:
      schema: jpa
```

### Dialect

책에서는 **방언**이라고 DB 종류마다 JPA에 설정을 해야한다고 했지만

저는 지금까지 한번도 설정 하지 않고 잘 작동해서 찾아본 결과, 해당 설정은 하지 않기로 했습니다

[[Spring boot] JPA Dialect(방언) 설정에 관하여](https://velog.io/@lehdqlsl/Spring-boot-JPA-Dialect방언-설정에-관하여)

---

### 시작

```java
public class Main {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("member");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();
        logic(em);
        tx.commit();
        
        emf.close();
    }

    private static void logic(EntityManager em) {
        // business logic
    }
}
```

- 엔티티 매니저 설정
- 트랜잭션 관리
- 비즈니스 로직

### 엔티티 매니저 설정

1. EntityManagerFactory
  1. JPA를 시작하려면 JPA 설정 정보를 이용해 `EntityManagerFactory` 를 생성해야 합니다.

   *`jakarta.persistence.Persistence` 를 이용해*  `EntityManagerFactory` 를 생성합니다.

   **JPA 구현체에 따라 DBCP 도 생성하므로 생성비용이 큰 과정이므로 애플리케이션에서 하나만 생성해 사용하도록 합니다.**

2. EntityManager
  1. **EntityManager를 사용해 CRUD가 가능합니다.** 내부에 데이터 소스( 데이터베이스 커넥션 ) 을 유지하고 있습니다. 데이터베이스 커넥션과 밀접한 관계가 있어서 스레드 간에 공유나 재사용은 지양합니다.

### 트랜잭션 관리

JPA를 사용하려면 항상 트랜잭션 안에서 데이터를 변경 해야합니다. 트랜잭션 없이 변경하면 예외가 발생합니다.

### 비즈니스 로직

회원 엔티티 하나를 생성한 후 CRUD를 하는 과정입니다.

CRUD가 전부 엔티티 매니저(em)을 통해 수행 됩니다.

```java
private static void logic(EntityManager em) {

        Member member = new Member();
        member.setName("cho");
        member.setAge(10);

        em.persist(member);

        member.setAge(20);
        Long id = member.getId();

        Member findMember = em.find(Member.class, id);
        System.out.println("findMember = " + findMember.getName() + ", age = " + findMember.getAge());

        List<Member> members = em.createQuery("select m from Member m", Member.class)
                .getResultList();
        System.out.println("members.size() = " + members.size());

        em.remove(member);
    }
```

```yaml
Hibernate: 
    select
        nextval ('hibernate_sequence')
findMember = cho, age = 20
```

```yaml

Hibernate: 
    insert 
    into
        member
        (age, name, id) 
    values
        (?, ?, ?)
```

```yaml
Hibernate: 
    update
        member 
    set
        age=?,
        name=? 
    where
        id=?
```

```yaml
Hibernate: 
    select
        member0_.id as id1_0_,
        member0_.age as age2_0_,
        member0_.name as name3_0_ 
    from
        member member0_
members.size() = 1
```

```yaml
Hibernate: 
    delete 
    from
        member 
    where
        id=?
```

- 등록
  - `em.persist(member)`
- 수정
  - `member.setAge(20)` 단순히 엔티티의 값을 변경 하는 것만으로 Update가 됩니다.
- 삭제
  - `em.remove(member)`
- 조회
  - `em.find(Member.class, id)`
  - `em.createQuery("select m from Member m", Member.class).getResultList()`
    - 이건 **JPQL** 이란걸 사용 한 겁니다.

---

### 🤔 근데 콘솔은 왜 순서대로 안 찍혀요?

### ✨지연 로딩(Delayed Loading) & **지연 쓰기 (Dirty Checking & Write Behind)**

JPA는 효율적인 데이터베이스 연산을 위해 지연 로딩과 지연 쓰기 전략을 사용합니다.

1. **지연 로딩 (Lazy Loading):** JPA는 조회 쿼리를 보내지 않고 대신 프록시 객체를 반환하여 데이터베이스에 대한 조회를 최대한 늦추는 전략입니다. 실제로 데이터를 사용하는 시점에 비로소 조회 쿼리를 보냅니다.
2. **지연 쓰기 (Dirty Checking & Write Behind):** JPA는 트랜잭션 커밋 시점에 변경 감지 (Dirty Checking)를 통해 변경된 엔티티를 찾고, 이들에 대한 쿼리를 한 번에 데이터베이스에 보내는 전략입니다.

1.  **`em.persist(member);`** 가 실행되지만, 이 시점에서는 데이터베이스에 저장하는 쿼리를 바로 보내지 않고 대신 이를 캐싱합니다.
2. 그 후 **`member.setAge(20);`** 실행 후에도 쿼리는 바로 실행되지 않습니다.
3. **`em.find(Member.class, id);`** 는 member의 id가 필요합니다. 이때 id는 **`persist()`**가 실행될 때 hibernate_sequence를 통해 생성됩니다. 따라서 이 시점에서 "select nextval ('hibernate_sequence')" 쿼리가 실행됩니다.
4. **`System.out.println("findMember = " + findMember.getName() + ", age = " + findMember.getAge());`** 이 출력됩니다.
5. 이후 **`em.createQuery("select m from Member m", Member.class).getResultList();`** 이 실행되기 직전에 변경된 member에 대한 쿼리 (INSERT, UPDATE)가 데이터베이스에 전송됩니다.
6. 마지막으로 **`em.remove(member);`** 실행시 DELETE 쿼리가 실행됩니다.

이렇게 JPA는 트랜잭션의 성능을 최적화하기 위해 여러 가지 전략을 사용합니다.

때문에 코드의 순서와 실제 쿼리가 실행되는 순서가 다를 수 있습니다.

---

### JPQL

모든 SQL은 JPA가 알아서 만듭니다.

하지만 애플리케이션을 개발하다가 보면 그렇지 못한 상황들이 더 많이 발생합니다.

엔티티 객체를 대상으로 검색하려면 DB의 모든 데이터를 불러와서 엔티티 객체로 변경 후 검색 해야하는데 이는 불가능한 일입니다.

결국 검색 조건이 포함된 SQL을 사용하게 되는데 이때 JPQL을 사용합니다.

JPQL은 엔티티 객체를 대상으로 쿼리 한다는 점이 SQL과의 차이점입니다.