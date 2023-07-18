# PostgreSQL HA

## 클러스터 vs 레플리케이션

클러스터는 기본적으로 멀티 마스터 구조로, 모든 노드가 읽기 및 쓰기 작업을 수행할 수 있기 때문에  기본적으로 높은 가용성과 병렬 처리가 가능하지만, 데이터 일관성 유지를 위한 복잡한 문제를 일으킬 수 있다. 또한, 모든 마스터 노드가 일관된 상태를 유지해야 하므로, 데이터가 동시에 여러 위치에서 변경될 때 충돌이 발생할 수 있다.

Replication 구조는 일반적으로 마스터와 하나 이상의 슬레이브로 구성된다. 모든 쓰기 작업은 마스터에서 이루어지고, 슬레이브는 읽기 작업과 가용성을 위한 백업 역할을 한다. 이 구조는 데이터 일관성 유지가 비교적 쉽지만, 마스터가 실패하면 복구 시간 동안 쓰기 작업이 중단될 수 있다. 다만, auto failover 설정을 통해 다른 슬레이브를 마스터로 승격시키는 방법으로 이 문제를 완화할 수 있다.

## Replication 방식
![](2023-07-14-10-42-21.png)

- WAL(Write Ahead Log) 기반 Physical Replication 방식
    - WAL: DB 변경 사항을 저장한 Log
        - Log Shipping: WAL 파일 자체를 전달하는 방식
        - Streaming: 로그 내용을 전달하는 방식
    - DB의 모든 변경 사항을 물리적으로 복제
    - Standby 서버는 Primary 서버의 정확한 복사본이 됨
    - HA 및 DR의 용도
- Logical Replication
    - 복제 식별자(PK)를 기반으로 변경 사항을 복제하는 방식 (블록 address 를 바이트 단위로 복제하는 physical replication과 반대 개념)
    - Pirmary 서버에서 발생하는 변경사항을 선택적으로 복제 가능
        - 뿐만 아니라, 다른 버전의 PostgreSQL 로도 복제 가능
        - 운영 중인 DB를 새 버전으로 마이그레이션 할 때 사용
    - pub sub 모델 사용
        - 변경 사항을 synchronization worker가 감지

## Pgpool-II
PostgreSQL 의 HA 를 구성할 때, auto failover 를 구성할 수 있는 선택지 중 하나이다. 뿐만 아니라, load balancing, connection pool 등의 기능도 제공하고 있다.

Pgpool 이 클라이언트 요청을 DB 서버로 분배하는 동작 모드는 다음과 같다.
- Load Balance mode: 읽기 쿼리를 여러 PostgreSQL 서버에 분산하여 요청한다.
  - `statement_level_load_balance` 값에 따라 load balancing 전략이 조금 달라진다. on 이면 매 쿼리마다 요청하는 DB 서버가 달라지고, off 이면 세션을 연결할 때 쿼리를 요청할 DB가 결정된다. 
- Master/Slave mode: 쓰기 쿼리는 마스터 서버에, 읽기 쿼리는 슬레이브 서버에 라우팅한다.
- Replication mode: 여러 PostgreSQL 서버가 같은 데이터를 가지도록 분산 복제 저장한다.

### Pgpool 은 어떤 DB가 마스터인지 어떻게 알 수 있을까?
위의 기능들을 보면 Pgpool은 마스터를 감지하여 쿼리를 라우팅한다. 
이는 `pgpool.conf`의 `backend_hostname` 에 명시되어 있는 PostgreSQL 인스턴스에 따라 구분한다. 해당 설정에서 첫 번째로 명시된 호스트를 마스터로 간주한다.

### Replication mode 와 WAL-based-replication은 무엇이 다른걸까?
Pgpool 이 제공하는 Replication mode 는 statement 기반으로 작동한다. 

즉, 클라이언트로 부터 받은 쿼리를 모든 DB 서버에 전달하는 것으로, 서버의 변경 사항을 전달하는  WAL-based-replication 과는 차이가 있다. 


### Watchdog
Pgpool 역시 SPOF(Signe Point of Failure)가 될 수 있다. 이를 위해 Pgppol 노드를 추가하여 HA를 구성해야 할 필요가 있는데, 이를 수행해 주는 것이 Watchdog이다. 

기본적으로, keepalived 와 유사하게 Pgpool 인스턴스 간 heartbeat check를 통해 VIP를 이전하는 active-stand by 구조이다.