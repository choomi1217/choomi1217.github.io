---
title:  "[자바 ORM 표준 JPA] ch6.다양한 연관관계 매핑 - 다대다"

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

# 다대다

DB에선 테이블 두개만으로 표현 할 수 없습니다.

그래서 중간에 **관계 테이블**을 추가합니다.

객체지향에선 객체 2개로 다대다 관계를 만들 수 있습니다.

### ➡️ 단방향

![Untitled](/assets/images/java-orm-jpa/6/6_3.png)

`@JoinTable` 의 속성

- @JoinTable.name : 연결 테이블의 이름입니다. 코드상으로는 `member_product` 가 됩니다.
- @JoinTable.joinColumns : `@JoinTable` 을 사용한 `Member` 에서 조인에 사용할 컬럼입니다.
- @JoinTable.inverseJoinColumns : `@JoinTable` 을 사용하지 않은 `Product` 에서 조인에 사용할 컬럼입니다.

```java
@Entity
@NoArgsConstructor
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String username;
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",
            joinColumns = @JoinColumn(name = "MEMBER_ID"),
            inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    private List<Product> products = new ArrayList<Product>();

  }
```

```java
@Entity
@NoArgsConstructor
public class Product {
    @Id @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private Long id;
    private String name;
}
```

### 코드 설명

- `@ManyToMany` 는 기본적으로 **지연로딩**입니다. 따라서 `Member` 를 조회할 땐 Join이 발생하지 않고 `Product` 를 사용하려는 순간 조인이 발생 됩니다.
- 트랜잭션 커밋후 1차 캐시를 비우지 않고 조회하면 1차 캐시에 있는 데이터를 조회하므로 Select 쿼리가 날아가지 않게 됩니다.

```java
public class _1_Test {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();

        Product note = new Product();
        note.setName("note");
        Product pen = new Product();
        pen.setName("pen");
        em.persist(note);
        em.persist(pen);

        Member member = new Member();
        member.setUsername("cho");
        member.getProducts().add(note);
        member.getProducts().add(pen);
        em.persist(member);

        tx.commit();
        em.clear(); 

        Member cho = em.find(Member.class, member.getId());
        System.out.println("----- " + cho.getUsername() + " 회원님의 주문상품 목록 -----");
        cho.getProducts().forEach(product -> {
            System.out.println("product = " + product.getName());
        });

        emf.close();
    }
}
```

- JPA LOG

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
            Product
            (name, PRODUCT_ID) 
        values
            (?, ?)
    Hibernate: 
        insert 
        into
            Product
            (name, PRODUCT_ID) 
        values
            (?, ?)
    Hibernate: 
        insert 
        into
            Member
            (username, MEMBER_ID) 
        values
            (?, ?)
    Hibernate: 
        insert 
        into
            MEMBER_PRODUCT
            (MEMBER_ID, PRODUCT_ID) 
        values
            (?, ?)
    Hibernate: 
        insert 
        into
            MEMBER_PRODUCT
            (MEMBER_ID, PRODUCT_ID) 
        values
            (?, ?)
    Hibernate: 
        select
            member0_.MEMBER_ID as member_i1_0_0_,
            member0_.username as username2_0_0_ 
        from
            Member member0_ 
        where
            member0_.MEMBER_ID=?
    ----- cho 회원님의 주문상품 목록 -----
    Hibernate: 
        select
            products0_.MEMBER_ID as member_i1_1_0_,
            products0_.PRODUCT_ID as product_2_1_0_,
            product1_.PRODUCT_ID as product_1_2_1_,
            product1_.name as name2_2_1_ 
        from
            MEMBER_PRODUCT products0_ 
        inner join
            Product product1_ 
                on products0_.PRODUCT_ID=product1_.PRODUCT_ID 
        where
            products0_.MEMBER_ID=?
    product = note
    product = pen
    ```


![Untitled](/assets/images/java-orm-jpa/6/6_4.png)

![Untitled](/assets/images/java-orm-jpa/6/6_5.png)

---

### ↔️ 양방향

양쪽 엔티티 모두 `@ManyToMany` 를 사용합니다.

```java
@Entity
@NoArgsConstructor
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",
            joinColumns = @JoinColumn(name = "MEMBER_ID"),
            inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    private List<Product> products = new ArrayList<Product>();

    //getter&setter
		
		//연관관계 편의 메소드
    public void addProduct(Product product) {
        products.add(product);
        if (!product.getMembers().contains(this)) {
            product.addMember(this);
        }
    }

}
```

```java
@Entity
@NoArgsConstructor
public class Product {

