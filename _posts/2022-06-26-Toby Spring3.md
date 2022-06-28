# 토비의 스프링 3장 - 템플릿

## UserDao의 try - catch

PreparedStatement와 Connection 객체는 사용후에 반납 해야하는 객체이며 예외가 발생 할 수 있기 때문에
try-catch로 예외를 잡고 finally를 통해 메소드 실행 후 언제나 close()를 할 수 있도록 만들었습니다.

```
public void deleteAll() throws SQLException {

        
        PreparedStatement ps = null;

        try {
            this.conn = connectionManager.getConnection();
            ps = conn.prepareStatement("truncate table users2");
            ps.executeUpdate();
        }catch (SQLException e){
            throw e;
        }finally {
            if(ps!=null){
                try {
                    ps.close();
                }catch (SQLException e){
                    e.printStackTrace();
                }
            }
            if(conn!=null){
                try {
                    conn.close();
                }catch (SQLException e){
                    e.printStackTrace();
                }
            }
        }

    }
```

## 코드의 중복 문제

이렇게 try-catch로 감싸는 부분이 모든 sql 실행 부분에서 중복된다는 문제가 나타났습니다.
그래서 중복되는 코드를 분리하려 했으나 

#### 중복되는 부분

```
PreparedStatement ps = null;

try {
	this.conn = connectionManager.getConnection();


```

#### 중복되지 않는 부분 

```
	ps = conn.prepareStatement("truncate table users2");
	ps.executeUpdate();

```

#### 중복되는 부분

```
	}catch (SQLException e){
				throw e;
			}finally {
				if(ps!=null){
					try {
						ps.close();
					}catch (SQLException e){
						e.printStackTrace();
					}
				}
				if(conn!=null){
					try {
						conn.close();
					}catch (SQLException e){
						e.printStackTrace();
					}
				}
			}

```

이런식으로 중복되는 부분이 중복되지 않는 부분을 감싸고 있으므로 중복되는 부분의 추출이 어렵습니다!
그래서 토비는 이를 반대로 합니다.

### 중복되지 않는 부분을 메소드로 추출합니다.

```
try {
	this.conn = connectionManager.getConnection();
	ps = makeStatement(conn);
	ps.executeUpdate();
}
```

이렇게 ` ps = makeStatement(conn); ` makeStatement(conn)으로 추출했습니다!


## 템플릿 메소드 패턴
템플릿 메소드는 상속을 통한 기능을 확장해서 사용하는 부분입니다.
변하지 않는 부분은 슈퍼클래스의 추상메소드로 두고 상속을 통해 새롭게 정의해서 쓰도록 합니다.

makeStatement를 추상메소드로 바꿉시다!!

UserDao에 아래처럼 추상 메소드를 하나 추가합니다.

```
	// 추가한 추상 메소드
    public abstract PreparedStatement makeStatement(Connection conn) throws SQLException;
```

이를 상속받아 구현한 UserDeleteAll 클래스를 만듭니다!

```
	public class UserDeleteAll extends UserDAO{

    public UserDeleteAll(ConnectionManager connectionManager) {
        super(connectionManager);
    }

    public PreparedStatement makeStatement(Connection conn) throws SQLException {
        PreparedStatement ps = conn.prepareStatement("TRUNCATE TABLE USER2");
        return ps;
    }

	}

```
새로운 SQL문을 추가 시켜서 실행한다고 해도 기존 UserDAO에는 변화가 거의 없는 그럭저럭 OCP를 잘 지킨 구조는 만들었으나,,,

이런식으로 만들게 되면 문제가 있습니다.
새로운 sql문을 실행하려고 할 때마다 자식 클래스가 왕창 생긴다는것 입니다 ㅠㅠ
만약 sql문을 100개 정도 만들어서 실행한다고 하면 100개의 class가 만들어집니다

이렇게 템플릿 메소드는 
확장의 구조가 클래스를 설계하는 시점에서 고정된다? <= Q?? 고정이 된다는게 무슨 뜻이야?


## 템플릿 메소드 구조가 아닌 전략패턴의 사용 
OCP를 잘 지키면서도 템플릿 메소드보다 유연하고 확장성이 뛰어난 전략패턴을 사용하도록 하겠습니다!

