## `@Resource`

Spring 에서 의존성 주입을 위해 사용되는 어노테이션.

Spring의 스펙은 아니고, Java의 스펙이다.

Spring 에는 의존성 주입을 위한 `@Autowired` 어노테이션도 있는데, 둘은 차이가 있다.

### `@Resource` vs `@Autowired`
`@Resource`는 이름으로 먼저 빈을 찾고, 없으면 타입으로 찾는다. 
반면, `@Autowired`는 타입으로 빈을 찾고, 필요한 경우 `@Qualifier` 어노테이션을 사용하여 빈의 이름을 지정할 수 있다.

```Java
@Repository
public class TestAutowired {
    @Resource(name="BlueSqlSessionTemplate")
    private SqlSessionTemplate sqlSession;
 }
```

```Java
@Repository
public class TestResource {
    @Autowired
    private SqlSessionTemplate sqlSession;
}
```
