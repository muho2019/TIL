# Spring Basic

## 스프링 컨테이너 Spring Container

스프링 프레임워크의 핵심 기능으로 객체의 생성부터 소멸까지 전체 생명주기를 관리하는 역할을 합니다.

- 빈(Bean) 객체의 생성, 초기화, 소멸 관리
- 의존성 주입(Dependency Injection) 수행
- 빈들 간의 관계 설정 및 관리
- 설정 정보를 바탕으로 애플리케이션 구성

스프링 컨테이너는 `ApplicationContext`라는 인터페이스로 구현됩니다.

설정하는 방식에는 두 가지가 있습니다.

- XML
- 어노테이션

일반적으로는 어노테이션 방식을 많이 이용합니다.

## 스프링 빈 Spring Bean

스프링 빈은 스프링 컨테이너가 생성, 관리, 소멸시키는 객체를 의미합니다.

- 스프링 컨테이너에 의해 생명주기가 관리됨
- 기본적으로 싱글톤 패턴으로 생성 (하나의 인스턴스만 존재)
- 의존성 주입을 통해 다른 빈들과 연결됨

일반적으로 스테레오타입 어노테이션(`@Component`, `@Service`, `@Repository`, `@Controller`)을 통해 클래스를 빈으로 등록합니다.

### 빈 스코프 Bean Scope

스프링 빈이 존재할 수 있는 범위를 정의합니다.

- **싱글톤 (Singleton)**: 컨테이너당 하나의 인스턴스만 생성
- **프로토타입 (Prototype)**: 요청할 때마다 새로운 인스턴스 생성
  - 프로토타입의 빈은 스프링 컨테이너가 초기화까지만 책임진다. 따라서 종료 로직은 실행되지 않는다.
- **리퀘스트 (Request)**: HTTP 요청 하나당 하나의 인스턴스 생성
- **세션 (Session)**: HTTP 세션 하나당 하나의 인스턴스 생성
- **애플리케이션 (Application)**: 서블릿 컨텍스트당 하나의 인스턴스 생성

리퀘스트, 세션, 애플리케이션은 웹 전용 스코프입니다.

```java
// 스코프 설정 방법
@Scope("prototype")
```

## 의존성 주입 방법

- 생성자 주입 (권장)
  - 의존성 주입된 객체는 불변해야 하므로 생성자 주입을 권장
- Setter 주입
- 필드 주입

생성자 주입 예시

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

`lombok` 라이브러리를 이용하면 조금 더 편하게 생성자 주입을 이용할 수 있습니다. 애노테이션 `@RequiredArgsConstructor`을 이용하며, `final` 필드에 대해 생성자를 자동으로 생성합니다.

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    // Lombok이 자동으로 생성자를 생성
    // public UserService(UserRepository userRepository) {
    //     this.userRepository = userRepository;
    // }
}
```

### 다형성으로 구현된 빈의 의존성 주입 방법

**조회 대상 빈이 2개 이상일 때 해결 방법:**

- 필드/파라미터명 매칭
- `@Qualifier`
- `@Primary`
- 컬렉션 이용

#### 필드/파라미터명 매칭

구체 클래스의 이름으로 필드/파라미터명으로 설정하면 해당 타입의 객체가 주입됩니다.

**장점:**

- 코드가 간결해집니다. `@Qualifier`와 같은 어노테이션이 필요하지 않기 때문입니다.

**단점:**

- 주입받는 변수명이 구체 클래스에 의존하기 때문에 코드가 유연하지 못하고 결합도가 높아집니다. 추후에 주입받을 객체가 다른 타입의 구체 클래스로 변경되면 클라이언트 코드도 변경해야 하므로 OCP(개방-폐쇄 원칙)에 위배될 수 있습니다.

```java
// SmtpEmailSender.java
@Service
public class SmtpEmailSender implements EmailSender {
    // ...
}

// MockEmailSender.java
@Service
public class MockEmailSender implements EmailSender {
    // ...
}

