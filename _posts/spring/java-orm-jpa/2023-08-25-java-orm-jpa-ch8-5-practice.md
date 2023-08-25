---
title:  "[자바 ORM 표준 JPA] ch8.프록시와 연관관계 관리 - 연습문제"

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

# 연관관계 관리
1. `Purchase` - `Member` : 다대일, lazy 설정
2. `Purchase` - `Delivery` : 일대일, lazy 설정, cascade 설정
3. `Purchase` - `PurchaseItem` : 일대다, cascade 설정
4. `PurchaseItem` - `Purchase` : 다대일, lazy 설정
5. `PurchaseItem` - `Item` : 다대일, lazy 설정

### 변경전
```text
----- cho 회원님의 주문내역 -----
Hibernate: 
    select
        purchases0_.MEMBER_ID as member_i7_5_0_,
        purchases0_.PURCHASE_ID as purchase1_5_0_,
        purchases0_.PURCHASE_ID as purchase1_5_1_,
        purchases0_.createdAt as createda2_5_1_,
        purchases0_.lastModifiedAt as lastmodi3_5_1_,
        purchases0_.DELIVERY_ID as delivery6_5_1_,
        purchases0_.MEMBER_ID as member_i7_5_1_,
        purchases0_.purchaseDate as purchase4_5_1_,
        purchases0_.status as status5_5_1_,
        delivery1_.DELIVERY_ID as delivery1_2_2_,
        delivery1_.createdAt as createda2_2_2_,
        delivery1_.lastModifiedAt as lastmodi3_2_2_,
        delivery1_.city as city4_2_2_,
        delivery1_.status as status5_2_2_,
        delivery1_.street as street6_2_2_,
        delivery1_.zipcode as zipcode7_2_2_ 
    from
        Purchase purchases0_ 
    left outer join
        Delivery delivery1_ 
            on purchases0_.DELIVERY_ID=delivery1_.DELIVERY_ID 
    where
        purchases0_.MEMBER_ID=?
* 주문번호 : 309
* 주문상태 : PURCHASE
* 주문날짜 : 2023-08-23 13:54:44.068
Hibernate: 
    select
        purchaseit0_.PURCHASE_ID as purchase5_6_0_,
        purchaseit0_.PURCHASE_ITEM_ID as purchase1_6_0_,
        purchaseit0_.PURCHASE_ITEM_ID as purchase1_6_1_,
        purchaseit0_.count as count2_6_1_,
        purchaseit0_.ITEM_ID as item_id4_6_1_,
        purchaseit0_.PURCHASE_ID as purchase5_6_1_,
        purchaseit0_.purchasePrice as purchase3_6_1_,
        item1_.ITEM_ID as item_id2_3_2_,
        item1_.createdAt as createda3_3_2_,
        item1_.lastModifiedAt as lastmodi4_3_2_,
        item1_.name as name5_3_2_,
        item1_.price as price6_3_2_,
        item1_.stockQuantity as stockqua7_3_2_,
        item1_.artist as artist8_3_2_,
        item1_.author as author9_3_2_,
        item1_.isbn as isbn10_3_2_,
        item1_.actor as actor11_3_2_,
        item1_.director as directo12_3_2_,
        item1_.DTYPE as dtype1_3_2_ 
    from
        PurchaseItem purchaseit0_ 
    left outer join
        Item item1_ 
            on purchaseit0_.ITEM_ID=item1_.ITEM_ID 
    where
        purchaseit0_.PURCHASE_ID=?
* 상품명 : album
Hibernate: 
    select
        categoryit0_.ITEM_ID as item_id2_1_0_,
        categoryit0_.CATEGORY_ID as category1_1_0_,
        categoryit0_.CATEGORY_ID as category1_1_1_,
        categoryit0_.ITEM_ID as item_id2_1_1_,
        category1_.CATEGORY_ID as category1_0_2_,
        category1_.createdAt as createda2_0_2_,
        category1_.lastModifiedAt as lastmodi3_0_2_,
        category1_.name as name4_0_2_,
        category1_.parentId as parentid5_0_2_ 
    from
        CategoryItem categoryit0_ 
    inner join
        Category category1_ 
            on categoryit0_.CATEGORY_ID=category1_.CATEGORY_ID 
    where
        categoryit0_.ITEM_ID=?
* 상품 카테고리 : lineNote
* 상품가격 : 10000
* 상품수량 : 1
* 총 가격 : 10000
-----
```