전략패턴은 확장 되는 부분을 별도의 클래스로 만들어서 인터페이스를 통해 위임하는 방식입니다? 
Q?? 부모형으로 객체 선언하고 자식을 넣는다 이 말인건가?

UserDeleteAll은 JDBC를 이용해 DB를 업데이트하는 작업이라는 변하지않는 맥락(context)을 가집니다.
1. db 커넥션 가져오기
2. prepareStatement 생성 메소드 호출하기 (이걸 내가 만들겁니다!)
3. 리턴 받은 prepareStatement 객체를 실행하기
4. 예외가 발생하면 예외 던지기
5. close()

여기서 2번째 과정이 전략 패턴에서 전략이라고 불립니다.

Q?? 이게 뭔 말이야 생성하는 전략을 호출한다니.. 
2번째 기능을 인터페이스로 만들어두고 이 메소드를 통해 prepareStatement를 생성하는 전략을 호출하면 됩니다

context가 만들어둔 Connection을 전달 받고 PreparedStatement를 만들고 PreparedStatement를 돌려준다.


이 내용을 인터페이스로 정의

```

public interface StatementStrategy {

    PreparedStatement makePreparedStatement(Connection conn) throws SQLException;

}

```

인터페이스 구현
```

public class DeleteAllStatement implements StatementStrategy{

    @Override
    public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {
        PreparedStatement ps = conn.prepareStatement("TRUNCATE TABLE USER2");
        return ps;
    }
}

```
사용
```

	this.conn = connectionManager.getConnection();
	StatementStrategy strategy = new DeleteAllStatement();
	ps = strategy.makePreparedStatement(conn);
	ps.executeUpdate();

```

### 어라? UserDAO는 StatementStrategy의 구현체인 DeleteAllStatement를 알고 있다.
UserDAO를 컨텍스트라고 하는데 컨텍스트가 인터페이스뿐 아니라 구현 클래스까지 직접 알고 있다는 건..
전략패턴에도 맞지 않고 OCP에도 맞지 않는다!


## DI적용을 위한 클라이언트/컨텍스트 변경

전략패턴에 따르면 컨텍스트가 어떤 전략을 사용하게 할 것인가는 컨텍스트를 사용하는 앞단의 클라이언트가 정한다.
컨텍스트는 전달받은 그 구현체 클래스를 사용한다.

잘 보면..

1. 클라이언트가 구현체를 결정
2. 구현체를 컨텍스트에 전달
3. 컨텍스트는 구현체를 인터페이스에 전달
4. 구현체 사용됨

클라이언트는 UserDaoFactory
컨텍스트 UserDAO
인터페이스 StatementStrategy
구현체 DeleteAllStatement

초난감 UserDAO를 벗어나게 해줬던 패턴과 같다고 생각합니다.

그럼 현재 컨텍스트에 있는
```
StatementStrategy strategy = new DeleteAllStatement();
```
이 부분이 클라이언트로 빠지면 됩니다.
우선 컨텍스트를 메소드로 분리합니다.

Q?? 여기서 잘 모르겠는것
클라이언트, 컨텍스트 ? 
클라이언트는 의존 관계를 알고 구현체를 주입해주는 오브젝트고
컨텍스트는 해당 클래스의 중복되지만(?) 주요한 기능을 컨텍스트라고 함.

#### 컨텍스트를 분리합니다.
```
    public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
        Connection conn = null;
        PreparedStatement ps = null;

        try {
            conn = connectionManager.getConnection();
            ps = stmt.makePreparedStatement(conn);
            ps.executeQuery();
        }catch (SQLException e){
            throw e;
        }finally {
            if(ps!=null){ try{ ps.close(); } catch (SQLException e){} }
            if(conn!=null){ try{ conn.close(); } catch (SQLException e){} }
        }

    }
```

#### 클라이언트가 깨끗해졌습니다
```
public void deleteAll() throws SQLException {
	StatementStrategy strategy = new DeleteAllStatement();
	jdbcContextWithStatementStrategy(strategy);
}
```

