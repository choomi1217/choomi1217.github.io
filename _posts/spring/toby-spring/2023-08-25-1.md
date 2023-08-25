---
title:  "[Toby Spring 3.1] 초난감 DAO"

categories:
  - spring
tags:
  - [toby-spring]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-08-25
last_modified_at: 2023-08-25
---

# 초난감 DAO

- UserVO

```java
public class UserVO { 

	String id; 

	String name; 

	String password; 

	public String getId() { 
		return id; 
	} 
	public void setId(String id) { 
		this.id = id; 
	} 
	public String getName() { 
		return name; 
	} 
	public void setName(String name) { 
		this.name = name; 
	} 
	public String getPassword() { 
		return password; 
	} 
	public void setPassword(String password) { 
		this.password = password; 
	} 
}
```

```java

public class UserDAO {

    public void add(UserVO user) throws SQLException {
        String conn_url = "jdbc:oracle:thin:@127.0.0.1:1521/xe";
        String conn_id = "cho";
        String conn_pw = "0000";
        Connection conn = DriverManager.getConnection(conn_url, conn_id, conn_pw);

        PreparedStatement ps = conn.prepareStatement("insert into users2(id, name, password) values (?,?,?)");
        ps.setString(1,user.getId());
        ps.setString(2,user.getName());
        ps.setString(3,user.getPassword());

        ps.executeUpdate();

        ps.close();
        conn.close();

    }

    public UserVO get(String id) throws SQLException {
        String conn_url = "jdbc:oracle:thin:@127.0.0.1:1521/xe";
        String conn_id = "cho";
        String conn_pw = "0000";
        Connection conn = DriverManager.getConnection(conn_url, conn_id, conn_pw);

        UserVO user = new UserVO();
        PreparedStatement ps = conn.prepareStatement("select * from users2 where id = ?");
        ps.setString(1,id);

        ResultSet rs = ps.executeQuery();

        rs.next();

        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        conn.close();

        return user;
    }

}

```

```java
public class Main {
    public static void main(String[] args) throws SQLException {

        UserDAO dao = new UserDAO();
            UserVO user = new UserVO();

        user.setId("testID1");
        user.setName("테스트1");
        user.setPassword("testPWD1");

        dao.add(user);

        UserVO user2 = dao.get("testID1");
        System.out.println("ID : " + user2.getId());
        System.out.println("NAME : " + user2.getName());
        System.out.println("PASSWORD : " + user2.getPassword());

    }
}

```

---

### 🤔왜 초난감인가?

변화에 민감한 코드다

객체지향은 미래의 변화를 대비해 설계하고 개발하는 방법인데 초난감 DAO의 경우 변화에 민감하다.

미래에 있을지도 모를 변화들을 가정해보자.

1. 커넥션 정보가 변경 될 수도 있다.
2. 쿼리가 변경 될 수도 있다.
3. DB가 변경 될 수도 있다.
4. 등등 아주 많음

이에 대해 내가 고민해야할 것은 이 변화들이 일어날 때 필요한 작업을 최소화하면서 변화한 코드가 다른 곳에 어떻게 영향을 주지 않을 수 있을까?

이에 대한 해답은

### 관심사의 분리

커넥션정보에 대한 관심사의 분리, 쿼리 실행에 대한 관심사의 분리...

이렇게 분리해서 하나하나 클래스로 가지고 있게 된다면 변화가 생겼을때 최소한의 작업으로 다른 곳에 영향을 주지 않고 유지보수가 가능해진다.

또한 중복된 코드도 분리해주도록 하겠습니다.

그럼 UserDAO의 관심사를 보면

1. Connection에 대한 관심, Connection 정보의 중복
2. 쿼리 실행에 대한 관심
3. Connection 오브젝트를 닫아주는 관심

```
public class UserDAO {

    public void add(UserVO user) throws SQLException {

        Connection conn = getConnection();

        PreparedStatement ps = conn.prepareStatement("insert into users2(id, name, password) values (?,?,?)");
        ps.setString(1,user.getId());
        ps.setString(2,user.getName());
        ps.setString(3,user.getPassword());

        ps.executeUpdate();

        ps.close();
        conn.close();

    }

    public UserVO get(String id) throws SQLException {

        Connection conn = getConnection();

        UserVO user = new UserVO();
        PreparedStatement ps = conn.prepareStatement("select * from users2 where id = ?");
        ps.setString(1,id);

        ResultSet rs = ps.executeQuery();

        rs.next();

        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        conn.close();

        return user;
    }

    public Connection getConnection() throws SQLException {

        String conn_url = "jdbc:oracle:thin:@127.0.0.1:1521/xe";
        String conn_id = "cho";
        String conn_pw = "0000";
        Connection conn = DriverManager.getConnection(conn_url, conn_id, conn_pw);

        return conn;
    }

}

```

