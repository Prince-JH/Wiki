## bulk insert
### save vs saveAll
Spring Data Jpa는 벌크 인서트를 지원하지 않는다. 그렇다면 saveAll 메소드는 무엇일까?

`saveAll` 은 벌크 인서트와는 달리 save 메소드가 for 문을 돌며 일어나지만, AOP로 동작하는 `@Transactional` 을 한 번만 프록시 객체로 호출하고, 반복문은 프록시를 타지 않고 직접 호출한다. 
따라서 `save` 메소드를 반복문으로 사용했을 때 보다는 성능이 향상된다.

### SpringDataJpa는 왜 벌크인서트를 지원하지 않을까
사실 Spring Data Jpa가 지원하지 않기 보다는, Hibernate가 지원하지 않는 것인데, 이는 Hibernate의 기본 철학인 쓰기 지연(Write-Behind) 때문이다. 

쓰기 지연 때문에 마지막까지 영속성 컨텍스트에서 데이터를 가지며 flush 를 연기하게 되는데, `GenerationType.IDENTITY`방식 즉, PK 값을 auto_increment 하는 방식의 경우 실제 insert를 하기 전까지는 ID에 할당된 값을 알 수 없기 때문에 벌크 인서트가 불가하다.

### 벌크인서트를 하려면 어떻게 해야하나
방법은 다양하겠지만(Pure JDBC, ) 주로 Jdbc Template을 사용하여 벌크 인서트를 하는듯 하다.

또는, `GenerationType.SEQUENCE` 방식을 사용하고 `spring.jpa.properties.hibernate.jdbc.batch_size` 옵션을 주어 벌크 인서트를 적용할 수도 있다.

참고) 

https://www.baeldung.com/jpa-hibernate-batch-insert-update

https://dkswnkk.tistory.com/682

### IDENTITY vs SEQUENCE
기본적으로 DBMS에 따라 지원하는 기본키 채번 방식이 다르다.

`IDENTITY`전략은 기본키 생성을 DB에게 위임하기 때문에 DBMS 벤더에 의존적이다. MySQL과 PostgreSQL은 Auto Increment방식을 사용하는데,  PostgreSQL의 경우 Auto Increment 구현을 위해 내부적으로 시퀀스를 사용한다. 실제로 테이블을 만들 때 날아가는 쿼리를 보면 `id  bigserial not null`로 DDL이 날아가는데, `serial` 과 `bidserial` 구현을 위해 자동으로 시퀀스를 생성한다.

`SEQUNCE`전략 역시 DBMS에 의존적이다. MySQL은 `SEQUNCE` 전략을 지원하지 않으므로 PostgreSQL로 비교하여 보면, `IDENTITY` 방식 역시 내부적으로 시퀀스를 사용하기 때문에 동작 방식이 유사할 수 있다. 

그러나 큰 차이가 있는데, 이는 **시퀀스 할당 타이밍**이 다르다는 것이다.

`IDENTITY`방식은 엔터티가 데이터베이스에 저장될 때 (즉, INSERT문이 실행될 때) 시퀀스 값이 할당된다. `persist()`를 호출하는 즉시 insert 쿼리가 DB에 전달되며 쓰기 지연이 동작하지 않는다.
반면, `SEQUENCE`방식은 시퀀스 값을 미리 할당받을 수 있다. 따라서 여러 엔터티의 ID를 미리 가져온 후, 배치로 데이터베이스에 저장할 수 있다.