## 마이크로 DI
DI의 다양한 형태.
제 3자의 도움을 통해 두 오브젝트 사이가 유연하게 되는것.
이 개념만 잘 배운다면 DI를 다양하게 만들 수 있습니다.
일반적으로 DI는 의존관계에 잇는 두개의 오브젝트와 이 관계를 설정해주는  DI 컨테이너(오브젝트 팩토리)
그리고 이를 사용하는 클라이언트라는 4개의 오브젝트가 생성 됩니다.

deleteAll : 클라이언트
jdbcContextWithStatementStrategy : DI 컨테이너(오브젝트 팩토리), 컨텍스트
StatementStrategy : 인터페이스
DeleteAllStatement : 구현체

## 전략 클래스의 추가
user를 추가하는 add() 메소드도 전략패턴을 적용해봅시다!

어라? 만들고 나서 보니 user는 어디서 가져오지 싶습니다
```
public class AddUserStatement implements StatementStrategy{
    @Override
    public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {
        
        PreparedStatement ps = conn.prepareStatement("INSERT INTO USER2(ID, NAME, PASSWORD) VALUES(?,?,?)");
        ps.setString(1,user.getId());
        ps.setString(2,user.getName());
        ps.setString(3,user.getPassword());

        ps.executeUpdate();
        
        return ps;
    }
}
```
이럴 땐 UserVO를 생성자를 통해 받을 수 있도록 해줍시다.

```
public class AddUserStatement implements StatementStrategy{

    UserVO user;

    public AddUserStatement(UserVO user) {
        this.user = user;
    }

    @Override
    public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {

        PreparedStatement ps = conn.prepareStatement("INSERT INTO USER2(ID, NAME, PASSWORD) VALUES(?,?,?)");
        ps.setString(1,user.getId());
        ps.setString(2,user.getName());
        ps.setString(3,user.getPassword());

        ps.executeUpdate();

        return ps;
    }
}

```

```
	public void add(UserVO user) throws SQLException {

        StatementStrategy strategy = new AddUserStatement(user);
        jdbcContextWithStatementStrategy(strategy);

    }
```
완성~!


## 전략과 클라이언트의 동거

클라이언트는 UserDAO의 add(), deleteAll()이 클라이언트입니다.
전략은 현재 AddUserStatement, DeleteAllStatement 이 두 클래스 입니다.

토비는 DAO의 메소드 마다 전략을 생성하고 있다는걸 전략과 클라이언트의 동거라고 표현합니다.

파일의 개수가 굉장히 많이 늘어날 것 같고 런타임 시 다이내믹하게 DI를 해준다는 점을 제외하면 템플릿 메소드 패턴 적용과 별 다를게 없어보입니다.
또한 add 할 때 처럼 UserVO를 필요로 한다거나 하는 부가정보가 필요할 경우
그 때마다 생성자를 생성해야합니다.

## 로컬 클래스
클래스 파일이 많아지는 문제는 
StatementStrategy 전략 클래스를 매번 독립된 파일로 만드는게 아니라 
UserDAO의 내부 클래스로 정의 해버리는 것입니다.
DeleteAllStatement, AddUserStatement 둘 다 UserDAO 외엔 아무데서도 쓰이지 않습니다.
UserDAO에 강하게 결합 되어 있다는 뜻입니다.

```
public void add(final UserVO user) throws SQLException {
	jdbcContextWithStatementStrategy( new StatementStrategy(){
			@Override
			public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {
				PreparedStatement ps = conn.prepareStatement("INSERT INTO USERS2(ID, NAME, PASSWORD) VALUES(?,?,?)");
				ps.setString(1,user.getId());
				ps.setString(2,user.getName());
				ps.setString(3,user.getPassword());
				ps.executeUpdate();
				return ps;
			}
		}
	);
}
```

```
public void deleteAll() throws SQLException {
	jdbcContextWithStatementStrategy( new StatementStrategy(){
		  @Override
		  public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {
			  return conn.prepareStatement("TRUNCATE TABLE USERS2");
		  }
	  }
	);
}
```

완성~!

## 컨텍스트와 DI
전략패턴의 구조로 보면 
- 클라이언트 : dao의 메소드
- 전략 : 익명 내부 클래스
- 컨텍스트 : jdbcContextWithStatementStrategy

