# JPA
JPA는 Java 객체와 관계형 DB 사이의 매핑을 쉽게 처리할 수 있도록 도와주는 ORM(Object-Relational Mapping)

JPA의 기본적인 쿼리 작성 방법은 JPQL(Java Persistence Query Language) 이나 Criteria API

# QueryDSL
QueryDSL 은 타입 세이프한 쿼리를 생성할 수 있는 프레임워크로, SQL 뿐만 아니라 JPA, JDO, Lucene, MongoDB 등 다양한 데이터 저장소에 대한 쿼리를 작성할 수 있다.

타입 체크를 통해 쿼리의 오류를 미리 잡아낼 수 있다.

# JPA 와 QueryDSL의 관계
둘은 함께 사용할 수도, 함께 사용하지 않을수도 있다.

### JPA를 사용하는 QueryDSL 예시
```Java
import static com.myapp.QUser.user;

import com.myapp.User;
import com.querydsl.jpa.impl.JPAQuery;
import javax.persistence.EntityManager;

public class UserDAO {

    private EntityManager em;

    public UserDAO(EntityManager em) {
        this.em = em;
    }

    public User findUserById(Long id) {
        JPAQuery<User> query = new JPAQuery<>(em);
        return query.from(user)
                .where(user.id.eq(id))
                .fetchOne();
    }
}
```
`JPAQuery` 객체, `EntityManager` 를 이용하여 DB와 상호작용 한다.

`JPAQuery`를 직접 생성하는 대신 `JPAQureyFactory`를 사용할수도 있다.

```Java
import static com.myapp.QUser.user;

import com.myapp.User;
import com.querydsl.jpa.impl.JPAQueryFactory;
import javax.persistence.EntityManager;

public class UserDAO {

    private JPAQueryFactory queryFactory;

    public UserDAO(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    public User findUserByUsername(String username) {
        return queryFactory.selectFrom(user)
                .where(user.username.eq(username))
                .fetchOne();
    }
}
```

### JPA를 사용하지 않는 QueryDSL 예시
```Java
import static com.myapp.QUser.user;

import com.myapp.User;
import com.querydsl.sql.SQLQuery;
import com.querydsl.sql.SQLQueryFactory;
import com.querydsl.sql.Configuration;
import javax.sql.DataSource;
import java.sql.Connection;

public class UserDAO {

    private DataSource dataSource;
    private Configuration configuration;

    public UserDAO(DataSource dataSource, Configuration configuration) {
        this.dataSource = dataSource;
        this.configuration = configuration;
    }

    public User findUserById(Long id) throws Exception {
        try (Connection conn = dataSource.getConnection()) {
            SQLQueryFactory queryFactory = new SQLQueryFactory(configuration, dataSource);
            SQLQuery<User> query = queryFactory.query();
            return query.from(user)
                    .where(user.id.eq(id))
                    .fetchOne();
        }
    }
}
```
`SQLQuery` 객체를 사용하여 쿼리를 새성하고, JDBC의 `DataSource`를 이용하여 DB와 상호작용 한다.