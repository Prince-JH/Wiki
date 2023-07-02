# Spring Container

Spring 에서는 Spring 컨테이너(IoC 컨테이너)라는 개념을 사용한다. 컨테이너는 인스턴스의 생명주기를 관리하며, 개발자가 작성한 코드의 처리과정을 위임하여 실행한다.

Spring 컨테이너가 코드를 실행하는 방법은 다음과 같다.

1. 컨테이너 초기화: 애플리케이션 시작 시, Spring 컨테이너는 구성 파일(XML, Java Config) 또는 어노테이션을 사용하여 초기화됩니다. 이 과정에서 컨테이너는 어떤 객체들을 생성하고, 어떻게 조립할지에 대한 정보를 읽어들인다.

2. 빈 생성: 컨테이너는 이 정보를 바탕으로 빈 객체들을 생성한다. 이 때, 객체들은 싱글턴, 프로토타입 등 다양한 범위(Scopes)로 생성될 수 있다.

3. 의존성 주입: 생성된 빈들이 서로 의존하는 경우, 컨테이너는 이 의존성을 해결하기 위해 필요한 객체를 주입(Dependency Injection)한다. 의존성 주입은 주로 생성자 또는 세터 메서드를 통해 이루어진다.

4. 빈 사용: 애플리케이션 코드는 컨테이너로부터 빈을 요청하여 사용한다. 이 때, 애플리케이션은 직접 객체를 생성하지 않고, 컨테이너로부터 주입받거나 요청하여 사용한다. 이를 통해 코드 간 결합도(Coupling)가 낮아져 유지보수가 용이해진다.

5. 컨테이너 종료: 애플리케이션이 종료될 때, 컨테이너는 빈들의 생명주기를 관리하는 책임을 가지고 있다. 컨테이너는 빈들의 종료 메서드를 호출하여 리소스를 정리한다.

각각의 스텝에 대해 좀 더 자세히 알아보자.

## 컨테이너 초기화, 빈 생성

Spring initializer 에서 프로젝트를 받아 시작하면, 기본적으로 Main Method가 아래와 같이 생겼다.
```Java
@SpringBootApplication
public class MyApplication {


    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

여기 붙어 있는 `@SpringBootApplication`은 여러 어노테이션을 포함하고 있는데, 설정과 관련된 어노테이션은 다음과 같다.
- `@EnableAutoConfiguration`: Spring Boot가 클래스 경로 설정, 다른 빈, 다양한 프로퍼티 설정에 따라 빈을 자동으로 설정한다.
- `@ComponentScan`: 현재 패키지와 하위 패키지를 스캔하여 Component, Service, Repository 등과 같은 Spring 어노테이션들을 자동으로 인식한다.

위와 같은 어노테이션 기능으로 인해, 보통 Spring Boot 애플리케이션에서 `@Configuration` 을 따로 사용할 필요가 없다. 하지만 특별한 구성이 필요하다면, 아래와 같이 `@Configuration` 을 통해 직접 설정이 가능하다.


```Java
@Configuration
public class AppConfig {
    @Bean
    public MessageService messageService() {
        return new MessageServiceImpl();
    }
}
```
이 처럼 프로젝트 구성(Configuration)을 정의하면, Spring 컨테이너는 해당 정보를 바탕으로 초기화된다.

이 때, `@Bean` 어노테이션이 붙은 메서드들을 실행하여 빈 객체를 생성한다. `@Component` 어노테이션이 등록되어 있는 경우에는 Spring이 어노테이션을 확인하고 자체적으로 빈으로 등록한다.


빈은 크게 두 가지 방식으로 생성이 가능하다.

1. 싱글톤 (Singleton): 기본적으로 Spring은 빈을 싱글톤으로 생성한다. 싱글톤이라 함은, 빈이 한 번만 생성되고 이후로는 동일한 인스턴스를 재사용한다는 의미이다. 이렇게 함으로써, 빈이 메모리를 효율적으로 사용할 수 있다.

2. 프로토타입 (Prototype): 빈이 프로토타입 범위로 설정된 경우, Spring 컨테이너는 빈이 주입될 때마다 새 인스턴스를 생성한다.

## 의존성 주입

아래는 생성자를 통해 의존성을 주입한 예시이다. set 메소드를 통해서도 주입이 가능하다. 필드 변수로도 주입이 가능하나, 이는 의존성을 외부에서 주입하는 것이 아니라 클래스 내부에서 선언해 버리는 것이기 때문에 안티패턴으로 작용한다.
```Java
@Component
public class MessagePrinter {
    private final MessageService messageService;

    @Autowired
    public MessagePrinter(MessageService messageService) {
        this.messageService = messageService;
    }

    public void printMessage() {
        System.out.println(this.messageService.getMessage());
    }
}
```

## 빈 사용
아래는 생성된 빈 객체를 사용하는 예시이다. MyController 는 빈으로 등록되고, MessagePrinter 빈이 의존성으로 주입된다.

```Java
@RestController
public class MyController {
    private final MessagePrinter messagePrinter;

    @Autowired
    public MessageController(MessagePrinter messagePrinter) {
        this.messagePrinter = messagePrinter;
    }

    @GetMapping("/messages")
    public String print() {
        return messagePrinter.printMessage();
    }
}
```

Spring 컨테이너는 빈들을 관리하고, 필요한 곳에 자동으로 의존성을 주입하여 개발자가 직접 객체를 생성하고 관리하는 부담을 줄여준다. 

이러한 Spring의 IoC 패턴으로 인해 프레임워크가 애플리케이션의 흐름을 제어하고, 개발자는 필요한 부분만 구현하면 되기 때문에 비즈니스 로직에 집중할 수 있다.