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
4. `query.getResultList()`는 결과를 리스트로 반환합니다. 결과가 없으면 빈 리스트를 반환합니다.
5. `query.getSingleResult()`는 결과가 정확히 하나일 때 사용합니다. 결과가 없으면 `NoResultException` 예외가 발생하고, 둘 이상이면 `NonUniqueResultException` 예외가 발생합니다.
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
1. 이름 기준 파라미터
JPQL은 **이름 기준 파라미터 바인딩**을 지원합니다.
`WHERE m.name = :name` 이 부분에 `:name` 이 파라미터 바인딩 입니다.
파라미터를 셋팅 할 때 `name`이란 이름으로 바인딩하면 됩니다.
```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m WHERE m.name = :name", Member.class);
query.setParameter("name", "kim");
``` 
2. 위치 기준 파라미터
위치 기준 파라미터는 `?`를 사용합니다.
```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m WHERE m.name = ?1", Member.class);
query.setParameter(1, "kim");
```

### 프로젝션
SELECT 절에 조회활 대상을 지정하는 것을 프로젝션이라고 합니다.
1. 엔티티 프로젝션
아래 sql 둘 다 원하는 엔티티를 바로 조회합니다.
이렇게 조회한 엔티티는 영속성 컨텍스트에서 관리됩니다.
```text
List<Member> resultList = em.createQuery("SELECT m FROM Member m", Member.class).getResultList();
```
2. 임베디드 타입 프로젝션
임베디드 타입은 엔티티처럼 FROM절에 사용할 수 없습니다.
임베디드가 포함된 엔터티를 조회하되 임베디드 타입만을 조회하면 됩니다.
```text
List<Address> resultList = em.createQuery("SELECT o.address FROM Order o", Address.class).getResultList();
```
3. 스칼라 타입 프로젝션
숫자, 문자, 날짜와 같은 기본 데이터 타입입니다.
```text
List<String> resultList = em.createQuery("SELECT m.name FROM Member m", String.class).getResultList();
List<Double> ageAverage = em.createQuery("SELECT AVG(m.age) FROM Member m", Double.class).getResultList();
```
4. 여러 값 조회
여러 값 조회는 `Object[]` 타입으로 조회합니다.
```text
List<Object[]> resultList = em.createQuery("SELECT o.member, o.product, o.orderAmount FROM Order o").getResultList();
resultList.forEach(objects -> {
    Member member = (Member) objects[0];
    System.out.println("member = " + member.getName());
    Product product = (Product) objects[1];
    System.out.println("product = " + product.getName());
    int orderAmount = (int) objects[2];
    System.out.println("orderAmount = " + orderAmount);
});
```
5. NEW 명령어
생성자를 통해서 조회 결과를 바로 해당 객체로 받을 수 있습니다.
```text
List<MemberDto> resultList = em.createQuery("SELECT new org.example.ch10.jpql.dto.MemberDto(m.name, m.age) FROM Member m", MemberDto.class).getResultList();
```

### 페이징 API
이 두개만 설정하면 됩니다.
`setFirstResult(int startPosition)`, `setMaxResults(int maxResult)`
```java
List<Member> resultList = em.createQuery("SELECT m FROM Member m", Member.class)
  .setFirstResult(0)
  .setMaxResults(10)
  .getResultList();
```
```text
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
        **Member member0_ limit ? offset ?**
```

### 조인
