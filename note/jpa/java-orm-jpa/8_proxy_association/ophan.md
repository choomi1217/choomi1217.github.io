JPA는 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제한다.

부모 객체가 사라지면 자식을 삭제하고 부모에게서 자식을 `remove` 한다고 자식이 삭제 되지 않습니다.
( 아래처럼 자식을 삭제하고 왜 자꾸 delete 가 안 날아가지 라고 생각하고 있었습니다 🤔 )

```java
public class Main {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();
        Parent parent = em.find(Parent.class, 255L);
        parent.getChild().forEach(child -> {
            System.out.println("child = " + child.getName());
        });

        parent.getChild().remove(0);
        tx.commit();

        System.out.println("--- after remove ---");
        parent.getChild().forEach(child -> {
            System.out.println("child = " + child.getName());
        });

        emf.close();
    }
}
```

```java
@Entity
public class Parent {
    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    private String name;
    @OneToMany(mappedBy = "parent", orphanRemoval = true)
    private List<Child> child = new ArrayList<Child>();
		//getter&setter
}
```

```java
public class Main {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myApp");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();
        Parent parent = em.find(Parent.class, 269L);
        List<Child> children = parent.getChild();
        children.forEach(child -> {
            System.out.println("child = " + child.getName());
        });

        em.remove(parent);
        tx.commit();

        try {
            em.find(Child.class, children.get(0));
        }catch (IllegalArgumentException e) {
            System.out.println("child is null");
        }

        emf.close();
    }
}

-----

Hibernate: 
    select
        parent0_.PARENT_ID as parent_i1_1_0_,
        parent0_.name as name2_1_0_ 
    from
        Parent parent0_ 
    where
        parent0_.PARENT_ID=?
Hibernate: 
    select
        child0_.PARENT_ID as parent_i3_0_0_,
        child0_.CHILD_ID as child_id1_0_0_,
        child0_.CHILD_ID as child_id1_0_1_,
        child0_.name as name2_0_1_,
        child0_.PARENT_ID as parent_i3_0_1_ 
    from
        Child child0_ 
    where
        child0_.PARENT_ID=?
child = child1
child = child2
**Hibernate: 
    delete 
    from
        Child 
    where
        CHILD_ID=?
Hibernate: 
    delete 
    from
        Child 
    where
        CHILD_ID=?
Hibernate: 
    delete 
    from
        Parent 
    where
        PARENT_ID=?
child is null**
```

### 영속성 전이 + 고아 객체, 생명주기

부모 엔티티를 통해 자식의 생명주기를 관리할 수 있게 됩니다.

자식을 저장하려면 부모에 등록하고, 자식을 삭제하려면 부모에서 제거하면 됩니다.