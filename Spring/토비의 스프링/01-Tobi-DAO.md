# 오브젝트와 의존관계

# DAO

**DAO**란 (Data Access Object)라고 하고 이것은 **데이터를 조회하거나 조작하는 기능**을 전담하게 하는 객체를 말합니다.

보통 **DAO**를 사용하기 위해서는 **DTO**라는 (Data Transfer Object)를 만들게 됩니다.

정말 간단하게 **DTO**는 사용자가 어떤 데이터를 받는지 객체에 저장하는 것을 말합니다.

이들은 Getter/Setter로 묶여있어 데이터를 접근하거나 반환받을 수 있습니다.

우선 User라는 **DTO**를 만들도록 하겠습니다.

```java
public class User{
	String id;
	String name;
	String password;

	public String getId(){
		return id;
	}

	public void setId(String id){
		this.id = id;
	}

	public String getName(){
		return name;
	}

	public void setName(String name){
		this.name = name;
	}

	public String getPassword(){
		return password;
	}

	public void setPassword(String password){
		this.password = password;
	}

}
```

이러한 **DTO**는 그냥 현재 id, name, password로 구현이 된 상태입니다.

이런식으로 Getter/Setter만 있는 것이 **DTO**입니다

이러한 **DTO**에 데이터 베이스에서 꺼낸 정보가 들어가게 됩니다. 

이러한 DTO에 꺼내고 저장하는 역할을 하는것이 바로 **DAO**입니다.

일단 데이터 베이스를 구축하기 위해서 테이블을 하나 만듭니다

필드와 정보는 다음과 같습니다