// 서비스 클래스에서 주입
@Service
public class UserService {
    // 필드명(파라미터명)을 구체 클래스 이름의 첫 글자를 소문자로 바꾼 형태로 설정
    private final EmailSender smtpEmailSender;

    // @Qualifier 없이도 SmtpEmailSender 빈을 주입받음
    @Autowired
    public UserService(EmailSender smtpEmailSender) {
        this.smtpEmailSender = smtpEmailSender;
    }
}
```

```java
@Autowired
private DiscountPolicy rateDiscountPolicy // RateDiscountPolicy가 주입됨
```

#### `@Qualifier`

`@Qualifier`는 '자격' 또는 '한정자'라는 의미로, 주입할 빈의 이름을 명시적으로 지정하는 데 사용됩니다. 가장 일반적으로 사용하는 방법입니다.

**장점:**

- 명확한 의도
  - 주입하려는 빈의 이름이 명시적으로 드러나기 때문에 코드의 가독성을 높이고 어떤 빈이 주입되는지 쉽게 파악됩니다.
- 유연성
  - 주입되는 객체가 다른 빈으로 변경되면 `@Qualifier` 이름만 변경하면 되므로 코드 수정이 유연합니다.

**단점:**

- 결합도
  - 필드/파라미터명으로 매칭하는 경우와 마찬가지로 특정 빈의 이름에 의존하기 때문에 결합도가 높아집니다.

```java
// SmtpEmailSender.java
@Service("smtpEmailSender") // 빈 이름 지정
public class SmtpEmailSender implements EmailSender {
    // ...
}

// MockEmailSender.java
@Service("mockEmailSender") // 빈 이름 지정
public class MockEmailSender implements EmailSender {
    // ...
}

// 서비스 클래스에서 주입
@Service
public class UserService {
    private final EmailSender emailSender;

    @Autowired
    public UserService(@Qualifier("smtpEmailSender") EmailSender emailSender) {
        this.emailSender = emailSender;
    }
}
```

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {}

// 주입
@Autowired
public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```

#### `@Primary`

`@Primary`는 여러 빈 중 우선적으로 주입될 빈을 지정합니다. `@Qualifier` 없이 주입될 기본 빈을 선택하는 방법입니다.

**장점:**

- 간결성
- 기본값 설정

**단점:**

- 모호성
  - `@Primary`가 설정된 빈을 찾아보지 않는 이상 어떤 빈이 주입되는지 알 수 없습니다. 이는 가독성을 해칠 수 있습니다.
- 제한적인 선택
  - 다른 빈을 선택해야 하는 경우에는 결국 `@Qualifier`를 사용해야 합니다.
- 잠재적 충돌
  - `@Primary` 빈이 없거나 여러 개일 경우 스프링이 어떤 빈을 선택할지 결정하지 못해 오류가 발생할 수 있습니다.

```java
// SmtpEmailSender.java
@Service
public class SmtpEmailSender implements EmailSender {
    // ...
}

// MockEmailSender.java
@Service
@Primary // 기본 빈으로 설정
public class MockEmailSender implements EmailSender {
    // ...
}

// 서비스 클래스에서 주입 (별도의 @Qualifier 없이 @Primary 빈이 주입됨)
@Service
public class UserService {
    private final EmailSender emailSender;

    // 스프링이 @Primary가 붙은 MockEmailSender를 찾아 주입
    @Autowired
    public UserService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }
}
```

#### 컬렉션 이용

동일한 인터페이스를 구현한 모든 빈을 주입받아야 할 때 리스트를 사용합니다.

```java
// MessageHandler.java (인터페이스)
public interface MessageHandler {
    void handle(String message);
}

// SmsHandler.java
@Service("smsHandler")
public class SmsHandler implements MessageHandler {
    // ...
}

// EmailHandler.java
@Service("emailHandler")
public class EmailHandler implements MessageHandler {
    // ...
}

// 모든 MessageHandler를 주입받아 사용
@Service
public class MessageService {
    private final List<MessageHandler> handlers;

    @Autowired
    public MessageService(List<MessageHandler> handlers) {
        this.handlers = handlers; // [SmsHandler, EmailHandler]가 주입됨
    }

    public void processMessage(String message) {
        for (MessageHandler handler : handlers) {
            handler.handle(message);
        }
    }
}
```

