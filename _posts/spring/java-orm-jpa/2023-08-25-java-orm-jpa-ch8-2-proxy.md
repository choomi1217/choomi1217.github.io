---
title:  "[자바 ORM 표준 JPA] ch8.프록시와 연관관계 관리 - 프록시"

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

# 프록시

`printMember()` 는 `member` 만 사용하므로 `team` 까지 미리 조회해 두는 건 좋지 않습니다.

이 문제를 해결하기 위해 **지연로딩**을 제공합니다.

`member.getTeam()` 처럼 실제 값을 사용할 때 DB에서 내용을 조회 합니다.

**지연로딩을 사용하기 위해선 실제 엔티티 객체 대신 DB 조회를 지연할 수 있는 가짜 엔티티 객체가 필요한데 이것을 프록시 객체라고 합니다.**

프록시는 JPA 구현체가 구현해야하고 책은 하이버네이트 기준으로 설명합니다.

```java
private static void printMember(EntityManager em) {
    Member member = em.find(Member.class, "cho");
    System.out.println("member's name = " + member.getUsername());
}

private static void printUserAndTeam(EntityManager em) {
    Member member = em.find(Member.class, "cho");
    System.out.println("member's name = " + member.getUsername());
    System.out.println("team's name = " + member.getTeam().getName());
}
```

### 프록시 기초

식별자로 엔티티 하나를 조회할 땐 `em.find()` 를 사용합니다. 이렇게 할 경우엔 DB를 조회합니다.

엔티티 사용하는 시점까지 DB 조회를 미루려면 `em.getReference()` 를 하면 됩니다.

`em.getReference()` 를 하면 JPA는 **DB를 조회하지 않고 DB 접근을 위임한 프록시 객체를 반환합니다.**

```java
Member member = em.find(Member.class, "member1");
Member proxyMember = em.getReference(Member.class, "cho");
```

- 프록시 특징

프록시 객체는 실제 객체에 대한 참조이므로 프록시의 메소드를 호출하면 실제 객체의 메소드가 호출 됩니다.

1. 처음 사용할 때 한 번만 초기화 됩니다.
2. 초기화를 해서 프록시 객체가 실제 엔티티가 되는 것은 아니고 프록시 객체를 통해 엔티티에 접근 할 수 있습니다.
3. 초기화는 영속성 컨텍스트의 도움이 있어야 합니다. 그러므로 준영속 상태의 프록시를 초기화하면 `org.hibernate.LazyInitializationException` 이 발생합니다.
  - source

      ```java
      // 준영속 상태가 된 프록시 객체에 초기화를 요청하면 에러가납니다.
      Member proxyMember = em.getReference(Member.class, "cho");
      em.close();
      proxyMember.getUsername();
      emf.close();
      
      -----
      
      Exception in thread "main" org.hibernate.LazyInitializationException: could not initialize proxy [org.example.ch8.proxy.Member#cho] - no Session
      ```


