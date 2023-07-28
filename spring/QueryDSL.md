# QueryDSL

## 왜 쓰는걸까
- 코드로 쿼리 작성하여 컴파일 시점 문법 오류 확인
- 동적 쿼리 작성이 편리
- 제약 조건을 메소드로 추출하여 재사용

## 동적 쿼리 생성 시 
- `BooleanBuilder`: 조건의 개수가 많아지면 어떤 쿼리인지 예상하기 어렵다.

- `BooleanExpression`: 쿼리의 생김새는 동일한데 조건에 따라 해당 조건이 삭제됨

`BooleanExpression`을 사용하자

## `exist()` 금지
SQL에서는 `exist()`가 `count(1)`보다 훨씬 빠르다. 
- `exist`:  데이터 발견 시 탐색 종료
- `count(1)` > 0: 스캔이 끝난 뒤에 조건 검사

그러나 QueryDsl에서의 `exist()`는 `count()` 쿼리를 사용한다. 

-> limit 1로 조회 제한하여 직접 구현: `fetch first()`사용하여 첫 번쨰 row를 조회, 단 조회결과가 없으면 `null`임을 유의

## Cross Join
cross join: 테이블의 모든 조합을 가져옴

QueryDSL 에서 묵시적 join 사용 시 `cross join`이 날아갈 수 있음, 이는 Hibernate 이슈라서 Spring Data JPA도 동일하게 발생 -> 명시적 join을 사용 하면 원하는 join이 날아가도록 해결 가능

## Entity 보다 DTO를 우선
- Entity 조회 시 불필요한 컬럼 조회, OneToOne N+1 쿼리 등 문제가 생길 수 있다.
    - OneToOne  은 일반적으로 Lazy Loading이 안됨
    - 연관된 Entity의 save를 위해서는 반대편 Entity의 ID만 있으면 된다.
- 사용 목적에 따라
    - 실시간으로 Entity 변경이 필요한 경우: Entity 조회
    - 고강도 성능 개선 or 대량의 데이터 조회가 필요한 경우: DTO 조회

## Gorup By 최적화
- MySQL에서 Group By를 실행하면 Index가 아닌 경우 Filesort 가 필수로 발생
    - `order by null` 을 사용하면 Filesort가 제거됨
    - QueryDSL에서는 `order by null` 문법을 지원하지 않음 -> 직접 작성이 필요

정렬일 필요할 때, 조회 결과가 100건이하라면 애플리케이션에서 정렬하는 편이 낫다. WAS가 DB보다 저렴하기 떄문에

## 커버링 인덱스
- 쿼리를 충족시키는데 필요한 모든 컬럼을 갖고 있는 인덱스
- JPQL은 `from`절의 서브쿼리를 지원하지 않는다. -> PK를 커버링 인덱스로 빠르게 조회하고, 조회된 Key들로 `SELECT` 컬럼들을 IN절을 통해 후속 조회

## Update 최적화
DirtyChecking 방식을 무분별하게 사용하면, 일괄 Update가 필요한 경우 해당 엔티티를 전부 조회하게 된다. Queydsl.update 사용으로 해결할 수 있다.

-> django의 for문 `save()`와, `update()`의 차이와 유사

-> 일괄 업데이트 시 하이버네이트 캐시 갱신이 안됨(django와 동일)

-> cache eviction이 필요

## BulkInsert
auto_increment일 때 bulk insert가 적용되지 않는다. -> `JdbcTemplate` 을 사용하는 것이 좋으나, 컴파일 체크, 코드-테이블간의 불일치 체크가 까다로움 
TypeSafe한 방식인, QueryDSL - natvie SQL 을 사용할 수 있으나, 이 또한 완전한 대안은 아님.

## 정리
1. 상황에 따라 **ORM / 전통적 Query** 방식을 골라 사용
2. JPA / QueryDSL로 발생하는 **쿼리 확인하기**