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
    var fontSize: Byte? = null,
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