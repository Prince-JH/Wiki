## Transaction marked as rollbackOnly
트랜잭션 메소드에서 예외가 발생한 경우 해당 트랜잭션은 롤백이 될 트랜잭션으로 마킹이 된다.

이는 트랜잭션의 `Atomocity` 와 관련이 있는데, 생각해 보면 예외가 터져서 해당 트랜잭션은 반영이 되지 않는 다는 것은 당연한 일이다.

그런데 이것이 트랜잭션 전이와 함께 일어나면 조금 혼동되는 상황이 발생할 수 있다.

`@Transactional`이 붙어있는 `outerMethod`와 `innerMethod` 가 있을 때, `innerMethod`에서 발생한 `RuntimeException`을 `outerMethod`가 `catch` 해주더라도, `innerMethod` 내에서는 예외 처리가 이루어지지 않았고, 이는 트랜잭션을 롤백으로 마킹했으므로 트랜잭션 전이에 의해 해당 트랜잭션은 재사용이 불가하다.