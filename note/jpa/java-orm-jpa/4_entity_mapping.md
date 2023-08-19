# 4장 엔티티 매핑

### 다양한 매핑 사용

- 회원은 일반 회원과 관리자로 구분 됩니다.
- 회원 가입일과 수정일이 있습니다.
- 회원을 설명할 수 있는 필드가 있습니다. ( 길이 제한 없음 )

```java
@Entity
@Table(name = "user", uniqueConstraints = {@UniqueConstraint(
        name = "NAME_AGE_UNIQUE",
        columnNames = {"name", "age"}
)})
@Getter
@Setter
public class User {
    @Id
    private Long id;

    @Column(name = "name", nullable = false, length = 10)
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob
    private String description;

}
```

`@Enumrated` : 자바의 enum 매핑시

`@Temporal` : 자바의 java.util.Date, java.util.Calendar 매핑시 

`@Lob` : DB의 `Lob` 계열의 컬럼을 매핑시

`@UniqueConstraint` : 유니크 제약조건

`@Transient` : 특정 필드는 매핑을 하지 않습니다.

`@Access` : JPA가 엔티티에 접근하는 방식을 지정합니다.

`nullable` : not null 제약조건

`length` : 컬럼 길이

### 기본 키 매핑

애플리케이션에서 직접 PK를 지목 하는 **수동 생성 방법**과 DB에서 생성해주는 PK 값을 사용하는 **자동 생성 방법**이 있습니다.

JPA는 자동 생성시 여러 전략을 사용합니다. DB 마다 PK를 생성 해주는 방법이 다르기 때문입니다.

### 수동 생성 방법

`@Id` 를 이용해 직접 매핑합니다.

```java
@Id
private Long id;

--

Member member = new Member();
member.setId(1); //직접 매핑
em.persist(member);
```

### IDENTITY 전략

`@GeneratedValue(strategy = GeneratedType.IDENTITY)` 를 이용해 PK 생성을 DB에게 맡깁니다.

MySQL, PostgreSQL, SQL Server 에서 사용합니다.

이 전략은 DB에 INSERT를 한 후에 PK를 조회할 수 있습니다. 따라서 엔티티에 식별자 값을 할당하려면 JPA는 추가로 DB를 조회 해야만 합니다. 즉, 레코드를 INSERT한 후에 그 INSERT된 레코드의 기본 키 값을 얻기 위해 다시 데이터베이스를 조회해야 합니다.

엔티티가 영속 상태가 되려면 식별자가 반드시 필요합니다. 그러므로 이 전략은 `em.persist()` 호출시 DB에 SQL이 날아갑니다.

따라서 **이 전략은 트랜잭션을 지원하는 쓰기 지연이 동작하지 않습니다.**

```java
@Id
@GeneratedValue(strategy = GeneratedType.IDENTITY)
private Long id;

--

Member member = new Member();
member.setName("cho");
member.setAge(10);
em.persist(member);
```

### SEQUENCE 전략

오라클, PostgreSQL 과 같은 DB에서만 사용 가능합니다.

`@SequenceGenerator` 를 이용해 시퀀스 생성기를 등록 해야합니다.

`@GeneratedValue( strategy = GeneratedType.SEQUENCE` 로 시퀀스 생성기를 선택합니다.

```java
@Entity
@SequenceGenerator(
        name = "USER_SEQ_GENERATOR",
        sequenceName = "USER_SEQ",
        initialValue = 1, allocationSize = 1
)
public class SequenceUser {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "USER_SEQ_GENERATOR")
    private Long id;

}
```

```java
Hibernate: 
    select
        nextval ('SEQUENCE_USER_SEQ')
Hibernate: 
    insert 
    into
        sequence_user
        (age, createdDate, description, lastModifiedDate, roleType, name, id) 
    values
        (?, ?, ?, ?, ?, ?, ?)
```

### IDENTITY와 SEQUENCE의 차이

INDETITY는 INSERT 후 ID를 조회, SEQUENCE는 ID 조회 후 INSERT를 합니다.

### SEQUENCE 전략과 최적화

데이터베이스 시퀀스를 통해 식별자를 조회하는 추가 작업 때문에 DB와 2번 통신하게 됩니다.

JPA는 시퀀스에 접근하는 횟수를 줄이기 위해 `@SequenceGenerator.allocationSize` 를 사용합니다.

설정한 값만큼 한번에 시퀀스를 증가 시키고 해당 값들을 메모리로 가지고 있는 방법입니다.

```java
@SequenceGenerator(
        name = "USER_SEQ_GENERATOR",
        sequenceName = "SEQUENCE_USER_SEQ",
        initialValue = 1, allocationSize = 1
)
```

### TABLE 전략

키 전용 테이블을 만들어 SEQUENCE를 따라한 전략입니다.

```java
Hibernate: 
    
    create table MY_SEQUENCES (
       sequence_name varchar(255) not null,
        next_val int8,
        primary key (sequence_name)
    )
Hibernate: 
    
    insert into MY_SEQUENCES(sequence_name, next_val) values ('USER_SEQ',0)

Hibernate: 
    select
        tbl.next_val 
    from
        MY_SEQUENCES tbl 
    where
        tbl.sequence_name=? for update
            of tbl
Hibernate: 
    update
        MY_SEQUENCES 
    set
        next_val=?  
    where
        next_val=? 
        and sequence_name=?
Hibernate: 
    insert 
    into
        TableUser
        (age, description, roleType, username, id) 
    values
        (?, ?, ?, ?, ?)
```

### AUTO 전략

`GenerationType.AUTO` 는 DB의 종류에 따라 위 3개 전략중 하나를 택합니다.

`GeneratedValue.strategy` 의 기본값이 `AUTO` 입니다.

### @Enumerated

`@Enumerated.ORDINAL` : DB에 enum에 정의된 순서가 저장됩니다. 저장되는 크기는 작지만 enum의 순서를 바꾸면 안됩니다.

`@Enumerated.STRING` : DB에 문자 그대로 저장됩니다. enum의 순서와 상관 없이 저장되지만 String으로 저장 되므로 ORDINAL 보단 크기가 큽니다.

### @Temporal

`@TemporalType.DATE`

`@TemporalType.TIME`

`@TemporalType.TIMESTAMP`

### @Access

JPA가 엔티티 데이터에 접근하는 방식을 지정합니다.

`AccessType.FIELD` : 필드에 직접 접근합니다.

`AccessType.PROPERTY` : 접근자(getter/setter) 를 사용합니다.

### FIELD 접근

```java
@Entity @Getter @Setter
@Access(AccessType.FIELD)
class AccessTypeEntity {

    @Id
    private Long id;

    private String data1;
    private String data2;
}
```

### PROPERTY 접근

```java
@Entity
@NoArgsConstructor
@Access(AccessType.PROPERTY)
class AccessTypePropertyEntity {

    private Long id;

    private String data1;

    private String data2;

    public AccessTypePropertyEntity(String data1, String data2) {
        this.data1 = data1;
        this.data2 = data2;
    }

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }

    @Column
    public String getData1() {
        return data1;
    }

    public void setData1(String data1) {
        this.data1 = data1;
    }

    public String getData2() {
        return data2;
    }

    public void setData2(String data2) {
        this.data2 = data2;
    }

}
```

### FEILD와 PROPERY 섞어서 접근

세밀하게 접근 제어가 필요한 경우 사용하지만 일관되게 사용하는 것이 좋을듯 싶습니다.

코드가 너무 복잡해지기도 하고 entity에서 로직이 있다는 점이 힘들 것 같습니다.