    @Id @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private Long id;

    private String name;

    @ManyToMany(mappedBy = "products")
    private List<Member> members = new ArrayList<Member>();

    //getter&setter
		
		//연관관계 편의 메소드
    public void addMember(Member member) {
        members.add(member);
        if (!member.getProducts().contains(this)) {
            member.addProduct(this);
        }
    }

}
```

```java
// 정방향 탐색
Member cho = em.find(Member.class, member.getId());
System.out.println("----- " + cho.getUsername() + " 회원님의 주문상품 목록 -----");
cho.getProducts().forEach(product -> {
    System.out.println("product = " + product.getName());
});

// 역방향 탐색
Product product = em.find(Product.class, note.getId());
System.out.println("-----" + product.getName() + " 상품을 구매한 회원 목록 -----");
product.getMembers().forEach(m -> {
    System.out.println("member = " + m.getUsername());
});
```

---

# 다대다 매핑의 한계, 연결 엔티티(관계 테이블)를 사용한 극복

`Member`  와 `Product` 의 관계테이블인 `member_product` 테이블에 날짜, 수량.. 등 컬럼들이 더 필요해지는 요구사항이 있을경우 엔티티로 관리되지 않으면 `member_product` 에 추가가 어렵기 때문에

**관계테이블을 엔티티로 관리하는 것이 좋습니다.**

- 복합키를 사용하는 방법
- 대리키를 사용하는 방법

![Untitled](/assets/images/java-orm-jpa/6/6_6.png)

### 💡 복합키를 사용한 관계 테이블 ( 식별관계 )

**부모의 PK(member_id, product_id)를 받아 본인의 PK이자 FK로 사용하는 것을 `식별관계`라고 합니다.**

### 코드 설명

- `Member`는 주인이 아닙니다. 회원 키를 FK로써 가지고 있는 `MemberProduct` 가 주인입니다. 고로 `Member` 는 `mappedBy` 속성을 사용해야 합니다.
- `Product` 에서 `Member` 로의 탐색 기능이 필요하지 않다고 판단해서 연관관계를 만들지 않음.
- `MemberProduct` 에는 PK를 매핑하는 `@Id` 와 FK를 매핑하는 `@JoinColumn` 이 같이 사용 됩니다.
  **외래키를 기본키로 사용합니다.**
- 또한 `@Id` 가 2개입니다. **복합키를 사용합니다.
  JPA에선 복합키를 사용하려면 별도의 클래스를 필요로 합니다.**
- `MemberProductId` 는 복합키 클래스로 `@IdClass` 에 명시 해줘야 하며, `Serializable` 을 구현해야 합니다.
- **조회를 관계 테이블 `MemberProduct`의 식별자 클래스 `MemberProductId` 로 해야합니다.**

```java
@Entity
@NoArgsConstructor
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts;
}
```

```java
@Entity
@NoArgsConstructor
public class Product {
    @Id @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private Long id;

    private String name;

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
@Entity
@IdClass(MemberProductId.class)
@NoArgsConstructor
public class MemberProduct {
    @Id
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @Id
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    private int orderAmount;

    private Date orderDate;
}

// 복합키 매핑 클래스
public class MemberProductId implements Serializable {
		//MemberProduct에 매핑시킬 객체와 이름이 같아야 합니다.
    private Long member; 
    private Long product;
}
```

```java
public class _1_Test {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();

				//회원등록
        Member member = new Member();
        member.setUsername("cho");
        em.persist(member);

				//상품등록
        Product note = new Product();
        note.setName("note");
        Product pen = new Product();
        pen.setName("pen");
        em.persist(note);
        em.persist(pen);