[User 테이블](https://www.notion.so/42e76b9ae31340f2a4d0c1a866326900)

다음처럼 테이블을 구성한다. 이때 처음 배운다는 가정하에 **Primary Key**를 모를 수도 있다고 생각합니다.

이러한 **Primary Key**는 고유의 값이라고 생각하면 편합니다.

**Primary Key**라고 설정을 하면 아무도 중복의 값을 가질 수 없습니다

이러한 테이블을 생성하기 위한 SQL 문법은 다음과 같습니다

```sql
create table users(
	id varchar(10) primary key,
	name varchar(20) not null,
	password varchar(20) not null
)
```

이렇게 입력을하고 실행을 하면 DB가 생성이 됩니다.

이제 UserDao를 생성해봅시다!!

이름은 당연히 UserDao라는 이름으로 클래스를 생성할 것 입니다. 

또한, 우선적으로 다음과 같은 메서드를 생성할 것 입니다

- 새로운 사용자를 생성하는 기능 (add 라는 메서드)
- 아이디를 가지고 사용자의 정보를 가져오는 기능 (get 이라는 메서드)

이러한 기능을 **JDBC**를 이용해서 진행할 예정입니다

### JDBC의 작업순서

**JDBC**를 작업하기 위해서는 다음과 같은 순서를 통해서 작업을 할 수 있습니다.

우리는 스프링을 배우고 있고 JDBC도 물론 배우겠지만 중점적으로 할 것은 아니기 때문에 일단 이런과정을 통해서 사용이 되는구나 라고 알아두면 좋을 것 같습니다

- DB 연결을 하기 위해서 Connection이라는 객체를 만들어 연결한다
- SQL을 담은 Statement를 만든다
- 만들 Statement를 실행한다
- 조회해서 SQL 쿼리의 실행결과들을 ResultSet으로 받아서 User라는 미리만든 DTO에 넣어준다.
- 작업이 끝난 후는 이 때 생성한 Statement, ResultSet, Connection 들을 작업을 마친 후에 반드시 닫아주어야 합니다
- JDBC API가 만들어내는 예외는 직접 처리하거나 메소드에 throws를 넘겨서 메소드 밖으로 던지게 합니다.

그래서 생성하면 다음과 같은 코드가 나옵니다

```java
import java.sql.*;
//...//
public class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException{
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                "jdbc:mysql:// 내 쿼리주소","spring","book"
        );

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)"
        );
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public void get(String id) throws ClassNotFoundException, SQLException{
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                "jdbc:mysql:// 내 쿼리주소","spring","book"
        );

        PreparedStatement ps = c.prepareStatement(
                "select * from users where id = ?"
        );
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
}
```

여기서 Connection과 Class를 건너뛰고 설명하자면 

- `PreparedStatement`는 내가 입력하고자하는 SQL문이 들어가는 곳이다.
- `ps.setString(1,id)`이런식의 메서드 활용이 많은데 이 뜻은 SQL문의 첫 번째 ?의 값을 id라고 설정하는 것이다.
- `ps.executeQuery()`를 사용해서 쿼리를 실행해서 ResultSet에 저장한다.

그래서 이러한 코드를 짜게 되었습니다. 이제 테스트코드를 사용해서 테스트를 한번 해봅시다😊

테스트 코드는 `main()`메서드에 생성을해서 `add()`와 `get()`메서드를 검증 해보도록 합니다

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException{
	UserDao dao = new UserDao();
	
	User user = new User();
	user.setId("haecahn");
	user.setName("유해찬");
	user.setPassword("1234");
	
	dao.add(user);

	System.out.println(user.getId() + "제대로 나왔습니다");
	
	User user2 = dao.get(user.getId());
	System.out.println(user2.getName());
	System.out.println(user2.getPassword());

	System.out.println(user2.getId() + "제대로 나왔습니다");
	
}
```

이런식으로 확인하면 제대로 값이 나오는 것을 볼 수 있다. 

하지만 이러한 코드는 스프링의 개발을 한 원칙으로 살펴보면 무척이나 무성의한 코드로 볼 수 있다. 

이러한 것을 Spring 스타일로 개선하는 작업을 이 책에서는 진행하려고 합니다.

# DAO의 분리

## 관심사의 분리

객체지향에서는 모든것이 변합니다. 비지니스에서 A를 개발하라고 했다가 다시 B로 돌아와라 라고 할 수도 있고 이러한 오브젝트에 대한 설계와 구현한 코드는 계속 게속 바뀝니다.

그래서 우리가 중점적으로 생각해야할 것은 미래를 위해서 다시 잘 사용하고 유지보수가 쉽게 설계하는 것을 목표로 하는 것이 개발자들이 가장 염두해야할 사항입니다.

이런 것이 너무 미래라고 생각할지도 모르지만 당장 코드를 짜고나서 한 시간 뒤에도 이런 변화는 찾아올 수 있기 때문에 객체지향 기술을 모두 잘 활용해서 이를 자유롭고 편리하게 변경, 발전, 확장시킬수 있어야합니다

이러한 것을 보다 효율적으로 하기 위해서 **관심사의 분리**라는 프로그래밍의 기초 개념을 이용합니다.

이러한 **관심사의 분리**라는 것은 관심사가 같은 것 끼리는 하나의 객체 안으로

하지만, 관심사가 다르다면 서로 영향을 주지 않기 위해서 분리하는 것 입니다.

물론 코드 작성은 그냥 관심사를 상관없이 대충 비슷한것 끼리 짜는 것이 편하지만 

이를 분리해줌으로써 같은 관심에 효과적으로 집중할 수 있게 만들어주는 것이 좋습니다.

### UserDao의 관심사항

우리가 아까전에 작성한 UserDao를 다시 돌아보면서 관심사항에 대해서 정리를 해봅시다.

총 3가지로 나뉠 수 있습니다

1. DB와의 연결을 위해서 우리는 Connection을 연결하고 가져왔습니다. 이러한 것을 DB연결에 대한 관심이라고 하나로 정의할 수 있습니다.
2. DB에 보낼 SQL Satement를 만들고 실행하는 것으로 볼 수 있습니다.
3. 마지막으로 사용한 Statement, ReusltSet, Connection들을 닫아주는 일을 하는 것이 있습니다.

첫 번째로 DB연결에 대한 관심사를 살펴보겠습니다.

현재는 각 메소드마다 동일한 Connetion정보인데 이를 여러번 사용하는 것을 볼 수 있습니다.

우리는 앞으로 수천 개의 DAO코드를 만들수도 있는데 이러한 것이 계속해서 중복으로 있다면 충분히 성능에 영향을 끼칠 것이라고 생각합니다.

만약 진짜 Connection에 대한 정보가 바뀐다면 모든 DAO에 있는 수천개의 Connection의 코드를 바꿔주어야합니다. 이러한 코드를 방지하기 위해서 메소드로 추출할 것 입니다.

### 중복 코드의 메소드 추출

이러한 중복된 DB 연결 코드를 `getConnection`이라는 메서드를 만들어서 수정할 것입니다.

동일하게 `getConnection`이라는 메서드에는 연결하는 코드가 존재합니다.

이제 연결을 할때는 그저 하나의 메서드만 사용하면 됩니다.

```java
private Connection getConnection() throws ClassNotFoundException, SQLException{
		Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection(
            "jdbc:mysql:// 내 쿼리주소","spring","book"
    );
		
		return c;
} 
```

이제 DAO에 대한 메서드가 엄청 많아진다고 해도 Connection을 할 때 접속 Address가 바뀌거나 하는 일이 생기면 `getConnection` 코드만 고치면 됩니다.

이러한 코드는 다른 코드에 영향을 주지도 않고 관심 내용이 독립적이기 때문에 수정도 간단해집니다.

### 변경 사항에 대한 검증

이러한 코드가 당연하게도 이전 코드를 똑같이 가져다 썼기 때문에 괜찮다 라고 생각할 수 있지만

항상 검증은 필요하다. 어떠한 문제점이 발견될 수 도 있기 때문이다.

## DB 커넥션 만들기의 독립

현재 메서드를 분리해서 Connection을 할 수 있도록 만들었습니다.

하지만 이렇게 만든 기술을 A회사와 B회사가 구매를 주문해서 사용한다고 합니다.

그런데 A회사랑 B회사가 각기 다른종류의 DB를 사용하고있고 DB 커넥션을 가져오는데 독자적인 방법을 사용한다고 합니다

또한, DB커넥션을 UserDao라는 우리가 만들 프로그램을 구입하고 나서도 계속 변경될 가능성이 있다고 합니다.

이러한 상황에서는 우리가 `UserDao`에 대한 소스코드를 공개하고나서 `getConnection`을 수정해서 사용하라고 하면 편하지만

이러한 `UserDao`는 우리만의 독자적인 기술이고 이러한 코드를 공개하고 싶지 않다 라고 했을 경우는 어떻게 해야할까요?

정말 난감한 상황입니다. 공개를 하고 싶다고 해서 소스코드를 줘버리면 다른 회사가 우리의 기술을 모두 익혀버려서 주문이 안들어올 수 도 있고 이러한 과정에서는 어떠한 방법을 사용해야할까요?

### 상속을 통한 확장

이런경우는 우리가 사용하고 있는 `UserDao` 의 코드를 한 단계 더 분리하면 해결할 수 있습니다.

`getConnection`을 추상 메서들로 만들어 버리면 A라는 회사와 B라는 회사에서 알아서 필요한 부분을 구현할 수 있기 때문에 나머지 `UserDao`라는 클래스의 코드는 그냥 `getConneciton` 메서드를 따로 수정할 필요도 없고 코드를 수정할 필요도 없어집니다.

우리가 생성한 `UserDao`라는 추상 클래스를 통해서 A회사와 B회사에 판매를 하면 이제 각각 회사에서 자식클래스를 만들어서 `getConnection`을 원하는 형태로 만들 수 있습니다.

그렇게 된다면 자유롭게 회사 입맛대로 `getConnection`이라는 메서드를 자유롭게 구현할 수 있습니다.

또한, 우리 회사는 UserDao에 대한 구현 기능을 넘기지 않아도 가능합니다.

```java
// UserDao에서 추상 메서드로 만든 getConnection
public abstract Connection getConnection() throws ClassNotFoundException, SQLException;

// A회사의 새로만든 클래스에서 구현한 getConnection()
public class AUserDao extends UserDao { // UserDao를 상속받음
	public Connection getConnection() throws ClassNotFoundException,
				SQLException{
				// A회사의 코드
			}
}

// B회사의 새로만든 클래스에서 구현한 getConnection()
public class BUserDao extends UserDao { // UserDao를 상속받음
	public Connection getConnection() throws ClassNotFoundException,
				SQLException{
				// B회사의 코드
			}
}
```

잘 살펴보면 상속을 받아서 추상 메서드인 `getConnection`을 구현함으로서 클레스 레벨로 `AUserDao`와 `BUserDao`가 독립적으로 분리되는 것을 볼 수 있습니다.

이제 `UserDao`의 코드를 건들일 필요 없이 각 회사의 클래스를 만들어서 거기서 `getConnection`이라는 추성 메서드만 생성하면 됩니다.

이런식으로 `UserDao`처럼 슈퍼 클래스에 기본적인 로직의 흐름을 만들고 

그 기능의 일부를 추상 메서드, 오버라이딩이 가능한 `protected`메서드로 만들어서

서브 클래스에서 메서드를 필요에 맞게 구현하는 방법을 **템플릿 메서드 패턴**이라고 합니다.

이러한 **템플릿 메서드 패턴**이라는 디자인 패턴은 스프링에서 자주사용하는 패턴입니다

다른 관점에서 보면 `UserDao`에서의 `getConnection`이라는 메서드는 `Connection`타입의 객체를 생성한다는 기능을 정의 해놓은 추상 메서드라고 할 수 있습니다.

그래서 `UserDao`의 서브 클래스의 `getConnection` 메서드는 어떠한 `Connection` 클래스의 객체를 어떻게 생성할 것인지 결정하는 방법이라고도 볼 수 있습니다.

이러한 서브클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을

**팩토리 메서드 패턴**이라고도 부릅니다. 

이렇게 변화한 코드도 장점만 있는 것은 아닙니다. 이제 슈퍼클래스를 변화한다면 서브클래스를 수정하거나 다시 개발해야 할 수도 있습니다.

또한, 확장된 기능인 DB Connection을 생성하는 코드는 오직 `UserDao`에서 만 사용  가능하고 다른 DAO 클래스에서는 사용할 수 없다는 단점이 생깁니다.

## DAO의 확장

모든 오브젝트는 변합니다. 하지만 반드시 오브젝트가 다 동일한 방식으로 변하는 것은 아닙니다.

변화의 성격은 여러가지 특징에 따라 달라집니다. ex) JDBC API를 사용하는가?, JPA를 사용하는가?

현재까지는 추상 클래스를 만들고 이를 상속한 서브클래스에서 변화가 필요한 부분을 바꾸어 쓸수 있도록 변화시켰습니다.

하지만, 여러가지 단점도 많은 것을 알게 되었습니다. 상속이라는 방법은 좋았지만 그럼에도 불구하고 단점이 존재합니다.

### 클래스의 분리

그래서 다른 방식으로 클래스를 분리하면서 두 개으 관심사를 독립시키고 동시에 손쉽게 확장할 수 있는 방법을 알아 봅시다.

현재까지는 다음과 같이 분리를 진행하였습니다.

1. 독립된 메소드를 사용해서 분리했습니다. → `getConnection`을 만들기
2. 상 하위 클래스로 만들어서 분리 했습니다. → 추상 메서드 사용

이제는 상속관계도 아닌 완전히 독립적인 클래스로 만들어보려고 합니다.

DB 커넥션과 관련된 부분을 서브클래스로 하는 것이 아닌 새로운 클래스를 만들어서

`UserDao`가 이것을 이용하게 만들면 됩니다.

우리는 `SimpleConnectionMaker`라는 클래스를 만들고 `getConnection` 기능을 그안에 넣을 것 입니다.

그러면 `UserDao` 에서는 `SimpleConnectionMaker`를 `new`를 통해서 객체를 만들어서 각각의 메서드에서 사용할 수 있게 합니다.

이 말은 꼭 `add()`나 `get()`에서 `SimpleConnectionMaker`를 `new`를 통해 객체를 만들라는 소리가 아니라

한번만 `SimpleConnectionMaker`를 만ㄷ르어서 저장해두고 사용할 것 입니다.

```java
public class UserDao {
	private SimpleConnectionMaker simpleConnectionMaker; // 한 번만 생성자를 통해서 만든다.
	
	public UserDao(){
		simpleConnectionMaker = new SimpleConnectionMaker(); // 객체 생성 -> 생성자라 한번만 생성
	}
}
```

이렇게 생성자를 통해서 하나만 만들면 다른 메서드에서 어떻게 사용하는지 확인해보자.

```java
public void add(User user) throws ClassNotFoundException, SQLException{
		Connection c = simpleConnectionMaker.makeNewConnection();
}

public void get(String id) throws ClassNotFoundException, SQLException{
		Connection c = simpleConnectionMake.makeNewConnection();
}
```

그리고 이제 DB 커넥션 생성 기능을 독립시킨 `SimpleConnectionMaker`는 다음과 같이 만들 수 있다.

`makeNewConnection()`이라는 메서드를 통해서 Connect를 해준다. 이제 상속을 사용하지 않으니 더이상 `getConnection()` 기능을 하는 `makeNewConnection()`은 `abstract`가 아니여도 된다.

```java
public class SimpleConnectionMaker{
		public Connection makeNewConnection() throws ClassNotFoundException,
		SQLException{
				Class.forName("com.mysql.jdbc.Driver");
				Connection c = DriverManager.getConnection(
						"SQL 주소"
				)
				return c;
		}
}
```

이를 통해서 Connection을 반환해준다.

이렇게 해서 분리는 하였지만 다시 문제가 발생하였습니다

이제 `UserDao`의 코드는 `SimpleConnectionMaker`라는 특정 클래스에 종속이 되기 때문입니다.

그렇기 때문에 상속을 사용했을 때 처럼 `UserDao`의 코드 수정없이 A회사랑 B회사가 Connection을 생성하는 기능을 변경할 방법이 사라졌습니다.

우리가 해결하야하는 문제는 그래서 다음과 같습니다.

1. `SimpleConnectionMaker` 의 메서드의 문제
    
    현재 `makeNewConnection()`으로 DB 커넥션을 가져오게 만들었습니다 이것으로 각 메서드에 Connection을 할당해줍니다. 만약, 변경이 있으면 `add()`, `get()` 뿐만아니라 많은 메서드가 생성이 되었으면 이를 일일히 변경하는 것은 무척이나 비효율적인 문제입니다.
    
2. DB 커넥션을 제공하는 클래스가 어떤 것인지 `UserDao`가 구체적으로 알고 있어야한다는 것입니다.
    
    만약 다른회사에서 다른 클래스를 구현하려고 한다면 불가피하게 `UserDao`를 수정을 해야 합니다. 
    
    왜냐하면 `SimpleConnetionMaker`라는 클래스 타입의 인스턴스 변수까지 정의를 해놨기 때문에 `UserDao`자체를 다시 수정해야합니다. 
    
    현재 근본적인 원인으로는 `UserDao`가 DB 커넥션을 가져오는 클래스에 대해서 너무 많이 알고 있기 때문에 커넥션을 가져올 때마다 우리는 메서드가 이름이 뭐고 어떤 클래스가 사용되는지 모든것이 구체적인 방법에 종속됩니다. 
    
    그래서 다른회사가 현재 우리의 `UserDao` 기술을 구매하려고하면 자유롭게 확장하기 때문에 상속을 안쓴만 못합니다.
    

### 인터페이스의 도입

이를 해결하기 위해서는 현재 `UserDao`와 `SimpleConnectionMaker`가 너무 긴밀하게 연결되어 있기 때문에 그렇습니다.

이러한 둘이 긴밀하게 연결을 끊기 위해서 중간에 추상적인 느슨한 연결고리를 만들어주는 것이 좋습니다.

이러한 것을 인터페이스를 사용해서 공통적인 성격을 뽑아 내서 이를 따로 분리해내면 해결할 수 있을 것 같습니다.

우리가 인터페이스를 이용해서 접근하게 한다면 만약 `SimpleConnecitonMaker`가 아닌 `HaechanConnectionMaker`를 바꿔서 사용해도 신경쓸일이 없습니다. 

왜냐하면 인터페이스는 변하지 않기 때문입니다.

이제 우리는 다음과 같이 구현할 것입니다. 

`UserDao`에서는 `ConnectionMaker`라는 인터페이스를 사용합니다

`ConnectionMaker`는 `makeConnection()`이라는 메서드를 가지고 

이러한 `makeConnection()`이라는 메서드는 각 회사의 사정에 맞춰서 구현된 클래스에 의해서 직접 구현하면 됩니다.

이제 `ConnectionMaker` 인터페이스를 정의해 봅시다. 이때 DB 커넥션을 가져오는 메서드 이름은 `makeConnection()`이라고 정의 할 것입니다.

```java
public interface ConnectionMaker{
		public Connection makeConnection() throws ClassNotFoundException,
		SQLException;
}
```

이제 납품을 할때 다음과 같이 Interface를 사용해서 이제 `UserDao`와 함께 `ConnectionMaker`라는 인터페이스를 전달하면 각 회사에 맞게 인터페이스를 구현한 클래스를 만들고 `makeConnection()`메서드를 작성하면 됩니다.

그러면 인터페이스를 사용한 `UserDao`는 어떻게 변했냐면 다음과 같습니다.

```java
public class UserDao{
	private ConnecitonMaker connectionMaker;

	public UserDao(){
		connectionMaker = new AConnectionMaker();
	}

	public void add(User user) throws ClasNotFoundException, SQLException{
		Connection c = connectionMaker.makeConnection();
	}

	public void get(String id) throws ClassNotFoundException, SQLException{
		Connection c = connectionMaker.makerConnection();
	}
}
```

변하긴 했는데 여기서도 `connectionMaker = new AConnectionMaker()`에 대해서는 

아직도 `AConnectionMaker()`의 클래스의 생성자를 호출해서 객체를 생성하는 코드가 남아있다.

다른 곳에서는 이러한 정보들을 모두 지웠는데 결론적으로는 `AconnectionMaker()`라는 클래스 이름을 넣어서 객체를 만들지 않으면 사용을 할 수가 없다. 

결론적으로 모두 지우지 못해서 원점으로 돌아오게 되었다.

### 관계설정 책임의 분리

현재까지의 진행상황은 `UserDao`와 `ConnectionMaker`라는 두 개의 관심을 인터페이스를 쓰면서 분리를 완료했습니다

하지만, 어찌해도 `UserDao`는 어떤 Connection을 사용할지 생성자에 명시가 되어있는 것을 볼 수 있습니다.

결론적으로는 Connection을 다른 회사에서 사용하면 `UserDao`는 아직까지도 무조건 변경을 해야합니다.

이렇게 된 이유는 `UserDao`안에 우리가 아직 분리하지 않은, 또 다른 관심사항이 존재하기 때문입니다.

현재 `UserDao`안에는 어떤 `ConnectionMaker`구현 클래스를 사용할지 결정하는 `new AConnectionMaker()`라는 코드가 존재합니다.

이 새로운 객체를 생성하는 코드는 매우 짧고 간단하지만 그 자체로 충분히 독립적인 관심사를 담고 있습니다.

왜냐면 이러 코드는 현재 `UserDao`의 관심사인 JDBC API와 `User` 객체를 이용해서 DB에 정보를 넣고 빼는 것을 하는 것도 아니고

`ConnectionMaker`인터페이스로 대표되는 DB 커넥션을 어떻게 가져올 것인가라는 관심사도 아니기 때문입니다.

결론적으로는 `UserDao`와 `UserDao` 가 사용할 `ConnectionMaker`의 특정 구현 클래스 사이의 관계를 설정해주는 것에 대한 관심을 가지고 있다

`UserDao`가 지닌 관심사와 `new AconnectionMaker()`라는 객체를 생성하는 관심사가 충돌하고 있기 때문에

**결코 독립적으로 확장 가능한 클래스가 될 수 없습니다.**

`UserDao`를 현재는 사용하는 클라이언트는 적어도 하나 존재합니다.

이러한 `UserDao`의 오브젝트의 기능을 사용하는 클라이언트가 `UserDao`를 사용하기 전에 `ConnectionMaker`의 구현 클래스를 사용할지 결정을 하도록 만들어 봅시다.

결론적으로 지금 우리가 하는일은 `UserDao` 오브제긑와 특정 클래스로 만들어진 `ConnectionMaker` 오브젝트 사이에 관계를 설정해주려고 하는 것 입니다.

현재 우리가 사용하고 있는 코드인 `connectionMaker = new AConnectionMaker()`는 

`AconnectionMaker`의 오브젝트의 Reference를 `UserDao`가 가지고 있는 `connectionMaker`라는 변수에 넣어서 사용하게 함으로써 **사용**이라는 서로의 관계를 맺어줍니다.

이런식으로 객체 사이의 관계가 만들어지기 위해서는 **만들어진 객체가 있어야합니다.**

지금까지 우리는 **생성자**를 통해서 객체를 만들어 왔습니다.

하지만 꼭 생성자를 사용하지 않고 **외부에서 만들어준 것을 가져오는 방법**도 존재합니다.

우리는 현재까지 생성자를 통해서 `UserDao`의 코드 내에서 객체를 생성해주었습니다.

 

이제 외부에서 만든 객체를 **Parameter**를 통해 가져온다면 사용을 할 수 있습니다.

이런 방식으로 한번 다시 리팩토링을 해봅시다.

현재는 `connectionMaker = new AConnectionMaker()`를 사용하고 있기 때문에 `AConnectionMaker`를

`UserDao`가 불필요하게 알고 있습니다. 이러한 `UserDao`코드는 `ConnectionMaker` 인터페이스 외에는 어떤 클래스와도 관계를 가지고 있으면 안됩니다!!

클라이언트는 현재 `UserDao`를 자기가 사용해야 할 입장이기 때문에 `UserDao`의 세부 전략인

`ConnectionMaker`의 구현 클래스를 선택하고, 선택한 클래스의 객체를 생성하여 `UserDao`와 연결할 수 있습니다.

이러한 관심을 분리해서 클라이언트에게 떠넘기는 방식으로 **Parameter**를 통해 사용해 봅시다.

`UserDao` 객체가 사용할 `ConnectionMaker`를 파라미터를 통해 전달받을 수 있는 형식으로 바꾸면 될 것 같습니다.

```java
public UserDao(ConnectionMaker connectionMaker){
	this.connectionMaker = connectionMaker;
}
```

이런식으로 하면 클라이언트에게 책임을 넘겼기 때문에 `AConnectionMaker`가 사라진 것을 볼 수 있습니다.

이를 이제 넘겨주는 `UserTest`라는 클래스를 하나 만들어서 이러한 `UserTest`에서 원래 실행했던 `new AConnectionMaker()`를 생성하도록 합니다.

그리고 `UserTest`안에서 `UserDao`객체를 파라미터를 넘겨 생성하도록 합니다.

```java
public class UserDaoTest{
	public static void main(String[] args) throws ClassNotFoundException,
		SQLException{
			ConnectionMaker connectionMaker = new AConnectionMaker(); // 객체 생성
			
			//Userdao 객체를 생성해서 내가 사용할 ConnectionMaker 객체를 넘겨준다.
			UserDao dao = new UserDao(connectionMaker); 
	}
}
```

이제 이렇게 생성된 `UserDaoTest`는 `UserDao`와 `ConnectionMkaer` 구현 클래스와의 런타임 객체 의존 관계를 설정하는 책임을 담당해아합니다.

이런식으로 까지 변환이 완료되었다면 테스트를 해봅시다!!

이제 `UserDao`에 있으면 안 되는 다른 관심사항을 클라이억트로 책임일 떠넘기는 작업을 완료했습니다.

`UserDaoTest` 덕분에 A와 B회사는 자신들이 원하는 DB 접속 클래스를 만들어서 `UserDao`가 사용할 수 있도록 할 수 있습니다. 

그저 각자의 회사에서 만든 객체를 넘겨주기만하면 됩니다.

이렇게 만든 코드에서 DB 커넥션을 가져오는 방법을 어떻게든 변경해도 이제 `UserDao`에 영향을 주지는 않습니다. 

이제 DB 접속 방법에 대한 관심은 한 군데에 집중되어 DAO가 많아져도 DB접속 방법이 변경되었을 때 한 곳만 고쳐주어서 유지보수하기 편하게 되었습니다.

## 원칙과 패턴

지금까지 DAO 코드를 수정하면서 객체지향 기술의 여러가지 이론을 사용했습니다. 

이제 다시한번 우리가 사용한 객체지향 기술에 대해서 어떤 이론을 사용했는지 보도록 합시다.

### 개방 폐쇄 원칙(OCP)

개방 폐쇄 원칙(Open-Closed Priciple)을 이용한다면 지금까지 해온 리팩토링 작업의 특징과 최종적으로 개선된 설계와 코드의 장점이 무엇인지 효과적으로 설명이 가능합니다.

이 원칙은 간단히 설명하면 **클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야한다.**라는 것을 의미합니다.

우리가 작성한 `UserDao`는 DB 연결 방법이라는 기능을 확장하는데는 매우 열려있습니다.

또한, `UserDao`에 전혀 영향을 주지 않고도 얼마든지 기능을 확장할 수 있게 되어 있습니다.

그리고 `UserDao` 는 자신의 핵심 기능을 구현한 코드는 변화에 영향을 받지 않고 유지가 가능하기 때문에 변경에는 닫혀 있다고 말할 수 있습니다.

제일 처음 작성했던 코드를 다시 살펴보자. 우리는 DB Connection에 대해서 변경을 하려고 한다면 

DAO 내부의 모든 메서드에 대해서 수정을 해야 했었습니다. 

결론적으로 DB Connection을 확장하려면, DAO 내부도 변경해야 하기 때문에 개방 폐쇄 원칙을 잘 따르지 못한 설계라고 할 수 있습니다.

### 높은 응집도와 낮은 결합도

이러한 개방 폐쇄 원칙은 높은 응집도와 낮은 결합도 라는 소프트웨어 개발의 원리로 설명이 가능합니다.

- 높은 응집도
    - 응집도가 높다는 것은 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 뜻입니다.
- 낮은 결합도
    - 하나의 변경이 발생할 때 어려가지 모듈과 객체에 대해 변경 요구가 전파되지 않는 상태를 말합니다.

**높은 응집도**는 변화가 일어날 때 해당 모듈에서 변하는 부분이 크다는 것으로 설명이 가능합니다.

우리가 제일 처음 작성한 DAO에 대해서는 DB 커넥션을 바꾸면 어떤 일을 해야하냐면

1. 변경이 필요한 부분을 모든 코드를 뒤져서 찾는다.
2. 변경한 것이 DAO의 다른 기능에 영향을 주어서 오류를 발생시키지 않는지 확인해야한다.

하지만 우리가 만약 인터페이스로 DB연결 기능을 독립 시켰다면 

1. CoonectionMaker 구현 클래스를 새로만든다.

이게 끝이다. 시간이 대폭 단축될 수 있습니다. 인터페이스로 구현을 한다면 DAO에 대해서 변화하는게 없기 때문에 모든 DAO를 일일이 테스트를 하지 않아도 됩니다.

**낮은 결합도**는 높은 응집도 보다 더 민감한 원칙입니다.

책임과 관심사가 다른 객체 또는 모듈과는 느슨하게 연결된 형태를 유지하는 것이 바람직 하다는 것입니다.

현재 우리가 작성한 코드는 인터페이스를 통해서 느슨하게 연결되게 하였습니다. 

이렇게 변하면 독립적이기 때문에 결합도가 낮아지면 변화에 대응하는 속도가 높아지고 구성이 깔끔해집니다.

또한, 확장하기에도 매우 편리합니다.

### 전략 패턴

우리가 최종적으로 만든 UserDaoTest → UserDao → ConnectionMaker 구조는 디자인 패턴의 시각으로 보면 전략 패턴에 해당한다고 볼 수 있습니다.

전략 패턴이란 무엇이냐면

- 자신의 기능 맥락(Context)에서, 필요에 따라 변경이 필요한 알고리즘을 인터페이스로 통해 통째로 외부로 분리시키고, 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴을 말합니다

우리가 만든 DAO를 보면 `UserDao` 는 전략 패턴의 맥락(Context)에 해당됩니다.

`UserDao`가 기능을 수행하는 데 필요한 기능 중에서 변경 가능한 DB Connection이라는 알고리즘은

`ConnectionMaker`라는 인터페이스에 정의를 했습니다.

우리는 전략이 바뀔 때 즉, DB Connection을 하는 방식이 바뀔 때 `ConnectionMaker`를 변경만 하면 됩니다. 이래서 **전략 패턴**이라고 불립니다.

이러한 **전략 패턴**은 `UserDaoTest`와 같은 클라이언트의 필요성에 대해서도 잘 설명하고 있습니다.

전략 패턴은 Context(`UserDao`)를 사용하는 클라이언트(`UserDaoTest`)가 사용하는 전략(`ConnectionMaker`를 구현하는 클래스)를 Context의 생성자를 통해서 제공해주는 것이 일반적입니다.

### Reference

- 토비의 스프링 3.1 Vol. 1: 스프링의 이해와 원리
