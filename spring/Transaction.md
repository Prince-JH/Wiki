# Transaction

메소드 또는 클래스에 트랜잭션 처리를 적용하는 데 사용
-> All or Nothing

## Transaction 내부 구현
@Transactional은 AOP를 통해 Proxy형태로 실제 로직의 전/후에 transaction관련 로직이 실행되게끔 동작한다.
```Java
public void realLogic(){
    try {
        transactionBegin(); // AOP로 생성된 트랜잭션 시작 

        super.realLogic(); // 실제 로직

        commit();  // AOP로 생성된 커밋
    } catch(.... e) {
        rollback();  // AOP로 생성된 롤백
    }
}
```
여기서 짚고 넘어가야할 것이 있는데, `@Transactional` 은 Spring 프레임워크에서 제공하는 기능으로, JPA에 의존하지 않는다. 실제로 내부를 까보면 트랜잭션 관리는 `AbstractPlatformTransactionManager` 라는 추상화를 기준으로 하는 것이고, JPA를 사용할 때는 이를 구현한 `JpaTransactionManager` 가 있는 것 뿐이다.

JPA를 사용한다면 `@Transactional`은 아래와 같은 프로세스를 타게 된다.
1. 트랜잭션 시작: @Transactional 어노테이션이 붙은 메서드가 호출될 때, Spring은 새로운 트랜잭션을 시작하거나 이미 진행 중인 트랜잭션에 참여한다. -> Transaction 전이와 이어지는 내용

2. EntityManager 연결: 트랜잭션이 시작되면, Spring은 현재 트랜잭션과 연결된 EntityManager를 준비한다. EntityManager는 데이터베이스 연결을 나타내며, 쿼리 실행, 엔티티 상태 관리 등의 작업을 수행한다.

3. 비즈니스 로직 실행: 어노테이션이 붙은 메소드의 비즈니스 로직이 실행됩니다. 이 과정에서 데이터베이스의 변경 작업이 발생할 수 있다.

4. 커밋 또는 롤백: 메소드 실행이 정상적으로 완료되면, EntityManager는 데이터베이스에 대한 변경 사항을 커밋한다. 만약 실행 도중 예외가 발생하면, 트랜잭션은 롤백된다.

5. 트랜잭션 종료: 트랜잭션이 커밋되거나 롤백된 후, 트랜잭션은 종료되고, EntityManager는 연결을 닫는다.

## Transaction 전이(Propagation)
`@Transactional`이 또 다른 `@Transactional`을 만날 때 전이 옵션에 따라 합쳐지거나 나눠진다.
- `REQUIRED` : 이미 시작된 트랜잭션이 있으면 참여하고, 없으면 새로운 트랜잭션을 시작한다. (디폴트 속성)
- `SUPPORTS` : 이미 시작된 트랜잭션이 있으면 참여하고, 없으면 트랜잭션 없이 처리한다.
- `REQUIRED_NEW` : 항상 새로운 트랜잭션을 시작한다.
- `MANDATORY` : 이미 시작된 트랜잭션이 있으면 참여하고, 없으면 새로운 트랜색션을 시작하는 대신 예외를 발생시킨다. 혼자서는 독립적으로 수행되면 안되는 경우에 사용된다.
- `NOT_SUPPORTED` : 트랜잭션을 사용하지 않고 처리하도록 한다. 이미 진행중인 트랜잭션이 있다면 그 트랜잭션은 일시 중단된다.
- `NEVER` : 트랜잭션을 사용하지 않도록 강제한다. 이미 진행중인 트랜잭션 또한 허용하지 않으며, 있다면 예외를 발생시킨다.
- `NESTED` : 이미 실행중인 트랜잭션이 있다면 중첩하여 트랜잭션을 진행한다. 부모 트랜잭션은 중첩 트랜잭션에 영향을 주지만 중첩 트랜잭션은 부모 트랜잭션에 영향을 주지 않는다.

## Transaction 커밋 시점
`@Transactional` 이 적용되면, 해당 메소드가 종료될 때 트랜잭션이 커밋된다.
그러나, Spring Data JPA에서 `save` 메소드는 즉시 쓰기 작업을 하지 않는다.
메소드가 종료된 이후에 커밋이 날아가기 때문에, 메소드 내에서 발생한 `DataIntegrityViolationException` 등의 예외는 `@Transactional` 메서드의 **try-catch 블록 내에서 잡히지 않는다**.