이렇게 Connection을 분리해서 변화에 대응은 했다.

만약 커넥션 정보에 대한 변화가 너무 많아진다면..?

가령 예를 들어

후에 100개의 기업들이 사용자관리를 위해 내가 만든 어플리케이션을 사갔고 100개의 기업들의 커넥션 정보가 다 다르다.

그럼 난... getConnection을 100개를 만들어야 하는가?

아니면 100개의 기업들에게 getConnection을 수정해서 쓰라고 해야하는가?

만약 getConnection이 나의 비밀무기라면?

### 어떻게 해야 UserDAO를 공개하지 않고 사용자가 db 커넥션 방식을 바꿀수있게 해줄 수 있는가

일단 내 생각

1. 사용자에게 커넥션 정보를 받아서 그걸로 커넥션을 만들고 쿼리를 실행해준다!
2. 메소드로 만들어둔 getConnection을 객체로 분리하고
3. 객체로 분리한 connection 객체를 상속받은 클래스를 하나 만듦 (기업이름+conn 이런 식으로, 구현체 없이) 클래스를 만들어서
4. 거기서 커넥션을 만들어서 돌려주도록 한다.

그럼 우선

1. 커넥션 클래스 분리
2. `public class ConnectionManager { public Connection getConnection() throws SQLException { String conn_url = "jdbc:oracle:thin:@127.0.0.1:1521/xe"; String conn_id = "cho"; String conn_pw = "0000"; Connection conn = DriverManager.getConnection(conn_url, conn_id, conn_pw); return conn; }`

}

```
2. DAO에서 Connection 해보기

```

