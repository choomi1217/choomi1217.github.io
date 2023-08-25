# Account와 Member

- email이 UK임에도 account_id를 PK 삼은 이유 : email은 바뀔 수 있기 때문에

```java
@Entity
public class AccountEntity {
    @Id
    @GeneratedValue(strategy = jakarta.persistence.GenerationType.IDENTITY)
    private Long accountId;
    private String email;
    private String password;
    private boolean isEmailVerified;
    private boolean isActive;
}
```