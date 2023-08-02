## jpa가 식별자 지정 시 insert를 하기 전에 그 대상을 select 하는 현상
jpa 에서 새 엔티티를 생성하고 저장할 때, 식별자(pk)를 직접 주게 되면 해당 데이터가 이미 존재하는지 `select`를 한 번 하게된다. 식별자 값이 `null`이라면 새로운 엔티티로 판단하고 `INSERT`를 하게 되고, `null` 이 아니라면 더티 체킹 이후 필요에 따라 `UPDATE`를 하게 된다.

당연하게도, 불필요한 `select` 없이 데이터 저장을 할 수 있다. `EntityManager`를 주입 받아 사용할 수도 있지만, Spring Data JPA를 사용하는 입장에서 `EntityManager` 를 직접 다루는 일은 최대한 피하고 싶다. 이럴 때는 `Persistable` 인터페이스를 상속받아 `isNew` 메소드와 `getId` 메소드를 오버라이드 하여 해결할 수 있다.

```Java
@Entity
public class TestEntity implements Persistable<String> {

    // ...

    @Override
    public String getId() {
        return this.id;
    }

    @Override
    public boolean isNew() {
        return /* 신규 엔티티일 때 true를 리턴하는 로직 */;
    }
}
```

## Transaction marked as rollbackOnly
트랜잭션 메소드에서 예외가 발생한 경우 해당 트랜잭션은 롤백이 될 트랜잭션으로 마킹이 된다.

이는 트랜잭션의 `Atomocity` 와 관련이 있는데, 생각해 보면 예외가 터져서 해당 트랜잭션은 반영이 되지 않는 다는 것은 당연한 일이다.

그런데 이것이 트랜잭션 전이와 함께 일어나면 조금 혼동되는 상황이 발생할 수 있다.

`@Transactional`이 붙어있는 `outerMethod`와 `innerMethod` 가 있을 때, `innerMethod`에서 발생한 `RuntimeException`을 `outerMethod`가 `catch` 해주더라도, `innerMethod` 내에서는 예외 처리가 이루어지지 않았고, 이는 트랜잭션을 롤백으로 마킹했으므로 트랜잭션 전이에 의해 해당 트랜잭션은 재사용이 불가하다.