### 의존성 주입 생명주기 불일치 문제

만약 싱글톤 빈에 리퀘스트 스코프 빈을 주입하려고 하면 문제가 발생합니다. 이 문제를 해결하기 위해서는 **Provider** 또는 **Scoped Proxy**를 이용합니다.

#### Provider (권장)

싱글톤 빈은 `Provider`를 가지고 있다가, 실제로 리퀘스트 스코프 빈이 필요할 때 `provider.get()`을 통해 빈을 찾아와 사용합니다. 코드가 명시적이라 의존성 탐색(DL, Dependency Lookup)이 언제 일어나는지 명확하게 알 수 있습니다.

```java
// 리퀘스트 스코프 빈
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;
import java.util.UUID;

@Component
@Scope("request") // 이 빈은 HTTP 요청마다 새로 생성됩니다.
public class MyRequestBean {

    private final String uuid = UUID.randomUUID().toString();

    public String getUuid() {
        return uuid;
    }
}

// 싱글톤 서비스
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import jakarta.inject.Provider; // jakarta.inject.Provider 또는 javax.inject.Provider

@Service
public class SingletonService {

    // MyRequestBean을 직접 주입받는 대신, Provider를 주입받습니다.
    private final Provider<MyRequestBean> requestBeanProvider;

    @Autowired
    public SingletonService(Provider<MyRequestBean> requestBeanProvider) {
        this.requestBeanProvider = requestBeanProvider;
    }

    public void logRequestUuid() {
        // 이 메서드가 호출될 때! 실제 Request Bean을 찾아서 가져옵니다. (DL)
        MyRequestBean requestBean = requestBeanProvider.get();
        System.out.println("Current Request UUID: " + requestBean.getUuid());
    }
}
```

#### Scoped Proxy

이 방법은 리퀘스트 스코프 빈에 프록시 설정을 추가하여, 마치 일반 빈처럼 주입해서 사용할 수 있습니다. 싱글톤 빈에 주입될 때는 프록시 객체가 주입되고 실제 빈을 사용할 때 프록시가 실제 빈을 찾아 메서드 호출을 위임합니다.

```java
// 리퀘스트 스코프 빈
import org.springframework.context.annotation.Scope;
import org.springframework.context.annotation.ScopedProxyMode;
import org.springframework.stereotype.Component;
import java.util.UUID;

@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS) // 프록시 모드 설정
public class MyRequestBean {
    // 내용은 동일
    private final String uuid = UUID.randomUUID().toString();
    public String getUuid() { return uuid; }
}

// 싱글톤 서비스
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class SingletonService {

    // 주입된 것은 진짜 MyRequestBean이 아닌, 가짜 프록시 객체입니다.
    private final MyRequestBean requestBean;

    @Autowired
    public SingletonService(MyRequestBean requestBean) {
        this.requestBean = requestBean;
        System.out.println("Proxy class: " + requestBean.getClass()); // 실제 클래스가 아닌 CGLIB 프록시 클래스가 출력됨
    }

    public void logRequestUuid() {
        // 프록시 객체의 메서드를 호출하면, 진짜 객체를 찾아 위임합니다.
        System.out.println("Current Request UUID: " + requestBean.getUuid());
    }
}
```

## @Configuration

`@Configuration`은 **스프링 설정 클래스**를 나타내는 어노테이션입니다. 일반적으로 스테레오타입 어노테이션을 이용한 클래스(컨트롤러, 서비스, 레포지토리)를 제외한 클래스 또는 복잡한 생성 로직이 필요한 클래스들의 빈 등록 시 사용합니다.

**특징**

- 빈 등록, 관리
  - `@Bean` 어노테이션을 사용하여 빈을 정의합니다.
- 싱글톤 보장
- 의존성 주입
  - `@Bean` 메서드의 파라미터를 통해 이루어지는 것이 가장 일반적이고 권장되는 방법입니다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