컨텍스트는 UserDAO의 PreparedStatement를 실행하는 기능을 가진 메소드에서 공유합니다.
그런데 JDBC의 일반적인 작업 흐름을 가진 jdbcContextWithStatementStrategy를 다른 DAO에서도 사용 할 수 있도록 하려고 합니다.

jdbcContextWithStatementStrategy를 DAO 밖으로 독립 시킵시다!

## 클래스 분리
아까 jdbcContextWithStatementStrategy 메소드가 하던 일을 그대로 class로 분리했습니다.
```
public class JdbcContext {
    
    private ConnectionManager connectionManager;
    
    public void setConnectionManager(ConnectionManager connectionManager){
        this.connectionManager = connectionManager;
    }
    
    public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException{
        Connection conn = null;
        PreparedStatement ps = null;

        try {
            conn = this.connectionManager.getConnection();
            ps = stmt.makePreparedStatement(conn);
            ps.executeQuery();
        }catch (SQLException e){
            throw e;
        }finally {
            if(ps!=null){ try{ ps.close(); } catch (SQLException e){} }
            if(conn!=null){ try{ conn.close(); } catch (SQLException e){} }
        }
    }
    
}
```

## 템플릿과 콜백
템플릿과 콜백의 동작원리

템플릿 : 고정된 작업 흐름을 가진 코드를 재사용한다. ( JdbcContext? )
콜백 : 템플릿 안에서 호출되는 것을 목적으로 만들어진 오브젝트. ( StatementStrategy? )

여러개의 메소드를 가진 인터페이스를 사용한는 전략패턴과 다르게
콜백은 단일 메소드의 인터페이스입니다.

### 템플릿의 작업 흐름 중 특정 기능을 위해 한 번 호출되는 경우가 일반적이기 때문입니다.

```
this.jdbcContext.workWithStatementStrategy( new StatementStrategy(){
	  @Override
	  public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {
		  return conn.prepareStatement("TRUNCATE TABLE USERS2");
	  }
  }
);
```
StatementStrategy처럼 하나의 메소드만 가진 인터페이스를 익명 내부 클래스로 만들어서 사용하는 것을 콜백이라고 합니다!

JdbcContext에서는 

#### 템플릿인 workWithStatementStrategy() 메소드에서 생성한 Connection 오브젝트를

#### 콜백(StatementStrategy)의 메소드인 makePreparedStatement를 실행 할 때 파라미터로 넘겨줍니다.

## 콜백의 분리와 재활용

```
this.jdbcContext.workWithStatementStrategy( new StatementStrategy(){
              @Override
              public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {
                  return conn.prepareStatement("TRUNCATE TABLE USERS2");
              }
          }
        );
```

```
this.jdbcContext.workWithStatementStrategy( new StatementStrategy(){
                @Override
                public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {
                    PreparedStatement ps = conn.prepareStatement("INSERT INTO USERS2(ID, NAME, PASSWORD) VALUES(?,?,?)");
                    ps.setString(1,user.getId());
                    ps.setString(2,user.getName());
                    ps.setString(3,user.getPassword());
                    ps.executeUpdate();
                    return ps;
                }
            }
        );
```

위를 보면 deleteAll과 add는 중복되는 코드부분이 많이 보입니다 이를 분리합니다.

```
	public void deleteAll() throws SQLException {
        executeSql("TRUNCATE TABLE USERS2");
    }

    private void executeSql(final String query) throws SQLException{
        this.jdbcContext.workWithStatementStrategy(
            new StatementStrategy(){
                @Override
                public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {
                    return conn.prepareStatement(query);
                }
            }
        );
    }
```

또 이걸 UserDAO 외에도 많은 DAO 클래스들이 사용해야하므로 JdbcContext 클래스로 옮겨놓겠습니다.

```
public void executeSql(final String query) throws SQLException{
	workWithStatementStrategy(
		new StatementStrategy(){
			@Override
			public PreparedStatement makePreparedStatement(Connection conn) throws SQLException {
				return conn.prepareStatement(query);
			}
		}
	);
}
```

```
public void deleteAll() throws SQLException {
	this.jdbcContext.executeSql("TRUNCATE TABLE USERS2");
}
```

템플릿 메소드 패턴과 전략 패턴의 차이점










































