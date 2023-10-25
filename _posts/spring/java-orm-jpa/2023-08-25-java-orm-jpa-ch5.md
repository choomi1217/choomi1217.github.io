---
title:  "[자바 ORM 표준 JPA] ch5.연관관계"

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


# 5장 연관관계

### 방향

누가 누구를 조회하는 객체의 방향에 대한 것입니다.

`단방향` 과 `양방향` 이 있습니다.

### 다중성

관계에 참여하는 객체들의 수에 대한 것입니다.

`1 : 1` , `N : 1` , `N : M` 이 있습니다.

### 연관관계의 주인

양방향 연관관계에선 연관관계의 주인을 정해야 합니다.

# 💡 단방향 연관관계

아래 N:1 단방향 연관관계를 보면 **객체는 단방향이지만 테이블은 양방향**입니다.

**객체는 참조로 연관관계를 맺고 테이블은 외래키로 연관관계**를 맺습니다.

**객체로 양방향을 하려면 단방향을 두개** 만들어야 합니다.

### N:1 단방향 연관관계

- 객체 연관관계
    - **회원과 팀은 단방향입니다.**
    - 회원은 팀을 조회 할 수 있지만 팀은 회원을 조회 할 수 없습니다.

![Untitled](docs/assets/images/java-orm-jpa/5/5_1.png)

- 테이블 연관관계
    - **회원과 팀은 양방향입니다.**

![Untitled](docs/assets/images/java-orm-jpa/5/5_2.png)

### 연관관계 사용

```java
@Entity
@NoArgsConstructor
class Member {
    
    @Id
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    public Member(String id, String username) {
        this.id = id;
        this.username = username;
    }
		//getter & setter
}

```

```java
@Entity
@NoArgsConstructor
class Team{
    @Id
    @Column(name = "TEAM_ID")
    private String id;
    private String name;

    public Team(String id, String name) {
        this.id = id;
        this.name = name;
    }
		//getter & setter
}
```

```java
public class _2_OneToManyEntity {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();
        Team team = new Team("team1", "팀1");
        em.persist(team);

        Player player = new Player("member1", "회원1");
        player.setTeam(team);
        em.persist(player);

        Team playerTeam = player.getTeam();
        System.out.println("playerTeam = " + playerTeam.getName());

        tx.commit();
        emf.close();

    }
}
```

### 저장

```java
Team team = new Team("team1", "팀1");
em.persist(team);

Player player = new Player("member1", "회원1");
player.setTeam(team);
em.persist(player);
```

```java
Hibernate: 
    insert 
    into
        Team
        (name, TEAM_ID) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        Player
        (TEAM_ID, username, MEMBER_ID) 
    values
        (?, ?, ?)
```

![Untitled](docs/assets/images/java-orm-jpa/5/5_3.png)

![Untitled](docs/assets/images/java-orm-jpa/5/5_4.png)

### 조회

- 객체 그래프 탐색

```java
Team playerTeam = player.getTeam();
System.out.println("playerTeam = " + playerTeam.getName());
```

- 객체지향 쿼리 사용 (JPQL 사용)

```java
String jpql = "select p from Player p join p.team t where t.name = :teamName";
List<Player> resultList = em.createQuery(jpql, Player.class)
        .setParameter("teamName", "팀1")
        .getResultList();
resultList.forEach(player -> System.out.println("player = " + player.getUsername()));
```

### 수정

```java
Team team2 = new Team("team2", "팀2");
em.persist(team2);

Player findPlayer = em.find(Player.class, "member1");
findPlayer.setTeam(team2);
```

![Untitled](docs/assets/images/java-orm-jpa/5/5_5.png)

![Untitled](docs/assets/images/java-orm-jpa/5/5_6.png)

### 연관관계 삭제

```java
findPlayer.setTeam(null);
```

![Untitled](docs/assets/images/java-orm-jpa/5/5_7.png)

![Untitled](docs/assets/images/java-orm-jpa/5/5_8.png)

# 💡 양방향 연관관계

Member와 Team은 N:1 관계 → Member.team

Team과 Member는 1:N 관계 → Team.member**s**

![Untitled](docs/assets/images/java-orm-jpa/5/5_9.png)

![Untitled](docs/assets/images/java-orm-jpa/5/5_10.png)

```java
@Entity
class TwoWayPlayer{
    @Id
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private OneWayTeam oneWayTeam;

    //getter&setter
}
```

```java
@Entity
@NoArgsConstructor
class TwoWayTeam{
    @Id
    @Column(name = "TEAM_ID")
    private String id;
    private String name;
    @OneToMany(mappedBy = "twoWayTeam")
    private List<TwoWayPlayer> twoWayPlayers = new ArrayList<TwoWayPlayer>();
		
		//getter&setter
}
```

### 일대다 컬렉션 조회

```java
TwoWayTeam twoWayTeam = em.find(TwoWayTeam.class, "team1");
List<TwoWayPlayer> twoWayPlayers = twoWayTeam.getTwoWayPlayers();
twoWayPlayers.forEach(player -> System.out.println("player = " + player.getUsername()));
```

### 연관관계의 주인

왜 연관관계에 주인이 필요할까?

객체의 참조는 둘인데 외래키는 하나라서 둘 사이에 차이가 발생합니다.

두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리하는 객체가 주인입니다.

**연관관계의 주인만이 DB 연관관계와 매핑되고 외래키를 관리 할 수 있습니다.**

주인이 아닌 쪽은 읽기만 가능합니다.

주인이 아닌 쪽은 `mappedBy` 를 사용해 연관관계의 주인을 지정해야 합니다.

### 연관관계의 주인은 외래키를 가진 테이블

![Untitled](docs/assets/images/java-orm-jpa/5/5_11.png)

ERD를 보면 `TEAM` 은 `MEMBER` 에 대한 정보를 아무것도 가지지 않은 테이블입니다.

`TEAM` 이 관계의 주인이 되면 물리적으로 연관이 없는 `MEMBER` 의 데이터도 관리하게 되므로 관계의 주인은 `MEMBER` 가 됩니다.

### 저장

`twoWayTeam.getTwoWayPlayers().add(player1)` 를 해도 `player` 에 `team` 이 추가되지 않습니다.

`team` 이 연관관계의 주인이 아니기 때문입니다.

```java
TwoWayTeam twoWayTeam = new TwoWayTeam("team1", "팀1");
em.persist(twoWayTeam);

TwoWayPlayer player1 = new TwoWayPlayer("member1", "회원1");
player1.setTwoWayTeam(twoWayTeam);

TwoWayPlayer player2 = new TwoWayPlayer("member2", "회원2");
player2.setTwoWayTeam(twoWayTeam);

em.persist(player1);
em.persist(player2);
```

![Untitled](docs/assets/images/java-orm-jpa/5/5_12.png)

![Untitled](docs/assets/images/java-orm-jpa/5/5_13.png)

### 순수한 객체까지 고려한 양방향 연관관계 ( 연관관계 편의 메소드 )

양쪽 모두에서 값을 입력 해주지 않으면 JPA를 사용하지 않는 순수한 객체 상태에서 문제가 발생합니다.

아래와 같이 `setTwoWayTeam()` 을 할때 양쪽 모두 설정 해주는 것이 좋습니다. 

양방향 관계를 설정하는 메소드를 `연관관계 편의 메소드` 라고 합니다.