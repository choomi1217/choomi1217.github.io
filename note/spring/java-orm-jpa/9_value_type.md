# 값 타입

JPA **엔티티 타입**과 **값 타입**으로 나눌 수 있습니다.
엔티티 타입은 `@Entity`로 정의하고, **값 타입은 자바 기본 타입이나 객체** 입니다.

엔티티 타입은 식별자를 통해 추적할 수 있지만, 
값 타입은 식별자가 없고 숫자나 문자같은 속성만 있으므로 추적할 수 없습니다.

![](image/9/jpa%20값과%20임베디드%20타입.png)
### 기본값 타입
회원 엔티티에 '근무 시작일, 근무 종료일, 도시, 거리, 우편번호' 이런 데이터가 개별적으로 존재하는 건 응집도가 떨어지고 객체지향적이지 않습니다.
이를 해결하기 위해 임베디드 타입으로 **근무시간** , **주소** 로 나눠 임베디드 타입으로 하는 것이 좋습니다.
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String name;
    private int age;
    
    //근무 기간
    @Temporal(TemporalType.DATE) java.util.Date startDate;
    @Temporal(TemporalType.DATE) java.util.Date endDate;
    
    //집 주소
    private String city;
    private String street;
    private String zipcode;
}
```

### 단순한 임베디드 타입
기본 값 타입과 다르게 응집력이 높고 객체지향의 특징중 하나인 재사용이 가능합니다.
UML로 표현시 컴포지션 관계입니다.
임베디드 타입은 엔티티의 값을 뿐이라서 따로 `Period` 테이블이 만들어지거나 하지 않습니다.
- `@Embeddable` : 값 타입을 정의
- `@Embedded` : 값 타입을 사용
```java
@Entity
public class Member {
  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;
  private String name;
  private int age;

  //근무 기간
  @Embedded Period workPeriod;
  //집 주소
  @Embedded Address homeAddress;
}

@Embeddable
class Period {
  @Temporal(TemporalType.DATE) java.util.Date startDate;
  @Temporal(TemporalType.DATE) java.util.Date endDate;
}

@Embeddable
class Address {
  private String city;
  private String street;
  private String zipcode;
}
```

### 복잡한 임베디드 타입 (임베디드 타입과 연관관계)
`Address`에 `Zipcode`가 포함 된 것 처럼 **임베디드에 임베디드를 할 수 있습니다.**
`PhoneNumber`에 `PhoneServiceProvider`가 포함 된 것 처럼 **임베디드에 엔티티를 참조할 수 있습니다.**
`Address`를 중복해서 사용한다면 `@AttributeOverrides`를 사용해서 컬럼명을 변경할 수 있습니다.
```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String name;
    private int age;

    @Embedded
    Address address;
    @Embedded
    PhoneNumber phoneNumber;
}
```
- 임베디드 안에 임베디드
```java
@Embeddable
public class Address {
    private String city;
    private String street;
    @Embedded
    Zipcode zip;
}

@Embeddable
public class Zipcode {
    String zip;
    String plusFour;
}
```

- 임베디드 안에 엔티티
```java
@Embeddable
public class PhoneNumber {
    String areaCode;
    String localNumber;

    @ManyToOne
    PhoneServiceProvider provider;
}

@Entity
public class PhoneServiceProvider {
    @Id
    @GeneratedValue
    @Column(name = "PHONE_SERVICE_PROVIDER_ID")
    Long id;
    @Column(name = "PHONE_SERVICE_PROVIDER_NAME")
    String name;
}
```

```java
public class Main {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();

        Member member = new Member();
        member.setName("John Doe");
        Address address = new Address();
        address.setCity("Seoul");
        address.setStreet("Gangnam Street");
        Zipcode zipcode = new Zipcode();
        zipcode.setZip("122");
        zipcode.setPlusFour("6789");
        address.setZip(zipcode);
        member.setAddress(address);
        PhoneNumber phoneNumber = new PhoneNumber();
        phoneNumber.setAreaCode("212");
        phoneNumber.setLocalNumber("555-1234");
        PhoneServiceProvider provider = new PhoneServiceProvider();
        provider.setName("KT");
        em.persist(provider);
        phoneNumber.setProvider(provider);
        member.setPhoneNumber(phoneNumber);
        em.persist(member);