// 이 클래스가 스프링 설정 클래스임을 명시
@Configuration
public class AppConfig {

    // "myService"라는 이름의 빈을 정의
    @Bean
    public MyService myService() {
        return new MyService(); // MyService 객체를 스프링 빈으로 등록
    }

    // "myRepository"라는 이름의 빈을 정의
    @Bean
    public MyRepository myRepository() {
        return new MyRepository();
    }
}
```

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

// 데이터베이스 연결을 담당하는 클래스라고 가정
class DataSource {
    public void connect() {
        System.out.println("데이터베이스에 연결되었습니다.");
    }
}

// 데이터를 저장하는 리포지토리 클래스라고 가정
class UserRepository {
    private final DataSource dataSource;

    // 생성자 주입을 통해 DataSource 의존성 주입
    public UserRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}

@Configuration
public class AppConfig {

    // 1. DataSource 빈 정의
    @Bean
    public DataSource dataSource() {
        return new DataSource();
    }

    // 2. UserRepository 빈 정의
    //    이 메서드는 파라미터로 DataSource를 받습니다.
    //    스프링 컨테이너가 AppConfig.dataSource() 메서드를 호출하여
    //    생성된 DataSource 빈을 자동으로 이 파라미터에 주입해 줍니다.
    @Bean
    public UserRepository userRepository(DataSource dataSource) {
        // 주입받은 DataSource 빈을 사용하여 UserRepository 객체를 생성
        return new UserRepository(dataSource);
    }
}
```

### @Value

애플리케이션 설정 파일(`application.properties` 등)에 정의된 값을 `@Bean` 메서드에 주입하여 구성할 수 있습니다.

```java
@Configuration
public class S3Config {

    @Value("${cloud.aws.s3.bucket-name}")
    private String bucketName;

    @Value("${cloud.aws.credentials.access-key}")
    private String accessKey;

    @Bean
    public AmazonS3 s3Client() {
        // @Value로 주입된 값을 사용하여 빈 생성
        return AmazonS3ClientBuilder.standard()
                                     .withRegion(Regions.AP_NORTHEAST_2)
                                     .withCredentials(new AWSStaticCredentialsProvider(new BasicAWSCredentials(accessKey, "secret-key")))
                                     .build();
    }
}
```

### @Profile

개발, 테스트, 운영 등 환경에 따라 다른 빈을 구성해야 할 때 `@Profile`을 사용할 수 있습니다.

```java
// 개발 환경에서만 활성화되는 빈
@Configuration
@Profile("dev")
public class DevConfig {
    @Bean
    public DatabaseConnector h2DatabaseConnector() {
        return new H2DatabaseConnector();
    }
}

// 운영 환경에서만 활성화되는 빈
@Configuration
@Profile("prod")
public class ProdConfig {
    @Bean
    public DatabaseConnector mysqlDatabaseConnector() {
        return new MySQLDatabaseConnector();
    }
}
```

### @Import

하나의 `@Configuration` 클래스로 구성을 하다보면 너무 커지고 복잡해지게 됩니다. 이때 **관심사의 분리**를 위하여 분야별로 `@Configuration` 클래스를 따로 구성해주는 것이 좋습니다. 이럴 때 `@Import` 어노테이션으로 여러 `@Configuration` 클래스를 결합할 수 있습니다.

물론 결합해주지 않아도 컴포넌트 스캔을 통해 아무런 문제없이 구성이 등록될 수도 있습니다. 하지만 다음과 같은 경우에는 `@Import`를 사용하여 결합해주는 것이 좋습니다.

- `@Configuration` 클래스가 컴포넌트 스캔 범위를 벗어난 경우 (메인 패키지 밖에 있는 라이브러리나 모듈의 구성 클래스를 가져와야 할 때)
- 명시적인 의존 관계를 표현할 때 (특정 `@Configuration` 클래스가 다른 설정에 의존하고 있다는 것을 명시적으로 표현)

