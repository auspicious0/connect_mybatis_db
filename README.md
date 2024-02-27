# connect mybatis_db

 1. [마리아DB 설정](#1-mariadb-setting)
 2. [Eclipse 프로젝트 생성 및 드라이버 추가](#2-eclipse-프로젝트-생성-및-드라이버-추가)
 3. [Mybatis 설정 파일 작성](#3-mybatis-설정-파일-작성)
 4. [Java 클래스 및 인터페이스 작성](#4-java-클래스-및-인터페이스-작성)
 5. [실행결과](#5-실행결과)


## 1. mariadb setting
우선 vmware를 활성화 한 뒤 mariadb를 설치 후 접속한다. 
접속 후 Database exampledb를 생성한다. 코드는 아래와 같다.

```
CREATE DATABASE exampledb;
USE exampledb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(50)
);

INSERT INTO users (username, email) VALUES ('john_doe', 'john@example.com');
INSERT INTO users (username, email) VALUES ('jane_doe', 'jane@example.com');
```

또 외부 ip에서 접근하는 것이기 때문에 사용자를 하나 추가한 후 모든 호스트 접근을 허용하여야 한다. 코드는 아래와 같다.

```
CREATE USER 'ID'@'%' IDENTIFIED BY '비밀번호';
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```


## 2. Eclipse 프로젝트 생성 및 드라이버 추가
eclipse 접속 후 java project를 생성한다. mybatis를 통해 db에 접속하려면 Mybatis 와 MariaDB JDBC 드라이버를 프로젝트에 추가해야 한다. 
버전 및 상세 정보는 아래와 같다.
   
![image](https://github.com/auspicious0/connect_mybatis_db/assets/108572025/1d52e8e9-cf0c-4fc3-9208-201a412f28cb)
![image](https://github.com/auspicious0/connect_mybatis_db/assets/108572025/9b55337a-05b7-4301-b476-757b48036637)

## 3. Mybatis 설정 파일 작성
프로젝트의 resources 폴더 아래나 페키지 폴더에 설정파일을 추가/수정해야 한다. 필요한 설정파일은 mybatis-config.xml과 UserMapper.xml이다. 
mybatis-config.xml을 통해 접근하고자 하는 ip, db port, db id pw 등의 정보를 기록한다. 
코드는 아래와 같다.

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <typeAliases>
        <typeAlias type="User" alias="User"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="org.mariadb.jdbc.Driver"/>
                <property name="url" value="jdbc:mariadb://*.*.*.*:3306/exampledb"/>
                <property name="username" value="id"/>
                <property name="password" value="password"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!-- 매퍼 XML 파일의 위치를 지정 -->
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

UserMapper.xml에는 db에서 하고자하는 작업을 추가한다. 코드는 아래와 같다.

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="UserMapper">
    <select id="getAllUsers" resultType="User">
        SELECT * FROM users;
    </select>
</mapper>

```

## 4. Java 클래스 및 인터페이스 작성
필요한 Java 클래스는 4개, 인터페이스는 1개이다. 
클래스에는 User, MyBatisUtil UserDao, Test가 있고 인터페이스는 UserMapper이다. 
각각이 하는 역할과 코드를 살펴보자.

- User클래스는 데이터베이스의 사용자 정보를 표현하는 클래스이다. 필드로는 사용자의 ID, 사용자의 이름, 이메일이 있다.
아래는 코드이다. 

```

public class User {
	private int id;
	private String username;
	private String email;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	
}

```

- MyBatisUtil 클래스는 MyBatis의 SqlSessionFactory를 생성하는 클래스이다. 
getSqlSessionFactory() 메서드는 SqlSessionFactory 인스턴스를 반환한다. 이 인스턴스는 데이터베이스와의 연결을 설정하고 SQL 세션을 생성하는 데 사용된다.

 ```
import java.io.InputStream;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
public class MyBatisUtil {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static SqlSessionFactory getSqlSessionFactory() {
        return sqlSessionFactory;
    }
}
```

- UserMapper인터페이스는 데이터베이스에서 사용자 정보를 가져오기 위한 메서드를 선언한다. 이 인터페이스는 SQL 쿼리를 실행하기 위한 메서드를 정의한다. 

```

import java.util.List;
 

public interface UserMapper {
	List<User> getAllUsers();
}

```

- UserDao 클래스는 데이터베이스와 통신하는데 사용된다. getAllUsers() 메서드는 mybatis를 사용하여 데이터베이스에서 모든 사용자 정보를 가져온다. 
이 클래스는 데이터베이스와 연결 및 세션 관리를 한다고 볼 수 있다.

```
import java.util.List;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

public class UserDao {
    public List<User> getAllUsers() {
        SqlSessionFactory sqlSessionFactory = MyBatisUtil.getSqlSessionFactory();
        try (SqlSession session = sqlSessionFactory.openSession()) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            return mapper.getAllUsers();
        }
    }
}

```

- Test 클래스는 프로그램을 실제로 실행하는 클래스이다. UserDao를 사용하여 사용자 정보를 가져온 후 출력한다.

```
import java.util.List;
public class Test {
	public static void main(String[] args) {
		UserDao userdao = new UserDao();
		List<User> users = userdao.getAllUsers();
		for(User user: users) {
			System.out.println("ID: " + user.getId() + ", Username: " + user.getUsername() + ", Email: " + user.getEmail());		
		}
			
	}
}

```

## 5. 실행결과
![image](https://github.com/auspicious0/connect_mybatis_db/assets/108572025/dec19de9-85ed-444b-9501-28ce88d2538f)

 
