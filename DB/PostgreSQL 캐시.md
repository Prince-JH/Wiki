# PostgreSQL 캐시

## PreparedStatement vs Statement
- **PreparedStatement**는 SQL 문장을 미리 컴파일하고 계획을 캐시에 저장하기 때문에 동일한 SQL 문장을 반복적으로 실행할 때, 이미 컴파일된 실행 계획을 재사용하므로 성능이 향상된다. 

    또한, PreparedStatement를 사용하면 처음 한 번만 SQL을 보내고, 이후에는 식별자와 파라미터 값만 보내게 된다. 따라서, 네트워크 I/O 또한 줄어든다. 또한 SQL 쿼리에 동적 파라미터를 삽입할 때 SQL Injection 공격을 방지할 수 있다.

- 반면에 일반 **Statement를** 사용하면 SQL 문장이 실행될 때마다 파싱, 컴파일, 실행 계획을 생성해야 하기 때문에 매번 CPU 자원을 사용하여 비효율적이다. 

이러한 이유로, Spring Boot 등 프레임워크가 DML을 날릴 때 기본적으로 PreparedStatement를 사용하고 있는듯 하다.