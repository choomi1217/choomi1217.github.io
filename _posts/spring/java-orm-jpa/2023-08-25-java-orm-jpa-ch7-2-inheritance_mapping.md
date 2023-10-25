---
title:  "[자바 ORM 표준 JPA] ch7.고급매핑 - 상속 관계 매핑"

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

# 상속 관계 매핑

RDBMS에는 상속이라는 개념이 없습니다. 대신 슈퍼타입과 서브타입이 있습니다.

슈퍼타입-서브타입 논리 모델을 물리 모델인 테이블로 구현할 땐 `각각의 테이블로 변환` , `통합 테이블로 변환` , `서브타입 테이블로 변환` 이 있습니다.

### **1. 조인전략**

자식 테이블이 부모 테이블 `item` 의 기본 키를 받아서 이를 기본 키로 사용하는 전략입니다.

**부모 테이블에서는 자식의 타입을 구분하기 위한 구분 컬럼이 필요합니다.**

- 부모 엔티티에 `@DiscriminatorColumn(name = "DTYPE")` 를 이용해 구분 컬럼을 명시합니다.
- 부모 엔티티에 `@Inheritance` 를 사용해 매핑 전략을 선택 해야 합니다.
- 조인전략은 `InheritanceType.JOINED` 매핑 전략을 선택
- 구분 컬럼에 들어가는 값은 자식 엔티티에 `@DiscriminatorValue("Album")` 를 이용합니다.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {
    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    private String name;
    private int price;
}

@Entity
@DiscriminatorValue("Album")
public class Album extends Item{
    private String artist;
}

@Entity
@DiscriminatorValue("Book")
public class Book extends Item{
    private String author;
    private String isbn;
}

@Entity
@DiscriminatorValue("Movie")
public class Movie extends Item{
    private String director;
    private String actor;
}
```

![Untitled](docs/assets/images/java-orm-jpa/7/7_1.png)

- 조인되는 ID 의 컬럼명을 아래와 같이 변경 할 수 있습니다.

```java
@Entity
@DiscriminatorValue("Book")
@PrimaryKeyJoinColumn(name = "BOOK_ID")
public class Book extends Item{
    private String author;
    private String isbn;
}
```

![Untitled](docs/assets/images/java-orm-jpa/7/7_2.png)

### 2. 단일 테이블 전략

조인을 사용하지 않고 한 테이블에

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    private String name;
    private int price;
}

@Entity
@DiscriminatorValue("Album")
public class Album extends Item {
    private String artist;
}

@Entity
@DiscriminatorValue("Book")
@PrimaryKeyJoinColumn(name = "BOOK_ID")
public class Book extends Item {
    private String author;
    private String isbn;
}

@Entity
@DiscriminatorValue("Movie")
public class Movie extends Item {
    private String director;
    private String actor;
}
```

![Untitled](docs/assets/images/java-orm-jpa/7/7_3.png)

### 3. 클래스마다 테이블 전략

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;
    private String name;
    private int price;
}

@Entity
public class Album extends Item {
    private String artist;
}

@Entity
public class Book extends Item {
    private String author;
    private String isbn;
}

@Entity
public class Movie extends Item {
    private String director;
    private String actor;
}
```

![Untitled](docs/assets/images/java-orm-jpa/7/7_4.png)