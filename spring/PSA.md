# PSA(Portable Service Abstraction)

Spring의 PSA란, 특정 기술과 환경에 종속되지 않고 일관된 프로그래밍 모델을 제공하여 개발자가 특정 기술에 대해 신경 쓰지 않고 비즈니스 로직에 집중할 수 있도록 하는 설계 원칙이다. PSA는 다양한 기술과 라이브러리를 추상화하여, 개발자가 손쉽게 교체하거나 확장할 수 있는 유연한 구조를 제공한다.

예를 들면, `@Transactional` 어노테이션의 내부를 까보면 트랜잭션 관리는 `AbstractPlatformTransactionManager`라는 추상화를 기준으로 하는 것이고, JPA를 사용할 때는 이를 구현한 `JpaTransactionManager` 를 통해 관리하고 JDBC를 사용한다면 이를 구현한 `DatasourceTransactionManager` 를 통해 관리할 수 있다.
다시 말하면, 여기도 결국 **DI**다.

그러면 Service Abstraction 까지는 명확한데, Portable은 뭘까?
'이동 가능성'이라는 사전적 의미를 가진 Portable이 여기서는 **비즈니스 로직의 수정없이 언제든지 변경할 수 있는 것**을 의미한다.