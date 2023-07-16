# PostgreSQL vs MySQL

1. SQL 준수:
    PostgreSQL은 ANSI 표준을 거의 완벽하게 준수하며, 사용자 정의 데이터 타입, 저장 프로시저, 트리거, 뷰 등과 같은 고급 기능을 제공한다. 반면에 MySQL은 ANSI 표준에서 벗어나는 몇 가지 독특한 구문을 사용하며, 이러한 구문은 다른 RDBMS와 호환되지 않을 수 있다. 예)`REPLACE INTO`, `INSERT INTO`

2. Replication:
    MySQL은 Physical Replication (Master-Slave Replication, Group Replication)만 지원한다. 반면 PostgreSQL은 Physical Replication과 Logical Replication을 지원한다.

3. Extensibility:
    PostgreSQL은 사용자 정의 함수, 연산자, 집계 함수, 데이터 타입 등을 지원한다. 


4. Performance:
    - 읽기 성능

        PostgreSQL은 클라이언트마다 별도의 백엔드 프로세스를 사용하는 프로세스 기반 모델을 사용한다. 이는 안정성과 견고성에 이점을 제공하지만, 프로세스간 컨텍스트 스위칭에 비용이 발생하므로 특정 경우에는 성능에 부담을 줄 수 있다.

        반면에 MySQL(InnoDB)는 스레드 기반 모델을 사용하며, 이는 각 클라이언트 연결에 대해 별도의 스레드를 생성한다. 스레드는 같은 메모리 공간을 공유하므로, 스레드간 컨텍스트 스위칭 비용이 적다. 이는 동시성 처리와 관련하여 높은 성능을 제공할 수 있지만, 스레드 간 격리가 덜 이루어져서 하나의 스레드에서 발생한 문제가 다른 스레드에 영향을 미칠 수 있다.
    - 쓰기 성능

        PostgreSQL은 MVCC(Multi-Version Concurrency Control)을 사용하여 트랜잭션을 관리하며, 이는 동시에 많은 수의 쓰기 작업을 처리하는 데 유리하다. 

        MVCC를 통해 데이터를 읽는 사람과 작성하는 사람이 서로 통신하여 PostgreSQL 데이터베이스를 동시에 관리할 수 있다. 따라서 데이터와 통신해야 할 때마다 읽기-쓰기 잠금을 할 필요가 없으므로 성능이 향상된다.
    

5. GIS 기능:
    PostgreSQL은 PostGIS 확장을 통해 지리적 정보 시스템(GIS) 기능을 지원한다. MySQL도 GIS 기능을 지원하지만, PostgreSQL의 PostGIS만큼 강력하지는 않다.