```java
// Database 관련 설정
@Configuration
public class DatabaseConfig {
    @Bean
    public DataSource dataSource() {
        // ...
    }
}

// Security 관련 설정
@Configuration
public class SecurityConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        // ...
    }
}

// 메인 설정 클래스에서 분리된 설정들을 결합
@Configuration
@Import({DatabaseConfig.class, SecurityConfig.class})
public class AppConfig {
    // 애플리케이션의 핵심 로직 관련 빈들
}
```

## 컴포넌트 스캔 Component Scan

컴포넌트 스캔은 스프링 프레임워크가 자동으로 빈 등록을 해주는 기능입니다.

스테레오타입 어노테이션을 등록한 클래스는 컴포넌트 스캔의 대상이 됩니다. 또한 `@Configuration` 역시 컴포넌트 스캔의 대상입니다.

컴포넌트 스캔은 `@SpringBootApplication` 어노테이션이 등록된 클래스의 경로부터 시작하여 그 하위의 모든 패키지를 스캔합니다.

`@ComponentScan` 어노테이션을 이용하여 필터링을 지정할 수 있습니다. 하지만 기본기능 그대로 사용하는 것이 가장 안전합니다.

**필터 종류:**

- `includeFilters`: 컴포넌트 스캔 대상을 추가로 지정
- `excludeFilters`: 컴포넌트 스캔에서 제외할 대상으로 지정

## 빈의 생명주기 관리

빈의 초기화 로직이 복잡한 경우 생성자에서 분리해주는 것이 좋습니다. 그리고 데이터베이스 커넥션 해제와 같이 빈의 소멸 로직이 필요한 경우도 있습니다.

이럴 때에는 `@PostConstruct`, `@PreDestroy` 어노테이션을 사용합니다. 이 어노테이션은 스프링에 종속적이지 않기 때문에 자유롭게 사용이 가능합니다.

- `@PostConstruct`: 의존성 주입이 완료된 후 호출되어야 하는 초기화 로직을 구현할 때 사용합니다.
- `@PreDestroy`: 빈이 소멸되기 직전에 호출되어야 하는 종료 로직(cleanup)을 구현할 때 사용합니다.

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class NetworkClient {

    private String url;

    // 1. 빈 생성 (생성자 호출)
    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
    }

    // 외부에서 의존성 주입 (Setter)
    public void setUrl(String url) {
        this.url = url;
    }

    // 2. 의존관계 주입 완료 후 초기화 로직 수행
    @PostConstruct
    public void connect() {
        System.out.println("초기화 로직... @PostConstruct 호출, url = " + url);
        // 이 시점에는 url과 같은 의존성이 모두 주입되었음이 보장됩니다.
        System.out.println("서버에 연결합니다: " + url);
    }

    // 3. 빈 소멸 전 종료 로직 수행
    @PreDestroy
    public void disconnect() {
        System.out.println("종료 로직... @PreDestroy 호출");
        System.out.println("서버 연결을 해제합니다: " + url);
    }
}
```

**빈 생명주기 콜백 순서:**

- 스프링 컨테이너 생성 및 빈 설정 정보 로드
- 빈 생성과 동시에 의존관계 주입 (생성자 호출 시점에 필요한 의존관계가 모두 해결되어 주입됩니다.)
- 초기화 콜백 (`@PostConstruct` 등)
  - 생성자에서 모든 의존성 주입이 보장되므로, 이 시점에서 주입된 의존성을 활용한 초기화 로직을 수행할 수 있습니다.
- 빈 사용
- 소멸 전 콜백 (`@PreDestroy` 등)
- 스프링 컨테이너 종료

### 그 외 방법

`@PostConstruct`, `@PreDestroy` 어노테이션을 사용하는 방법은 외부 라이브러리의 클래스처럼 코드를 직접 수정할 수 없는 클래스에는 적용할 수 없습니다.

이때에는 `@Bean`의 `initMethod`, `destroyMethod` 속성을 사용합니다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean(initMethod = "connect", destroyMethod = "disconnect")
    public NetworkClient networkClient() {
        NetworkClient client = new NetworkClient();
        client.setUrl("http://example.com");
        return client;
    }
}
```