### 변경후
```text
----- cho 회원님의 주문내역 -----
Hibernate: 
    select
        purchases0_.MEMBER_ID as member_i7_5_0_,
        purchases0_.PURCHASE_ID as purchase1_5_0_,
        purchases0_.PURCHASE_ID as purchase1_5_1_,
        purchases0_.createdAt as createda2_5_1_,
        purchases0_.lastModifiedAt as lastmodi3_5_1_,
        purchases0_.DELIVERY_ID as delivery6_5_1_,
        purchases0_.MEMBER_ID as member_i7_5_1_,
        purchases0_.purchaseDate as purchase4_5_1_,
        purchases0_.status as status5_5_1_ 
    from
        Purchase purchases0_ 
    where
        purchases0_.MEMBER_ID=?
* 주문자 : cho
* 주문번호 : 327
* 주문상태 : PURCHASE
* 주문날짜 : 2023-08-23 14:09:55.244
Hibernate: 
    select
        purchaseit0_.PURCHASE_ID as purchase5_6_0_,
        purchaseit0_.PURCHASE_ITEM_ID as purchase1_6_0_,
        purchaseit0_.PURCHASE_ITEM_ID as purchase1_6_1_,
        purchaseit0_.count as count2_6_1_,
        purchaseit0_.ITEM_ID as item_id4_6_1_,
        purchaseit0_.PURCHASE_ID as purchase5_6_1_,
        purchaseit0_.purchasePrice as purchase3_6_1_ 
    from
        PurchaseItem purchaseit0_ 
    where
        purchaseit0_.PURCHASE_ID=?
Hibernate: 
    select
        item0_.ITEM_ID as item_id2_3_0_,
        item0_.createdAt as createda3_3_0_,
        item0_.lastModifiedAt as lastmodi4_3_0_,
        item0_.name as name5_3_0_,
        item0_.price as price6_3_0_,
        item0_.stockQuantity as stockqua7_3_0_,
        item0_.artist as artist8_3_0_,
        item0_.author as author9_3_0_,
        item0_.isbn as isbn10_3_0_,
        item0_.actor as actor11_3_0_,
        item0_.director as directo12_3_0_,
        item0_.DTYPE as dtype1_3_0_ 
    from
        Item item0_ 
    where
        item0_.ITEM_ID=?
* 상품명 : album
Hibernate: 
    select
        categoryit0_.ITEM_ID as item_id2_1_0_,
        categoryit0_.CATEGORY_ID as category1_1_0_,
        categoryit0_.CATEGORY_ID as category1_1_1_,
        categoryit0_.ITEM_ID as item_id2_1_1_,
        category1_.CATEGORY_ID as category1_0_2_,
        category1_.createdAt as createda2_0_2_,
        category1_.lastModifiedAt as lastmodi3_0_2_,
        category1_.name as name4_0_2_,
        category1_.parentId as parentid5_0_2_ 
    from
        CategoryItem categoryit0_ 
    inner join
        Category category1_ 
            on categoryit0_.CATEGORY_ID=category1_.CATEGORY_ID 
    where
        categoryit0_.ITEM_ID=?
* 상품 카테고리 : lineNote
* 상품가격 : 10000
* 상품수량 : 1
* 총 가격 : 10000
-----
```
### 영속성 전이를 통해 아래처럼 Delivery 따로 Purchase 따로 저장 해야했던 부분을 간단하게 할 수 있게 됩니다.
```java
//상품구매
Delivery delivery = deliveryService.save(new Delivery(member.getCity(), member.getStreet(), member.getZipcode()));
Purchase purchase = purchaseService.save(new Purchase(member, delivery));
purchaseService.save(new PurchaseItem(purchase, album, 1));

public Purchase save(Purchase purchase) {
  EntityManager em = emf.createEntityManager();
  EntityTransaction tx = em.getTransaction();
  tx.begin();
  Member member = em.merge(purchase.getMember());
  Delivery delivery = em.merge(purchase.getDelivery());
  em.persist(purchase);
  member.addPurchase(purchase);
  tx.commit();
  return purchase;
}
```
