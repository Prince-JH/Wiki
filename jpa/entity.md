## Entity
*본 글은 kotlin 언어로 작성되었습니다.*

### default value

Entity 를 정의할 때, 테스트 시에 모든 nullable 칼럼에 일일이 값을 넣는 것이 귀찮아서
```kotlin 
var orientation: String? = null
```
과 같이 null을 집어넣어 버리는 경우가 있다. 

그런데 이 것이 DB 스키마 상으로 default 값이 설정되어 있다면 문제가 된다.

예를 들어, 다음과 같은 DB 스키마가 있다고 할 때
```sql
CREATE TABLE test
(
    seq                int unsigned                       not null primary key,
    orientation        char     default '1'               null comment 
    back_theme         char     default '1'               null comment 
    color_theme        char     default '3'               null comment 
    font_size          tinyint  default 0                 null comment
)
```
다음과 같이 Entity를 정의해 버리면
```kotlin
@Entity
@Table(name = "test")
class TestEntity(
    @Id
    @Column(name = "seq")
    var seq: Long? = null,

    @Column(name = "orientation")
    var orientation: String? = null,

    @Column(name = "back_theme")
    var backTheme: String? = null,

    @Column(name = "color_theme")
    var colorTheme: String? = null,

    @Column(name = "font_size")
    var fontSize: Byte? = null
)
```

`not null` 칼럼인 `seq` 만 정의하여 INSERT를 하려고 할 때 나머지 값들은 전부 `null`로 저장되게 된다.
```kotlin
val newTest = TestEntity(
    seq = 10
)  
```
이는 default 값들이 INSERT 시에 명시적으로 지정하지 않은 경우에만 적용되는데, JPA는 prepared statement를 사용하여 Entity에 정의한 값(null)을 바인딩하여 INSERT 하기 때문이다.
따라서, 스키마의 default 값을 Entity에도 설정해줘야 한다. ex) var orientation: String? = "1"

### `@DynamicInsert`, `@DynamicUpdate`
위의 문제를 Entity에 default 값을 주지 않고 해결할 수 있는 방법이다.

`@DynamicInsert`는 INSERT 구문 생성 시점에 null이 아닌 컬럼들만 포함하며, `@DynamicUpdate`는 UPDATE 구문 생성 시점에 null이 아닌 컬럼만(변경된 값만) 포함한다.

```kotlin
@DynamicInsert
@DynamicUpdate
@Entity
@Table(name = "test")
class TestEntity(
    @Id
    @Column(name = "seq")
    var seq: Long? = null,
    ...
)
```

다만 위와 같은 방식은, 어노테이션의 이름으로도 알 수 있듯이 쿼리를 **동적으로** 만들기 때문에 prepared statement의 이점인 **빠른 쿼리 실행 속도**는 더뎌질 수 밖에 없다.

또 한가지 주의해야 할 것은, 엔티티를 save하면 DB에는 default 값이 저장되지만 애플리케이션의 영속성 내에서는 여전히 null로 남아있다는 것이다.

따라서 경우에 따라, 생성 이후에 해당 엔티티의 영속 컨텍스트를 갱신해야 할 수 있다.