				//상품 주문
        MemberProduct memberProduct1 = new MemberProduct();
        memberProduct1.setMember(member);
        memberProduct1.setProduct(note);
        memberProduct1.setOrderAmount(2);
        memberProduct1.setOrderDate(new Date());
        MemberProduct memberProduct2 = new MemberProduct();
        memberProduct2.setMember(member);
        memberProduct2.setProduct(pen);
        memberProduct2.setOrderAmount(3);
        memberProduct2.setOrderDate(new Date());
        em.persist(memberProduct1);
        em.persist(memberProduct2);

        tx.commit();
        em.clear();

        //회원, 상품조회
        MemberProductId memberProductId1 = new MemberProductId();
        memberProductId1.setMember(member.getId());
        memberProductId1.setProduct(note.getId());

        MemberProductId memberProductId2 = new MemberProductId();
        memberProductId2.setMember(member.getId());
        memberProductId2.setProduct(pen.getId());

        MemberProduct findMemberProduct1 = em.find(MemberProduct.class, memberProductId1);
        MemberProduct findMemberProduct2 = em.find(MemberProduct.class, memberProductId2);

        System.out.println("회원명 : " + findMemberProduct1.getMember().getUsername());
        System.out.println("주문 아이템 : " + findMemberProduct1.getProduct().getName());
        System.out.println("주문 개수 : " +findMemberProduct1.getOrderAmount());
        System.out.println("주문 아이템 : " + findMemberProduct2.getProduct().getName());
        System.out.println("주문 개수 : " +findMemberProduct2.getOrderAmount());
    }
}
```

- JPA LOG

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
            (username, MEMBER_ID) 
        values
            (?, ?)
    Hibernate: 
        insert 
        into
            Product
            (name, PRODUCT_ID) 
        values
            (?, ?)
    Hibernate: 
        insert 
        into
            Product
            (name, PRODUCT_ID) 
        values
            (?, ?)
    Hibernate: 
        insert 
        into
            MemberProduct
            (orderAmount, orderDate, MEMBER_ID, PRODUCT_ID) 
        values
            (?, ?, ?, ?)
    Hibernate: 
        insert 
        into
            MemberProduct
            (orderAmount, orderDate, MEMBER_ID, PRODUCT_ID) 
        values
            (?, ?, ?, ?)
    Hibernate: 
        select
            memberprod0_.MEMBER_ID as member_i1_1_0_,
            memberprod0_.PRODUCT_ID as product_2_1_0_,
            memberprod0_.orderAmount as orderamo3_1_0_,
            memberprod0_.orderDate as orderdat4_1_0_,
            member1_.MEMBER_ID as member_i1_0_1_,
            member1_.username as username2_0_1_,
            product2_.PRODUCT_ID as product_1_2_2_,
            product2_.name as name2_2_2_ 
        from
            MemberProduct memberprod0_ 
        inner join
            Member member1_ 
                on memberprod0_.MEMBER_ID=member1_.MEMBER_ID 
        inner join
            Product product2_ 
                on memberprod0_.PRODUCT_ID=product2_.PRODUCT_ID 
        where
            memberprod0_.MEMBER_ID=? 
            and memberprod0_.PRODUCT_ID=?
    Hibernate: 
        select
            memberprod0_.MEMBER_ID as member_i1_1_0_,
            memberprod0_.PRODUCT_ID as product_2_1_0_,
            memberprod0_.orderAmount as orderamo3_1_0_,
            memberprod0_.orderDate as orderdat4_1_0_,
            member1_.MEMBER_ID as member_i1_0_1_,
            member1_.username as username2_0_1_,
            product2_.PRODUCT_ID as product_1_2_2_,
            product2_.name as name2_2_2_ 
        from
            MemberProduct memberprod0_ 
        inner join
            Member member1_ 
                on memberprod0_.MEMBER_ID=member1_.MEMBER_ID 
        inner join
            Product product2_ 
                on memberprod0_.PRODUCT_ID=product2_.PRODUCT_ID 
        where
            memberprod0_.MEMBER_ID=? 
            and memberprod0_.PRODUCT_ID=?
    회원명 : cho
    주문 아이템 : note
    주문 개수 : 2
    주문 아이템 : pen
    주문 개수 : 3
    ```


![Untitled](/assets/images/java-orm-jpa/6/6_7.png)

### 💡 대리키를 사용한 관계 테이블 ( 비식별 관계 )