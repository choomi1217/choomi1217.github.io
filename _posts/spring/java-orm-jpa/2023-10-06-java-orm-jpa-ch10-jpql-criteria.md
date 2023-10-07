---
title:  "[자바 ORM 표준 JPA] ch10.객체지향 쿼리 - JPQL Criteria편 "

categories:
  java-orm-jpa
tags:
  - [java-orm-jpa]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-10-06
last_modified_at: 2023-10-06
---

JPQL을 자바 코드로 작성할 수 있게 도와주는 빌더 클래스 API입니다.
- 문법 오류를 컴파일 단계에서 잡을 수 있습니다.
- 코드가 복잡하다는 단점이 있습니다.
### 기본
1. Criteria Builder를 생성합니다.
2. Builder로부터 Query를 생성합니다. 이때 반환 타입을 매개변수로 전달합니다.
3. From을 생성합니다. 반환된 값으로부터 select를 할 필드를 정하기 때문에 'Root'라고 합니다.
4. 조회 Query를 생성합니다.
5. Query를 entityManager에게 전달 후 결과를 조회합니다.

   **빌더 → 쿼리 → 루트 → 조회 → 엔티티매니저** 이렇게 생략할 수 있습니다.

```java
public class CriteriaBase {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
        EntityManager em = emf.createEntityManager();
        // 1
        CriteriaBuilder cb = em.getCriteriaBuilder();
        // 2
        CriteriaQuery<Member> cq = cb.createQuery(Member.class);
        // 3
        Root<Member> from = cq.from(Member.class);
        // 4
        cq.select(from);
        // 5
        List<Member> resultList = em.createQuery(cq).getResultList();
        resultList.forEach(member -> {
            System.out.println(member.getId() + "번 " + member.getName());
        });

    }
}
```
### 조건 조회
**빌더 → 쿼리 → 루트 → 조건 생성 → 조회 → 엔티티매니저**
기본적으로 빌더, 쿼리, 루트까지는 같습니다.
조회시 조건이 들어가야 하므로 조건을 만들어야 합니다.
빌더에게 루트를 매개변수로 전달하고 조건을 만들 수 있습니다.
조건을 생성하는 방법이 컬럼의 타입에 따라 조금씩 다릅니다.
```java
public class CriteriaBase {
  public static void main(String[] args) {
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
    EntityManager em = emf.createEntityManager();
    
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Member> cq = cb.createQuery(Member.class);
    Root<Member> from = cq.from(Member.class);
    
    // 조건 생성
    Predicate nameEqual = cb.equal(from.get("name"), "cho");
    Order idDesc = cb.desc(from.get("id"));
    Predicate ageGreateredThan = cb.greaterThan(from.<Integer>get("age"), 10);

    // 조건 전달
    cq.select(from).where(nameEqual, ageGreateredThan).orderBy(idDesc);

    List<Member> resultList = em.createQuery(cq).getResultList();
    resultList.forEach(member -> {
      System.out.println(member.getId() + "번 " + member.getName());
    });

  }
}
```