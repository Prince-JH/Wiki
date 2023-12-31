# NULL

DB를 설계할 때, 유연한 값 삽입을 위해 `NULLABLE`을 종종 사용하곤 했다. `NULL`을 사용해도 괜찮을까?

## 의미가 분명하지 않다.
컬럼이 NULL 값을 가지고 있으면 의미가 혼란스러울 수 있다.

예를 들어, `Child` 테이블의  `school` 칼럼이  `NULL` 이라면 어떤 의미일까?

해당 학생이 졸업을 해서 `NULL` 인지, 학교를 안다녀서 `NULL` 인지, 학교를 다니긴 하는데 아직 정보 입력이 안되어서 `NULL`인지 불분명하다

관계형 데이터 모델에서는 명제를 참, 거짓으로만 판명하는데 이러한 경우는 판명이 불가하므로 모순이 발생한다.


## SQL이 복잡해진다.
`COUNT(*)` 집계 함수를 사용할 때는 `NULL` 이 있는 행도 포함하여 계산한다.

그러면 `AVG()` 는 어떨까? `AVG()` 함수는 평균을 계산할 때, `NULL` 값은 자동으로 무시한다. `NULL` 값을 포함하여 계산하려면, `COALESCE()` 함수를 사용하여 기본값을 줄 수 있지만, 그러면 값을 뭐로 줘야할까? 정말 0일까?

단순한 조회문을 만들 때도 고려해야한다.

`WHERE` 절에 조건을 걸어 조회를 할 때, `IS NULL` 또는 `IS NOT NULL` 조건 까지 포함하여 조회해야할 것이다.

## NULL이 있는 칼럼에 인덱스를 사용할 수 있을까?
인덱스의 동작은 RDBMS의 종류(MySQL, PostgreSQL, Oracle 등)에 따라 다르다.

대부분의 RDBMS에서는 `NULL` 값을 인덱스에 포함시키며, `NULL` 값을 가진 레코드를 빠르게 검색하고 정렬하는 데 도움이 된다.

그러나 Oracle의 경우 `NULL` 값을  인덱스에 포함시키지 않으며, 이를 모르고 사용하면 아무리 쿼리 계획을 보더라도 이해가 되지 않을 수 있다.

보통 JPA와 같은 ORM을 사용하면 DB와 통신하는 부분은 추상화를 통해 알아서 수행해주는데, 이러한 점이 독으로 작용하여 아무 생각 없이 사용하는 DB를 바꿨다가 성능이 이상해지는 경우가 발생할 수 있다.