public class UserDAO {

```
ConnectionManager connectionManager = new ConnectionManager();

public void add(UserVO user) throws SQLException {

    Connection conn = connectionManager.getConnection();

    PreparedStatement ps = conn.prepareStatement("insert into users2(id, name, password) values (?,?,?)");
    ps.setString(1,user.getId());
    ps.setString(2,user.getName());
    ps.setString(3,user.getPassword());

    ps.executeUpdate();

    ps.close();
    conn.close();

}

public UserVO get(String id) throws SQLException {

    Connection conn = connectionManager.getConnection();

    UserVO user = new UserVO();
    PreparedStatement ps = conn.prepareStatement("select * from users2 where id = ?");
    ps.setString(1,id);

    ResultSet rs = ps.executeQuery();

    rs.next();

    user.setId(rs.getString("id"));
    user.setName(rs.getString("name"));
    user.setPassword(rs.getString("password"));

    rs.close();
    ps.close();
    conn.close();

    return user;
}

}

```

### 분리성공!

분리는 했으나 근본적인 문제는 해결되지 않았음

기업 100개의 커넥션 정보를 100개의 클래스파일로 만들어 줄 수는 없음!

나중에 추가 되거나 변경 되면 매번 클래스를 내가 변경해야하므로

기업이 추가 할 수 있도록 해줘야함 (dao는 내 비밀무기이므로 공개하지 않는 조건하에)

그럼 getConnection에 connection 정보를 기업이 생성 할 수 있도록 해야함

getConnection을 기업이 구현하도록 하면 됨

ConnectionManager를 추상클래스로 만들고 이를 기업이 상속받아서 구현토록 함

1. 인터페이스 생성

```
public interface ConnectionManager {
    Connection getConnection() throws SQLException;
}

```

1. 인터페이스 상속 받은 클래스 생성

```
public class AConnection implements ConnectionManager{
@Override
  public Connection getConnection() throws SQLException {

      String conn_url = "jdbc:oracle:thin:@127.0.0.1:1521/xe";
      String conn_id = "cho";
      String conn_pw = "0000";

      Connection conn = DriverManager.getConnection(conn_url, conn_id, conn_pw);

      return conn;
  }
}

```

1. DAO 커넥션 변경

```
public class UserDAO {

  private final ConnectionManager connectionManager;
  public UserDAO(){
      connectionManager = new AConnection(); //여기선 아직 AConnection이라는 구체적인 클래스명을 지시하고 있음
  }

  public void add(UserVO user) throws SQLException {
      Connection conn = connectionManager.getConnection();
      ...
  }
}

```

### " UserDAO에 팩토리 메소드 패턴을 적용해서 getConnection()을 분리합시다 "

" 여러가지 단점이 많은 상속이라는 방법을 사용한다는 사실이 불편하다 "라고 합니다.

상속은 클래스끼리의 결합도를 낮추는 방법이 아닌것 같습니다!

부모 클래스의 변경이 있을시 자식 클래스에도 변화가 일어나고, 변화가 일어나지 않게 부모클래스를 막거나, 자식 클래스는 다른 클래스를 상속 받지 못하게 되는 여러가지 문제들이 있습니다.

어쨌든 DAO에 손도 안 대고 이제 커넥션을 변경 할 수 있을까요?

아니용

여기선 아직 AConnection이라는 구체적인 클래스명을 지시하고 있습니다.

생성자에서요!

UserDAO의 모든 코드가 ConnectionManager 인터페이스를 제외한 어떤 클래스와도 관계를 갖지 못하게 하는 것이 목표입니다!

이 관심사는 UserDAO가 어떤 클래스의 객체를 이용할지를 결정하게 하는 관심사입니다..

이 관심사도 분리해야합니다

어떻게 분리할 것이냐면

UserDAO를 사용하는 클라이언트(제 3의 클라이언트)가 존재할 것입니다.

두개의 객체가 있고 한 오브젝트가 다른 오브젝트의 기능을 사용한다면 사용되는 쪽이 사용하는 쪽에게 서비스를 제공하는 것!

사용되는 객체는 서비스 사용하는 객체는 클라가 되고

### UserDAO 서비스를 이용하는 클라이언트가 있는곳 ((제 3의 클라이언트)현재로선 main이 되겠음) 이 곳이 바로 UserDAO와 ConnectionManager 구현 클래스의 관계를 결정 해주는 기능을 분리하기 좋은 곳임

UserDAO 객체를 만들어서 사용하기 전에

UserDAO가 어떤 ConnectionManager의 구현체중 어떤걸 골라서 쓸 것인지 결정하도록 합시다.

UserDAO오브젝트와 특정 클래스로부터 만들어진 ConnectionManager오브젝트 사이의 관계를 설정해줘야 한다.

클래스 사이의 관계 설정은 한 클래스가 인터페이스 없이 다른쪽 클래스를 사용한다는 뜻이므로 이번 같이 인터페이스가 낀 경우엔 오브젝트와 오브젝트 사이의 관계를 설정한다 라고 합니다.

오브젝트 사이이 관계는 런타임 시 한쪽이 다른쪽의 오브젝트의 레퍼런스를 갖고 있으면 됩니다.

`connectionManager = new AConnection();`

위 경우 AConnection의 레퍼런스를 UserDAO의 connectionManager에 넣음으로써 두 오브젝트가 사용이라는 관계를 맺게 합니다.

이 두 오브젝트 사이의 관계는 모델링시에 나오지 않고 런타임시 생기는 관계입니다.

바로 이런 런타임 오브젝트 관계를 만들어주는게 바로 클라이언트의 책임입니다.

클라이언트는 UserDAO를 사용하는 입장이므로 UserDAO의 세부적인 ConnectionManager의 구현 클래스를 선택하고(어떤게 ConnectionManager를 구현한지 알고 있어야 하므로 세부적이란 표현 사용)

선택한 구현 클래스의 오브젝트를 생성해서 UserDAO와 연결

기존의 UserDAO는 자신의 생성자에 자신이 구현 클래스의 오브젝트를 생성해서 문제가 발생.

오브젝트 사이의 관계가 만들어지려면 일단 만들어진 오브젝트가 있어야합니다.

오브젝트는 얼마든지 파라미터를 이용해 전달할 수 있으므로 외부에서 만들어준 것을 가져다가 쓰려고 합니다.

UserDAO에 파라미터의 타입을 인터페이스로 선언 해둔 생성자를 만들고

UserDAO의 생성자 호출 시 인터페이스의 구현체를 넘기기만 하면 됩니다.

그렇게 해서 바꾼 UserDAO(서비스)와 main(클라이언트)

### UserDAO

```
private final ConnectionManager connectionManager;

public UserDAO(ConnectionManager connectionManager){
    this.connectionManager = connectionManager;
}

```

### main

```

UserDAO dao = new UserDAO(new AConnection());

```

이제 UserDAO는 자신의 관심사이자 책임인 SQL생성 및 실행에만 집중 할 수 있다 (토비는 이걸 또 분리하고 싶어 보이긴 한다)

더이상 DB생성 방법에 대해서 고민하지 않아도 ㅗ딘다.

이처럼 인터페이스를 넣고 클라이언트의 도움을 얻는 방법(생성자를 통한 구현체 셋팅)은 유연합니다.

### 지금까지 초난감DAO를 개선하면서 사용된 기술적 용어들

1. 개방 폐쇄 원칙 ( Open Closed Principle )
  - 클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다.
  - UserDAO는 db connection이라는 기능 확장에 열려있다.
  - UserDAO의 핵심 기능인 SQL 수행에는 어떤 영향을 주지 않고도 얼마든지 Connection을 확장 할 수 있다.
2. 객체 지향의 설계 원칙 ( SOLID )
  - Single Responsibility Principle : 단일 책임 원칙
  - Open Closed Principle : 개방 폐쇄 원칙
  - Liskov Subsitution Principle : 리스코프 치환 원칙
  - Interface Segregation Principle : 인터페이스 분리 원칙
  - Dependency Inversion Principle : 의존관계 역전 원칙
3. 높은 응집도와 낮은 결합도
  - 높은 응집도
    - 개방 폐쇄 원칙은 높은 응집도와 낮은 결합도 라는 원리로도 설명이 가능하다.
    - 응집도가 높다 : 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 뜻.
    - 초난감DAO의 경우는 응집도가 많이 낮았습니다. db Connection기능과 SQL 수행하는 기능을 하고 있었기 때문입니다.
    - 이 초난감DAO에서 ConnectionManager 클래스를 분리함으로써 응집도를 높였고 이에 대한 효과는
      1. 변경이 필요한 부분을 찾는 일이 쉬워졌습니다. ( ConnectionManager의 구현체만 변경하면 됨 )
      2. 변경이 있더라도 다른 클래스의 수정을 요구하지 않습니다. ( ConnectionManager의 구현체만 변경하면 됨 )
      3. 변경한 기능이 UserDAO에 영향을 주지 않습니다.
      4. 테스트가 간편해졌습니다. ConnectionManager의 구현체만 테스트하면 됩니다.
  - 낮은 결합도
    - 낮은 결합도는 관계를 유지하는 데 꼭 필요한 최소한의 방법으로 연결되는 것입니다.
    - 초난감DAO에서는 UserDAO가 ConnectionManager의 구현체(AConnection)와 직접 관계를 갖지 않도록 하는 것이 낮은 결합도의 실천 방법이었습니다.
4. 전략패턴
  - main - UserDAO - ConnectionManager는 전략 패턴에 해당한다고 볼 수 있습니다.
  - 전략 패턴은 인터페이스를 통해 구현체를 갈아끼우는 패턴이다! 이렇게 공부했었습니다.
  - 중요한 점은 클라이언트의 역할입니다 ( 초난감DAO에서는 main이 되겟죠 )
  - UserDAO를 사용하는 main은 UserDAO가 사용할 전략을 UserDAO의 생성자를 통해 제공해줘야 합니다.
  - 책에서는 UserDAO를 컨텍스트, main을 클라이언트, AConnection을 전략 이라고 설명했습니다!

---

### IoC

현재의 초난감DAO엔 또 다른 문제가 있다

엉겁결에 main이 ConnectionManager의 구현체를 결정해주고 있는 것인데...

main은 그냥 main으로서 역할만 수행하면 되는데 지금 ConnectionManager의 구현체를 결정해주는 역할도 하고 있는 것이다.

이를 분리시키려고 한다.

이 기능을 분리시켜서 만들 오브젝트의 명칭은 팩토리이다.

팩토리 패턴과는 다르지만 이렇게 객체 생성 방법을 결정하고 그 오브젝트를 돌려주는 클래스를 팩토리 클래스라고 한다.

오브젝트를 생성하는 쪽과 오브젝트를 사용하는 쪽의 역할과 책임을 분리하는게 목적.

변경전

```
public static void main(String\\[\\] args) throws SQLException {
  UserDAO dao = new UserDAO(new AConnection());
}

```

변경후

```
public static void main(String\\[\\] args) throws SQLException {
    UserDAO dao = new DaoFactory().userDAO();

  public class DaoFactory {
    public UserDAO userDAO(){
    ConnectionManager conn = new AConnection();
    UserDAO userDAO = new UserDAO(conn);
    return userDAO;
  }
}

```

이렇게 만들어진

1. UserDAO
  - 핵심적인 데이터 로직
2. ConnectionManager
  - 데이터 접근을 위한 기술 로직
3. DaoFactory
  - 어플리케이션의 오브젝트 구성 및 그 관계를 정의
  - 설계도 같은 역할
  - 한 오브젝트가 어떤 오브젝트를 가져와 사용해야 하는지 정의해놓은 코드

이제 다른 기업에 UserDAO를 제공할 때 UserDAO, ConnectionManager, DaoFactory를 제공해야한다.

UserDAO는 소스 제공하지 않지만 DaoFactory는 소스제공 합니다.

새로 만든 ConnectionManager 구현 클래스로 변경이 필요하면 DaoFactory를 수정하면 됩니다.

애플리케이션의 컴포넌트 역할을 하는 오브젝트(애플리케이션 요소들, UserDAO, ConnectionManager ...)와 애플리케이션의 구조를 결정하는 오브젝트(DaoFactory)를 분리 했다는게 가장 의미가 있음.

### 근데 만약 userDAO가 아닌 다른 DAO를 생성하는 기능을 더 추가한다면?

```

public class DaoFactory {
  public UserDAO userDAO(){
    ConnectionManager conn = new AConnection();
    UserDAO userDAO = new UserDAO(conn);
    return userDAO;
  }
  public AccountDAO accountDAO(){
    ConnectionManager conn = new AConnection();
    UserDAO userDAO = new UserDAO(conn);
    return userDAO;
  }
  public MessageDAO messageDAO(){
    ConnectionManager conn = new AConnection();
    UserDAO userDAO = new UserDAO(conn);
    return userDAO;
  }

}

```

위와 같이 `new AConnection`을 하는 부분에 중복이 생성됩니다.

중복문제를 해결하기 위한 가장 좋은 방법은 분리해내는 것입니다.

```
public class DaoFactory {

  public ConnectionManager connectionManager(){
      return new AConnection();
  }

  public UserDAO userDAO(){
      ConnectionManager conn = connectionManager();
      UserDAO userDAO = new UserDAO(conn);
      return userDAO;
  }
}

```

위와 같이 ConnectionManager를 생성하는 부분만 메소드로 분리하면 됩니다.

### 제어권의 이전을 통한 제어관계 역전

일반적으로 main()메소드와 같이 프로그램이 시작되는 지점에서 다음 사용할 오브젝트를 결정하고, 결정한 오브젝트를 생성하고, 만들러진 오브젝트에 있는 메소드 호출하는 등의 작업을 반복한다.

즉, 초난감DAO는 main에서 UserDAO 오브젝트를 직접 생성하고, 만들어진 오브젝트의 메소드를 사용한다.

UserDAO 또한 ConnectionManager를 능동적으로 결정하고 필요할 때 쓴다.

모든 오브젝트가 능동적으로 자신이 사용할 클래스를 정하고 스스로 관장하는 것, 사용하는 쪽이 제어하는 구조. 이걸 뒤집는 것이 IoC이다.

오브젝트가 오브젝트를 선택하는 것이 아니다. 오브젝트인 날 누가 어디서 부를지 모른다. 이것이 IoC이다.

나에 대한 제어 권한을 남에게 위임하는 것이다.

제어의 역전에서는 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않는다. 당연히 생성하지도 않는다.

UserDAO는 자신이 어떤 ConnectionManager 구현체를 만들고 사용할지 결정할 권한을 DaoFactory에게 넘겼으므로 UserDAO는 자신이 사용할 오브젝트를 스스로 선택하지 못하게 됨.

UserDAO 자신도 DaoFactory에 의해 만들어짐.

또한 main 또한 DaoFactory가 만들어주고 공급해주는 ConnectionManager만을 사용하게 됨.

### 스프링의 IoC

현재 DaoFactory는 훌륭하게 IoC기능을 수행하고 있지만 이를 스프링이 할 수 있도록 만드려고 함.

스프링에서는 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 IoC가 적용된 오브젝트들을 빈팩토리라고 함.

즉, DaoFactory는 빈 팩토리 상태임 ( UserDAO와 ConnectionManager간의 관계 설정 및 생성까지 하는 오브젝트이므로 )

빈팩토리를 조금 더 확장한 애플리케이션 컨텍스트라는 것이 있는데 빈팩토리는 빈을 생성하고 관계를 설정하는 IoC의 기본 개념에 촛점을 맞춘 것이고

애플리케이션 컨텍스트는 애플리케이션 전반에 걸쳐 모든 구성요소의 제어 작업을 담당하는 IoC 엔진이다.

현재는 DaoFactory 자체가 설정정보까지 담고 있는 IoC 엔진(스스로가 빈팩토리)이었다면 이걸 애플리케이션 컨텍스트의 설정정보로 사용되게끔 할 것이다.

그럼 DaoFactory의 IoC 작업은 애플리케이션 컨텍스트가 하게 될 거임.

### ApplicationContext에 오브젝트를 올릴 설정파일 만들기

1. DaoFactory를 빈 팩토리가 사용할 수 있는 설정정보로 만듦. (어떤 클래스가 빈에 대한 설정 정보를 담았으며, 어떤 오브젝트를 설정할지의 정보를 담도록)
2. 빈 팩토리를 통해 오브젝트를 가져와서 사용함.

```
@Configuration
public class DaoFactory {
  @Bean
  public ConnectionManager connectionManager(){
      return new AConnection();
  }

  @Bean
  public UserDAO userDAO(){
      ConnectionManager conn = connectionManager();
      UserDAO userDAO = new UserDAO(conn);
      return userDAO;
  }
}

```

1. `@Configuration`
  - 스프링에게 이 클래스가 빈 팩토리를 위한 오브젝트 설정을 담당하고 있음을 알려줌
2. `@Bean`
  - 오브젝트를 만들어주는 메소드임을 알려줌

이렇게 ApplicationContext로부터 getBean으로 오브젝트를 가져올 수 있게 됩니다.

더이상 아래처럼 DaoFactory가 오브젝트를 생성하지 않아도 됩니다.

```
UserDAO dao = new DaoFactory().userDAO();

```

### ApplicationContext로부터 오브젝트를 꺼내와서 사용하기

어떤 오브젝트를 ApplicationContext에서 사용할 것인지 설정하는 클래스를 만들었으니

ApplicationContext로부터 그 오브젝트를 꺼내서 사용해봅시다.

```
ApplicationContext context = new AnnotationConfigApplicationContext(Main.class);
UserDAO dao2 = context.getBean("userDAO",UserDAO.class);

```

### ApplicationContext의 동작방식

DaoFactory를 빈목록에 등록하고 userDAO를 요청하면 그 때 생성 요청하고 생성하고 사용합니다!

왜 ApplicationContext를 사용하냐면

1. 구체적인 팩터리 클래스를 알 필요가 없어집니다.
  - `UserDAO dao2 = context.getBean("userDAO",UserDAO.class);`
  - 위처럼 어디서도 DaoFactory에 대한 정보가 없는데도 오브젝트를 잘 가지고 옵니다.
2. 많은 IoC 관련 서비스를 제공합니다.

### 싱글톤 레지스트리와 오브젝트 스코프

ApplicationContext를 이용한 것과 DaoFactory를 직접 사용하는 것에 큰 차이점이 느껴지지 않습니다.

그저 userDao란 이름의 빈을 요청하면 아래 코드를 이용해 userDAO를 돌려주겠구나 생각한다.

```
@Bean
public UserDAO userDAO(){
    ConnectionManager conn = connectionManager();
    UserDAO userDAO = new UserDAO(conn);
    return userDAO;
}

```

하지만 스프링의 ApplicationContext는 기존 오브젝트 팩토리와 차이점이 분명히 존재합니다.

오브젝트의 동일성과 동등성입니다. 값만 같은가 아니면 아예 같은 오브젝트인가.

아래 코드에서 dao1과 dao2는 서로 다른 오브젝트입니다.

```

DaoFactory daoFactory = new DaoFactory();
UserDAO dao1 = daoFactory.userDAO();
UserDAO dao2 = daoFactory.userDAO();

```

아래 코드는 dao1과 dao2가 완전히 동일한 오브젝트입니다.

```

ApplicationContext context = new AnnotationConfigApplicationContext(Main.class);
UserDAO dao1 = context.getBean("userDAO",UserDAO.class);
UserDAO dao2 = context.getBean("userDAO",UserDAO.class);

```

기존의 오브젝트 팩토리와는 확연히 다른 동작방법인것 같습니다.

스프링은 내가 100번을 요청해도 100번 다 같은 오브젝트를 돌려준다는 것입니다.

단순하게 getBean()을 할 때마다 userDao() 메소드를 호출하고 new에 의해 UserDAO가 생성되지 않는단 뜻입니다.

한번 오브젝트를 생성하고 이렇게 생성한 오브젝트를 재사용하는 방식을 싱글톤패턴이라고 합니다.

스프링은 기본적으로 빈 오브젝트를 모두 싱글톤으로 생성합니다.

디자인 패턴의 싱글톤과 비슷한 개념이지만 구현법이 틀립니다.

만약 UserDAO에 싱글톤패턴을 적용한다면..

```

public class UserDAO{
  private static UserDAO INSTANCE;
  pprivate final ConnectionManager connectionManager;

  private UserDAO(ConnectionManager connectionManager){
      this.connectionManager = connectionManager;
  }

  public static synchronized UserDAO getInstance(){
      if(INSTANCE == null) INSTANCE = new UserDAO( T-T..? );
      return INSTANCE;
  }
}

```

위처럼 코드도 상당히 지저분해지고 private으로 만든 생성자는 외부에서 호출도 불가능해서 DaoFactory가 UserDAO를 생성해서 ConnectionManager를 넣어주는 것도 불가능해졌으며

싱글톤패턴은 오브젝트주입이 어려워 가짜 오브젝트를 넣어 테스트하는 것도 불가능합니다.

그렇다면 싱글톤패턴을 직접 구현하면 이렇게나 제약이 많은데 스프링은 왜 싱글톤으로 오브젝트를 만들까요?

스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공합니다.

그것이 바로 싱글톤 레지스트리라고 하는겁니당!

싱글톤레지스트리는 private과 static으로 떡진 클래스가 아니라 평범한 자바 클래스를 싱글톤으로 활용하도록 합니다.

### 싱글톤과 오브젝트의 상태

싱글톤이 멀티스레드 환경에서 사용되는 경우엔 Stateless 상태로 만들어야 합니다.

상태정보를 내부에 가지고 있지 않은 상태를 말합니다.

멀티스레드 환경에서 스레드들이 동시에 싱글톤 오브젝트의 인스턴스 변수를 수정하는 일은 위험합니다.

저장된 오브젝트는 하나뿐이니 서로 값을 덮어씌우고 결과적으로 자신이 저장한 값이 아닌 것도 읽어오게 됩니다.

따라서 싱글톤은 Stateless하게 만듭니다.

그렇다면 Stateless방식으로 만든 클래스는 어떻게 정보를 다뤄야 할까요??

파라미터, 로컬변수, 리턴 값등을 이용해서 독립된 공간을 갖는 장소에 오브젝트의 정보들을 옮겨놓으면 됩니다!

아래는 인스턴스 변수를 사용하도록 수정한 UserDAO입니다.

```
public class UserDAO {
  private final ConnectionManager connectionManager;
  private Connection conn; //인스턴스 변수
  private UserVO user; //인스턴스 변수

  public UserDAO(ConnectionManager connectionManager){
      this.connectionManager = connectionManager;
  }

  public UserVO get(String id) throws SQLException {

      Connection conn = connectionManager.getConnection();
      this.user = new UserVO();
      ...
 }

```

#DI ( 의존관계 주입 )

IoC와 DI ( 제어의 역전과 의존관계 주입 )

IoC의 핵심을 짚어주는, 좀 더 의도가 명확한 DI라는 단어를 쓴 것 뿐입니다.

DI는 오브젝트의 레퍼런스를 외부로부터 제공 받고 이를 통해 다른 오브젝트와 의존관계가 만들어지는 걸 나타내는 단어입니다.

### 초난감DAO에서는 계속해서 UserDAO와 ConnectionManager간의 관계에 대해 설명하고 있는것 같습니다!

### 그렇다면 UserDAO에는 어떤 의존관계가 있는가?

UserDAO는 ConnectionManager의 getConnection을 꺼내와서 사용하는 의존관계가 있습니다.

다만 이것이 인터페이스로 연결이 되어서 약한 의존관계입니다.

강한 의존관계란 뭘까요? 결합도가 높은 상태입니다.

ConnectionManager 인터페이스가 아닌 실제 구현체와 연결되어 ConnectionManager 내부의 변경에 예민하게 반응하게 되면 안됩니다.

이를 위해서 ConnectionManager를 인터페이스로 두고 AConnection을 구현한겁니다.

일종의 완충제라고 생각합니다!

### 런타임 의존관계?

사실 UserDAO는 ConnectionManager랑만 연결되어 있는것이 아닙니다!

런타임 오브젝트 사이에서 만들어지는 의존관계도 있습니다!

설계시 UserDAO가 런타임시 사용할 오브젝트가 어떤 클래스로 만들어진 것인지 미리 알 수 없습니다.

코드상엔 ConnectionManager만 보이니까요! 헌데 UserDAO가 만들어지고 나서 런타임시엔 실제 사용 대상인 오브젝트가 보입니다.

이를 의존오브젝트라고 합니다!

의존관계 주입의 핵심은 설계 시점에선 알지 못했던 두 오브젝트 ( UserDAO와 ConnectionManager ) 의 관계를 맺도록 도와주는 제 3의 존재 ( DaoFactory )가 있다는 겁니다!

DI에서의 제 3의 존재는 바로 관계설정 책임을 가진 코드를 분리해서 만든 오브젝트입니다.

DI에 대해서 정리를 해보자면

1. 오브젝트의 레퍼런스를 외부로부터 제공 받고 이를 통해 다른 오브젝트와 의존관계가 만들어지는 것입니다.
2. 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정
3. 런타임 시점의 의존관계를 갖기 위해선 인터페이스에만 의존하고 있어야 합니다.

의존관계 주입의 핵심은 두 오브젝트의 관계를 맺도록 도와주는 제3의 존재가 있다는 것

제 3의 존재는 바로 관계설정 책임을 가진 코드를 분리해서 만들어진 오브젝트입니다.

### UserDAO의 DI

초난감DAO의 마지막 난관은 UserDAO 스스로가 사용할 구체적인 클래스가 무엇인지 알고 있다는 것이었습니다.

```
public UserDAO(){
connectionManager = new AConnection();
}

```

위 코드에 따르면 UserDAO는 설계 시점에서 AConnection이라는 구체적인 클래스의 존재를 알고 있는 것입니다.

그래서 IoC의 방법을 사용해서 UserDAO로부터 런타임 의존관계를 드러내는 코드를 제거하고 제 3의 존재에게 관계설정의 책임을 분리한것입니다.

### 기능구현의 교환

###기능교환

만약 회사에서 기능개발을 위한 로컬 DB 서버를 연결해두고 사용하다가 운영서버에 배치하기 위해 DB 커넥션 정보를 바꿔야 한다면

앞으로 DaoFactory의 이 부분만 바꾸면 됩니다.

```
@Bean
public ConnectionManager connectionManager(){
    return new AConnection();
}

```

###기능추가

DAO가 DB에 얼마나 많이 연결되어 사용되는지 파악하고 싶다면 DB 연결횟수 카운트하기 위해 DAO의 connectionManager() 메소드가 얼마나 생성되는 카운트해야 할까요?

그럼 카운트를 했다치고 작업이 완료되면 코드를 지울건가요?

DAO 코드의 수정은 피해야하며, DB 연결횟수를 세는 것은 DAO의 관심사가 아닙니다.

어떻게는 분리해야합니다.

DI컨테이너는 이를 간단하게 해결합니다.

UserDAO와 ConnectionManager 사이에 연결횟수를 카운팅하는 오브젝트를 하나 더 추가합니다.

```
public class CountingConnectionManager implements ConnectionManager{
  int count = 0;
  private ConnectionManager realConnectionManager;

  public CountingConnectionManager(ConnectionManager realConnectionManager){
      this.realConnectionManager = realConnectionManager;
  }

  @Override
  public Connection getConnection() throws SQLException {
      this.count++;
      return realConnectionManager.getConnection();
  }

  public int getCount(){
      return this.count;
  }
}

```

DaoFactory

```
@Bean
public ConnectionManager connectionManager(){
    return new CountingConnectionManager(realConnectionManager());
}

@Bean
public ConnectionManager realConnectionManager(){
    return new AConnection();
}

```

main

```
ApplicationContext context = new AnnotationConfigApplicationContext(Main.class);
//UserDAO dao = context.getBean("userDAO",UserDAO.class);

CountingConnectionManager ccm = context.getBean("connectionManager",CountingConnectionManager.class);
System.out.println(ccm.getCount());

```