        tx.commit();
        em.close();
        emf.close();

    }
}
```
![](image/9/9_1.png)

- 중복된 임베디드 값
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String name;
    
    @Embedded private Address houseAddress;
    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name = "city", column = @Column(name = "COMPANY_CITY")),
            @AttributeOverride(name = "street", column = @Column(name = "COMPANY_STREET")),
            @AttributeOverride(name = "zipcode", column = @Column(name = "COMPANY_ZIPCODE"))
    })
    private Address companyAddress;
}

@Embeddable
class Address {
    private String city;
    private String street;
    private String zipcode;
}

```

## 값 타입과 불변 객체
`값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념` 이라고 책에 쓰여 있는데 
이 말은 굳이 엔티티로 매핑하지 않아도 엔티티를 객체지향적으로 관리할 수 있기 때문에 복잡성을 감소 한다. 
라는 말인것 같습니다.

### 값 타입 공유 참조
값 타입은 공유하면 위험합니다.
아래와 같이 `a.getHomeAddress()`를 통해 `homeAddress`를 가져오고 `homeAddress.setCity("busan")`를 통해 값을 변경하고 memberB에 할당하면 memberB의 값만 변경되길 원하지만 사실은 memberA의 값도 변경됩니다.
이렇듯 뭔가를 수정했는데 예상치 못한 문제가 발생하는 것을 `side effect`라고 합니다.
```java
tx.begin();
Member a = new Member();
a.setName("member A");
a.setAge(20);
a.setHomeAddress(new Address("seoul", "gangnam", "123-456"));
em.persist(a);
tx.commit();

tx.begin();
Address homeAddress = a.getHomeAddress(); //memberA의 주소 참조
homeAddress.setCity("busan"); //memberA의 주소 변경
Member b = new Member();
b.setName("member B");
b.setAge(30);
b.setHomeAddress(homeAddress); //memberA의 주소 공유
em.persist(b);
tx.commit();
```
```text
Hibernate: 
    insert 
    into
        Member
        (age, city, street, zipcode, name, MEMBER_ID) 
    values
        (?, ?, ?, ?, ?, ?)
Hibernate: 
    ***update***
        Member 
    set
        age=?,
        city=?,
        street=?,
        zipcode=?,
        name=? 
    where
        MEMBER_ID=?
```
![](image/9/9_2.png)

### 값 복사
`homeAddress.clone();` clone 메소드를 아무리 오버라이드 해서 객체를 복사해도 근본적인 문제는 해결되지 않습니다.
`homeAddress.setCity("busan");` 이런 식으로 변경할 수 있다는 사실은 변하지 않습니다.
그러므로 값 타입은 **불변 객체**로 만들어야 합니다.
```java
Address homeAddress = a.getHomeAddress();
  Address clone = (Address)homeAddress.clone();
clone.setCity("busan");
Member b = new Member();
b.setHomeAddress(clone);
```

### 불변 객체
불변이라는 제약으로 부작용을 막을 수 있다는게 제일 큰 장점인 것 같습니다.
매번 생성자를 만들어야 한다는 조금의 불편함을 감수하자!
```java
tx.begin();
Member a = new Member();
a.setName("member A");
a.setAge(20);
a.setHomeAddress(new Address("seoul", "gangnam", "123-456"));
em.persist(a);
tx.commit();

tx.begin();
Member b = new Member();
b.setName("member B");
b.setAge(30);
b.setHomeAddress(new Address("busan", "haewoondae", "789-123"));
em.persist(b);
tx.commit();
```

### 값 비교
단순하게 설명하자면 객체의 비교는 `equals` 를 사용해야 합니다.
`equals`, `hashcode` 를 재정의 해서 사용해야 합니다.