[Java에서 구현할 수 있는 Proxy들 (Pure, JDK, CGLIB)](https://lob-dev.tistory.com/entry/Java에서-구현할-수-있는-Proxy-들-Pure-JDK-CGLIB)

- 프록시 객체의 초기화

`member.getName()` 처럼 실제 사용될 때 DB를 조회해서 실제 엔티티 객체를 생성합니다.

![Untitled](docs/assets/images/java-orm-jpa/8/8_1.png)

### 프록시와 식별자

- 엔티티를 프록시로 조회할 때 식별자를 전달하는데 이때 프록시는 식별자 값을 보관하고 조회시 보관된 식별자를 반환하므로 DB 조회가 일어나지 않습니다.
  - source

      ```java
      // 식별자가 보관되고 초기화 되지 않음
        Team teamA = em.getReference(Team.class, "A");
        System.out.println(teamA.getId());
      
      -----
      
      Aug 15, 2023 4:30:08 PM org.hibernate.jpa.internal.util.LogHelper logPersistenceUnitInformation
      INFO: HHH000204: Processing PersistenceUnitInfo [name: myApp]
      Aug 15, 2023 4:30:08 PM org.hibernate.Version logVersion
      INFO: HHH000412: Hibernate ORM core version 5.4.32.Final
      Aug 15, 2023 4:30:08 PM org.hibernate.annotations.common.reflection.java.JavaReflectionManager <clinit>
      INFO: HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
      Aug 15, 2023 4:30:08 PM org.hibernate.engine.jdbc.connections.internal.DriverManagerConnectionProviderImpl configure
      WARN: HHH10001002: Using Hibernate built-in connection pool (not for production use!)
      Aug 15, 2023 4:30:08 PM org.hibernate.engine.jdbc.connections.internal.DriverManagerConnectionProviderImpl buildCreator
      INFO: HHH10001005: using driver [org.postgresql.Driver] at URL [jdbc:postgresql://localhost:5432/postgres?currentSchema=jpa]
      Aug 15, 2023 4:30:08 PM org.hibernate.engine.jdbc.connections.internal.DriverManagerConnectionProviderImpl buildCreator
      INFO: HHH10001001: Connection properties: {password=****, user=postgres}
      Aug 15, 2023 4:30:08 PM org.hibernate.engine.jdbc.connections.internal.DriverManagerConnectionProviderImpl buildCreator
      INFO: HHH10001003: Autocommit mode: false
      Aug 15, 2023 4:30:08 PM org.hibernate.engine.jdbc.connections.internal.DriverManagerConnectionProviderImpl$PooledConnections <init>
      INFO: HHH000115: Hibernate connection pool size: 20 (min=1)
      Aug 15, 2023 4:30:08 PM org.hibernate.dialect.Dialect <init>
      INFO: HHH000400: Using dialect: org.hibernate.dialect.PostgreSQL10Dialect
      Aug 15, 2023 4:30:08 PM org.hibernate.resource.transaction.backend.jdbc.internal.DdlTransactionIsolatorNonJtaImpl getIsolatedConnection
      INFO: HHH10001501: Connection obtained from JdbcConnectionAccess [org.hibernate.engine.jdbc.env.internal.JdbcEnvironmentInitiator$ConnectionProviderJdbcConnectionAccess@2b289ac9] for (non-JTA) DDL execution was not in auto-commit mode; the Connection 'local transaction' will be committed and the Connection will be set into auto-commit mode.
      Aug 15, 2023 4:30:09 PM org.hibernate.engine.transaction.jta.platform.internal.JtaPlatformInitiator initiateService
      INFO: HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
      A
      Aug 15, 2023 4:30:09 PM org.hibernate.engine.jdbc.connections.internal.DriverManagerConnectionProviderImpl$PoolState stop
      INFO: HHH10001008: Cleaning up connection pool [jdbc:postgresql://localhost:5432/postgres?currentSchema=jpa]
      ```

- `Property` 는 초기화를 진행하지 않지만, `Field` 는 초기화를 진행합니다.
  - 잘 모르겠어요... 😢 왜 Field 일 때 초기화를 진행 하는지..
    `@Access(AccessType.FIELD)일 땐 getId() 메소드가 id만 조회하는 메소드인지 다른 필드까지 활용해서 어떤 일을 하는 메소드인지 알지 못해서 프록시 객체를 초기화 한다.` 라고 책에 적혀 있는데 전혀 이해가 안됩니다. 반대라고 생각했어요 (Propery) 일 때 초기화가 되어야 한다고 생각했는데.. 그러고 아래 소스코드를 실행 해보면 둘 다 DB 조회는 일어나지 않는 것 같아서 더 혼란스럽습니다. 😵
  - source

      ```java
      public static void main(String[] args) {
          EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
          EntityManager em = emf.createEntityManager();
          EntityTransaction tx = em.getTransaction();
      
          Team teamA = em.getReference(Team.class, "A");
          System.out.println("id : " + teamA.getId());
          System.out.println("class : " + teamA.getClass());
      
          emf.close();
      }
      
      -----
      
      id : A
      class : class org.example.ch8.proxy.accessType.field.Team$HibernateProxy$mFw53v4i
      ```

      ```java
      public static void main(String[] args) {
          EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
          EntityManager em = emf.createEntityManager();
          EntityTransaction tx = em.getTransaction();
      
          Team teamA = em.getReference(Team.class, "A");
          System.out.println("id : " + teamA.getId());
          System.out.println("class : " + teamA.getClass());
      
          emf.close();
      }
      
      -----
      
      id : A
      class : class org.example.ch8.proxy.accessType.property.Team$HibernateProxy$XSeNbh7u
      ```

- 연관관계를 설정할 때 유용하게 사용 할 수 있습니다.

```java
Member member = em.find(Member.class, "cho");
Team proxyTeam = em.getReference(Team.class, "A");
member.setTeam(proxyTeam);
```

### 프록시 확인

```java
Member proxyMember = em.getReference(Member.class, "cho");

System.out.println("isInitialized = " + emf.getPersistenceUnitUtil().isLoaded(proxyMember));
proxyMember.getUsername();
System.out.println("isInitialized = " + emf.getPersistenceUnitUtil().isLoaded(proxyMember));

-----

isInitialized = false
Hibernate: 
    select
        member0_.MEMBER_ID as member_i1_0_0_,
        member0_.team_TEAM_ID as team_tea3_0_0_,
        member0_.username as username2_0_0_,
        team1_.TEAM_ID as team_id1_1_1_,
        team1_.name as name2_1_1_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.team_TEAM_ID=team1_.TEAM_ID 
    where
        member0_.MEMBER_ID=?
isInitialized = true
```