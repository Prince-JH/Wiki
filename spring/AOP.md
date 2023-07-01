# AOP(Aspect-Oriented Programming)

AOP는 애플리케이션의 핵심 기능과 각 부분에서 공통적으로 사용되는 관심사(cross-cutting-concerns)를 분리하는데 초점을 맞춘다. 이렇게 분리함으로써 코드의 재사용성을 높이고 유지보수를 쉽게 할 수 있다.

예를 들어, 로깅, 트랜잭션 관리, 보안 등과 같은 기능은 여러 모듈이나 클래스에서 공통적으로 필요로 하는데, 이러한 기능들을 모든 곳에 직접 삽입하는 것은 비효율적이고 유지보수가 어렵다. AOP를 사용하면 이러한 **공통 기능을 별도의 모듈로 분리**하고, **애플리케이션의 다른 부분에 선언적으로 적용**할 수 있다.

정리하면 AOP의 포인트는 다음과 같다.
- 애플리케이션 전체에 흩어진 공통 기능이 하나의 장소에서 관리되어 유지보수가 용이하다.
- 핵심 로직과 부가 기능의 명확한 분리로, 핵심 로직은 자신의 목적 외에 사항들에는 신경쓰지 않는다.

AOP의 적용 방식은 크게 3가지이다.
- 컴파일 시점
    - .java 파일을 컴파일러를 통해 .class를 만드는 시점에 부가 기능 로직을 추가하는 방식
    - 모든 지점에 적용 가능
    - AspectJ가 제공하는 특별한 컴파일러를 사용해야 하기 때문에 특별한 컴파일러가 필요한 점과 복잡하다는 단점이 있다.
- 클래스 로딩 시점
    - .class 파일을 JVM 내부의 클래스 로더에 보관하기 전에 조작하여 부가 기능 로직 추가하는 방식
    - 모든 지점에 적용 가능
    - 특별한 옵션과 클래스 로더 조작기를 지정해야하므로 운영하기 어려움
- 런타임 시점
    - **스프링이 주로 사용하는 방식**
    - 컴파일이 끝나고 클래스 로더에 올라가 자바가 실행된 다음에 동작하는 런타임 방식
    - 실제 대상 코드는 그대로 유지되고 프록시를 통해 부가 기능이 적용
    - **프록시는 메소드 오버라이딩 개념으로 동작하기 때문에 메소드에만 적용 가능 -> 스프링 빈에만 AOP를 적용 가능**
    
스프링 AOP는 AspectJ 방식이 아닌 프록시 방식의 AOP를 제공한다.

여기서 짚고 넘어가야할 것이 있는데,
- 프록시 기반 AOP:
프록시는 원본 객체를 프록시로 감싸고, 이 프록시가 추가적인 동작(예를 들어 트랜잭션 관리)을 수행한 후, 대상 메소드를 대신 호출한다. private 메소드는 외부에서 상속 또는 오버라이드할 수 없으므로, 감쌀 방법 또한 없다. 따라서 public 메소드에만 적용할 수 있다. 

- 프록시와 타겟 객체의 분리: 객체 내부에서 직접 호출하는 경우, 프록시를 거치지 않고메소드를 직접 호출하게 되기 때문에, `@Transactional`과 같은 AOP 관련 어노테이션은 무시되게 된다. 이는 private 메소드뿐만 아니라, 동일한 클래스 내에서 다른 메소드를 호출하는 경우에도 해당된다.

예를 들어, 다음과 같은 코드가 있다고 해보자.
```Java
@Service
public class MyService {

    public void publicMethod() {
        System.out.println("Inside public method");
        privateMethod();
    }

    private void privateMethod() {
        System.out.println("Inside private method");
    }
}

@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.example.MyService.*(..))")
    public void beforeAdvice() {
        System.out.println("Before advice executed");
    }
}
```
위의 예제에서 `publicMethod`를 호출하면 "Before advice executed" 가 출력된 이후 "Inside public method" 가 출력된다. 그러나 내부에서 privateMethod 를 직접 호출하기 때문에 Aspect가 적용되지 않고 "Inside public method"만 출력된다.

그렇다면 다음 예제는 어떨까?
```Java
@Service
public class MyService {

    @Transactional
    public void publicMethod() {
        System.out.println("Inside public method");
        privateMethod();
    }

    @Transactional
    private void privateMethod() {
        System.out.println("Inside private method");
    }
}
```

`@Transactional` 또한 AOP의 일종이므로 privateMethod의 `@Transactional`은 무시된다. 따라서 트랜잭션 전이가 되지 않는다. 다만, 이미 publicMethod에서 열어놓은 트랜잭션 컨텍스트 내에서 실행되고 있기 떄문에, 우리가 예상한 대로 동작할 것이다.

AOP 용어에 대해 살펴보자.

- Aspect: 공통적인 기능을 정의하는 모듈. 예를 들어, 로깅을 관리하는 코드가 aspect가 될 수 있다.

- Join Point: 애플리케이션 실행 중에 aspect가 적용될 수 있는 지점. 예를 들어, 메소드 호출이나 예외 처리 등

- Advice: aspect가 실제로 수행하는 작업. 예를 들어, 메서드 호출 전에 로그를 기록하는 것이 advice가 될 수 있다.

- Pointcut: join point들 중에서 aspect가 적용될 구체적인 지점. 예를 들어, 특정 패키지 내의 모든 메서드 호출을 포인트컷으로 정의할 수 있다.

- Weaving: aspect를 애플리케이션 코드에 적용하는 과정. 이는 컴파일 타임, 로드 타임, 또는 런타임에 수행될 수 있다.

- AOP 프록시: AOP 기능을 구현하기 위해 만든 프록시 객체. 스프링에서 AOP 프록시는 JDK dynamic 프록시 또는 CGLIB 프록시이다.
    - JDK dynamic 프록시: 대상 객체가 인터페이스를 구현하는 경우 사용된다. 여기서 프록시는 대상 객체와 동일한 인터페이스를 구현한다. 이 프록시는 대상 객체에 대한 참조를 보유하고 있으며, 모든 메소드 호출을 해당 객체에게 위임한다.
    - CGLIB 프록시: 대상 객체가 인터페이스를 구현하지 않은 경우 사용된다. CGLIB은 클래스를 상속하여 새로운 '서브 클래스'를 만든다. 이 서브 클래스는 프록시로 작동하며, 메소드 오버라이딩을 통해 부모 클래스의 메소드를 호출한다.