```java
public class Main {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();
        Member memberA = em.find(Member.class, 290L);
        Address memberAHomeAddress = memberA.getHomeAddress();

        Member memberB = em.find(Member.class, 291L);
        Address memberBHomeAddress = memberB.getHomeAddress();
        tx.commit();
        emf.close();
        System.out.println("동일성 비교 : " + (memberAHomeAddress == memberBHomeAddress));
        System.out.println("동등성 비교 : " + (memberAHomeAddress.equals(memberBHomeAddress)));
    }
}
```
```text
Hibernate: 
    select
        member0_.MEMBER_ID as member_i1_0_0_,
        member0_.age as age2_0_0_,
        member0_.city as city3_0_0_,
        member0_.street as street4_0_0_,
        member0_.zipcode as zipcode5_0_0_,
        member0_.name as name6_0_0_ 
    from
        Member member0_ 
    where
        member0_.MEMBER_ID=?
Hibernate: 
    select
        member0_.MEMBER_ID as member_i1_0_0_,
        member0_.age as age2_0_0_,
        member0_.city as city3_0_0_,
        member0_.street as street4_0_0_,
        member0_.zipcode as zipcode5_0_0_,
        member0_.name as name6_0_0_ 
    from
        Member member0_ 
    where
        member0_.MEMBER_ID=?
동일성 비교 : false
동등성 비교 : true
```
```java
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;

    public Address() { }

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }

    public String getCity() {
        return city;
    }

    public String getStreet() {
        return street;
    }

    public String getZipcode() {
        return zipcode;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(city, address.city) && Objects.equals(street, address.street) && Objects.equals(zipcode, address.zipcode);
    }

    @Override
    public int hashCode() {
        return Objects.hash(city, street, zipcode);
    }
}
```

### 값 타입 컬렉션
- `@ElemetnCollection`,`@CollectionTable`을 이용해서 컬렉션을 매핑하게 됩니다.
- 컬렉션은 따로 테이블이 만들어지고 연관 관계는 `Member`의 id를 외래키로 가지게 됩니다.
- 컬렉션은 언제나 `Lazy Loading`이 기본입니다.
- 따로 만들어진 테이블은 그저 값들의 모임입니다.
```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String name;
    private int age;
    
    @ElementCollection
    @CollectionTable(name = "FAVORITE_FOODS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<>();
    
    @ElementCollection
    @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ArrayList<>();

}
```
- 등록
```java
Member member = new Member();
member.setName("cho");
member.setAge(20);
member.addFavoriteFood("치킨");
member.addFavoriteFood("피자");
member.addFavoriteFood("빵");
member.addAddressHistory(new Address("gangwondo", "donghae", "345-678"));
member.addAddressHistory(new Address("daejeon", "donggoo", "30-333"));
em.persist(member);
```

- 조회
```java
Member member = em.find(Member.class, 292L);
Set<String> favoriteFoods = member.getFavoriteFoods();

System.out.println("----- member favorite food list -----");
favoriteFoods.forEach(System.out::println);
```
```text
Hibernate: 
    select
        member0_.MEMBER_ID as member_i1_2_0_,
        member0_.age as age2_2_0_,
        member0_.name as name3_2_0_ 
    from
        Member member0_ 
    where
        member0_.MEMBER_ID=?
----- member favorite food list -----
Hibernate: 
    select
        favoritefo0_.MEMBER_ID as member_i1_1_0_,
        favoritefo0_.FOOD_NAME as food_nam2_1_0_ 
    from
        FAVORITE_FOODS favoritefo0_ 
    where
        favoritefo0_.MEMBER_ID=?
빵
치킨
카레
```

- 수정
```java
Member member = em.find(Member.class, 292L);
        Set<String> favoriteFoods = member.getFavoriteFoods();

        System.out.println("----- member favorite food list -----");
        favoriteFoods.forEach(System.out::println);

        tx.begin();
        System.out.println("----- member favorite food delete -----");
        favoriteFoods.remove("피자");
        favoriteFoods.add("카레");
        tx.commit();
```
```text
Hibernate: 
    select
        member0_.MEMBER_ID as member_i1_2_0_,
        member0_.age as age2_2_0_,
        member0_.name as name3_2_0_ 
    from
        Member member0_ 
    where
        member0_.MEMBER_ID=?
----- member favorite food list -----
Hibernate: 
    select
        favoritefo0_.MEMBER_ID as member_i1_1_0_,
        favoritefo0_.FOOD_NAME as food_nam2_1_0_ 
    from
        FAVORITE_FOODS favoritefo0_ 
    where
        favoritefo0_.MEMBER_ID=?
빵
치킨
피자
----- member favorite food delete -----
Hibernate: 
    delete 
    from
        FAVORITE_FOODS 
    where
        MEMBER_ID=? 
        and FOOD_NAME=?
Hibernate: 
    insert 
    into
        FAVORITE_FOODS
        (MEMBER_ID, FOOD_NAME) 
    values
        (?, ?)
```

![](image/9/9_3.png)
