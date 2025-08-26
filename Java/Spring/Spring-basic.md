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

일반적으로 `@Component`, `@Service`, `@Repository`, `@Controller`와 같은 어노테이션을 통해 클래스를 빈으로 등록합니다.

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
