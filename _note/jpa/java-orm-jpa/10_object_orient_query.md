# 객체지향 쿼리
`em.find()`는 식별자로만 객체를 탐색하기 때문에 한계가 있습니다. 
이를 위해 객체지향 쿼리를 사용합니다.

객체지향 쿼리는 객체를 대상으로 검색을하는 **객체지향 쿼리**입니다.
특정 SQL에 의존하지 않습니다.

- JPQL ( Java Persistance Query Language )
- Criteria 쿼리
  - JPQL을 편하게 작성할 수 있도록 도와주는 빌더 클래스
- QueryDSL
  - JPQL을 편하게 작성할 수 있도록 도와주는 빌더 클래스
- 네이티브 SQL
  - SQL을 직접 사용
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

# JPQL

### JPQL 기본
1. 엔티티와 속성은 대소문자를 구분합니다.
   - `SELECT m.name`은 정상실행 되지만 `SELECT m.Name` 대소문자가 틀리면 `QueryException` 이 발생 합니다
2. FROM 절에는 엔티티 이름이 들어갑니다.
3. 엔티티에 별칭은 필수 입니다.
```java
TypedQuery<String> query = em.createQuery("SELECT m.Name FROM Member m where m.age > 20", String.class);
-----
Caused by: org.hibernate.QueryException: **could not resolve property: Name** of: org.example.ch10.jpql.entity.Member [SELECT m.Name FROM org.example.ch10.jpql.entity.Member m where m.age > 20]
```
```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
List<Member> resultList = query.getResultList();
resultList.forEach(member -> System.out.println(member.getName()));
-----
Hibernate: 
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
```
```java
TypedQuery<String> query = em.createQuery("SELECT m.name FROM Member m where m.age > 20", String.class);
List<String> resultList = query.getResultList();
-----
Hibernate: 
    select
        member0_.name as col_0_0_ 
    from
        Member member0_ 
    where
        member0_.age>20
```

### TypeQuery, Query
- Query
- 쿼리의 결과물이 정해진 타입이 아닙니다.
- 결과가 여러개라면 Object[]로 반환되고, 하나라면 Object 입니다.
```java
Query query = em.createQuery("SELECT m.id, m.name FROM Member m");
  List resultList = query.getResultList();
  resultList.forEach(o -> {
      Object[] object = (Object[]) o;
      System.out.println("id: " + object[0]);
      System.out.println("name: " + object[1]);
  });
```
- TypeQuery
- 쿼리의 결과물이 정해진 객체입니다.
```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
List<Member> resultList = query.getResultList();
resultList.forEach(member -> System.out.println(member.getName()));
```

### 파라미터 바인딩
JPQL은 이름 기분 파라미터 바인딩을 지원합니다.
`WHERE m.name = :name` 이 부분에 `:name` 이 파라미터 바인딩 입니다.
파라미터를 셋팅 할 때 `name`이란 이름으로 바인딩하면 됩니다.
```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m WHERE m.name = :name", Member.class);
query.setParameter("name", "kim");
List<Member> resultList = query.getResultList();
resultList.forEach(member -> System.out.println(member.getName()));
``` 
