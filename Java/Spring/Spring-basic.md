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
- **리퀘스트 (Request)**: HTTP 요청 하나당 하나의 인스턴스 생성
- **세션 (Session)**: HTTP 세션 하나당 하나의 인스턴스 생성
- **애플리케이션 (Application)**: 서블릿 컨텍스트당 하나의 인스턴스 생성

리퀘스트, 세션, 애플리케이션은 웹 전용 스코프입니다.

```java
// 스코프 설정 방법
@Scope("prototype")
```

## 의존관계 주입 방법

- 생성자 주입 (권장)
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
