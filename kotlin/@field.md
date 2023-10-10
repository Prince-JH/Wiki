## `@field`

다음과 같은 클래스에서 tempField가 null이 불가하도록 Bean Validation을 걸어주고 싶을 때, `@NotNull`과 `@field:NotNull`의 차이는 뭘까? 

```kotlin
class TempClass(
    @NotNull
    @field:NotNull
    var tempField:String
)
```

Java Spring을 사용했다면 `@NotNull`에 익숙할 것이다.

그런데 kotlin은 조금 다르다. kotlin에서는 클래스를 생성하면 JVM 바이트 코드 수준에서 여러 컴포넌트가 생성된다.

- 실제 필드
- 게터
- 세터

여기에 그냥 `@NotNull`만 선언하면 해당 어노테이션이 정확히 어디에 적용되어야 하는지 알수 없다. 따라서, 어떤 수준에 NotNull을 걸어 주는지 명시가 필요하다.