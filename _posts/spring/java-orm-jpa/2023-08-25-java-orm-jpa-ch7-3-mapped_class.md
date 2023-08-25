---
title:  "[자바 ORM 표준 JPA] ch7.고급매핑 - 공통 클래스 상속"

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

# @MappedSuperclass  : 공통 내용 상속하기

상속관계에서 부모도 테이블, 자식도 테이블이었지만 `@MappedSuperclass` 를 사용하면 **부모는 테이블로 등록하지 않고 자식에게 매핑 정보만 제공 할 수 있습니다.**

또한 `@MappedSuperclass` 는 엔티티가 아니고 직접 사용할 일도 없으므로 추상 클래스로 만드는게 좋습니다!

![Untitled](../image/7/7_5.png)

### 재정의 하지 않고 부모의 것 그대로 사용

```java
@MappedSuperclass
public class BaseEntity {
    @Id @GeneratedValue
    private Long id;
    private String name;
}

@Entity
public class Member extends BaseEntity{
    private String email;
}

@Entity
public class Seller extends BaseEntity{
    private String shopName;
}
```

![Untitled](../image/7/7_6.png)

### 부모의 것 재정의

```java
@Entity
@AttributeOverrides({
        @AttributeOverride(name = "id", column = @javax.persistence.Column(name = "MEMBER_ID")),
        @AttributeOverride(name = "name", column = @javax.persistence.Column(name = "MEMBER_NAME"))
})
public class Member extends BaseEntity {
    private String email;
}

@Entity
@AttributeOverrides({
    @AttributeOverride(name = "id", column = @javax.persistence.Column(name = "SELLER_ID")),
    @AttributeOverride(name = "name", column = @javax.persistence.Column(name = "SELLER_NAME"))
})
public class Seller extends BaseEntity {
    private String shopName;
}
```

![Untitled](../image/7/7_7.png)