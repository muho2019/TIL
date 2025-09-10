# 서비스

서비스 클래스는 백엔드 애플리케이션의 **두뇌**와 같습니다. 어떻게 작성하느냐에 따라 시스템 전체의 유연성과 유지보수성이 결정됩니다.

가장 쉽게 비유하자면, 서비스 클래스는 레스토랑의 '헤드 셰프'와 같습니다.

- **Controller (웨이터)**: 손님(클라이언트)에게서 주문서(DTO)를 받습니다. 주문 내용이 유효한지 간단히 확인하고, 주방에 전달하는 역할만 합니다.
- **Service (헤드 셰프)**: 주문서를 보고, 레시피(**비즈니스 로직**)에 따라 요리를 시작합니다. 재료 손질 담당(**Repository**)에게 재료를 가져오게 시키고, 다른 셰프(**다른 Service**)와 협력하여 요리를 완성합니다. 모든 요리 과정이 순서대로, 문제없이 진행되도록 총괄 지휘(트랜잭션 관리)합니다.
- **Repository (재료 담당)**: 냉장고(DB)에서 특정 재료를 가져오거나(`find`), 새로운 재료를 보관(`save`)하는 역할만 전문적으로 수행합니다.

## 서비스 클래스의 핵심 역할

- **비즈니스 로직 처리**: 애플리케이션의 핵심 규칙과 계산을 수행합니다. "회원 등급이 VIP이면 포인트를 10% 더 적립한다"와 같은 로직이 여기에 해당합니다.
- **트랜잭션 관리**: 여러 데이터 변경 작업을 하나의 덩어리(원자적 연산)로 묶어 데이터의 일관성을 보장합니다. 주문 처리 중 하나라도 실패하면 모든 작업을 취소(롤백)하는 것이 대표적인 예입니다.
- **관심사의 분리 (Facade 역할)**: Controller가 DB나 외부 API에 대해 전혀 몰라도 되도록, 복잡한 과정을 서비스 클래스가 감싸서 단순한 인터페이스를 제공합니다. 웨이터가 주방의 복잡한 요리 과정을 몰라도 되는 것과 같죠.

## 좋은 서비스 클래스를 작성하는 5가지 원칙

### 1. 의존성은 생성자 주입으로 받으세요

필드에 직접 `@Autowired`를 붙이는 것보다 생성자를 통해 주입받는 것이 좋습니다.

- **불변성(Immutability)**: final 키워드를 사용할 수 있어, 서비스 객체가 생성된 후에 의존성이 바뀔 일이 없음을 보장합니다.
- **의존성 누락 방지**: 객체 생성 시점에 의존성이 모두 주입되었는지 컴파일러가 체크해 줍니다.
- **테스트 용이성**: 순수 자바 코드로 테스트할 때, 스프링 컨테이너 없이도 의존성을 쉽게 주입하며 객체를 생성할 수 있습니다.

### 2. DTO(Data Transfer Object)를 적극적으로 활용하세요

Controller와 Service 사이, Service와 외부 시스템 사이에서는 절대 DB와 직접 매핑된 Entity 객체를 주고받지 마세요. 대신, 각 계층에 필요한 데이터만 담은 DTO를 사용해야 합니다.

- **관심사 분리**: 화면(View)에 필요한 데이터와 DB에 저장되는 데이터는 다를 수 있습니다. DTO를 통해 각 계층이 필요한 데이터에만 집중할 수 있습니다.
- **안정성**: Entity를 직접 노출하면, Entity의 필드 하나가 바뀌었을 때 API 전체가 영향을 받을 수 있습니다. DTO는 API의 명세(스펙)를 고정하는 역할을 합니다.

### 3. `@Transactional`을 적극적으로 사용하되, 범위를 명확히 하세요

데이터를 변경하는(CUD) 모든 public 메소드에는 `@Transactional`을 붙여 트랜잭션을 적용하는 것이 좋습니다.

- **`@Transactional`**: 메소드가 성공적으로 끝나면 변경 사항을 DB에 커밋(Commit)하고, 중간에 예외가 발생하면 모든 작업을 롤백(Rollback)합니다.
- **`@Transactional(readOnly = true)`**: 데이터를 조회(Read)만 하는 메소드에 사용하면, 불필요한 변경 감지(Dirty Checking)를 생략하는 등 JPA 성능 최적화에 도움이 됩니다.

### 4. 하나의 메소드는 하나의 책임만 갖도록 하세요

서비스 메소드는 **하나의 비즈니스 유스케이스**를 표현해야 합니다. 메소드 이름만 봐도 무엇을 하는지 명확히 알 수 있어야 합니다. 만약 한 메소드가 너무 많은 일을 한다면, 의미 있는 단위의 private 헬퍼 메소드로 분리하는 것이 좋습니다.

```java
@Transactional
public Long createOrder(OrderRequestDto orderDto) {
    Member member = findMemberById(orderDto.getMemberId());
    Product product = findProductById(orderDto.getProductId());

    validateOrder(member, product, orderDto); // private 헬퍼 메소드로 로직 분리

    Order order = Order.create(member, product, orderDto.getQuantity());
    orderRepository.save(order);

    return order.getId();
}

private void validateOrder(Member member, Product product, OrderRequestDto orderDto) {
    // ... 주문 관련 검증 로직 ...
}
```

### 5. 명확한 예외(Exception)를 던지세요

"회원을 찾을 수 없음", "재고 부족" 등 비즈니스 상황에서 발생할 수 있는 예외는 `NullPointerException` 대신 `MemberNotFoundException`, `OutOfStockException처럼` 의미 있는 커스텀 예외를 만들어 던져야 합니다. 이렇게 하면 `@RestControllerAdvice`를 이용해 중앙에서 예외를 처리하고, 클라이언트에게 일관된 에러 응답을 보내기 용이해집니다.

## `@Transactional`

`@Transactional`은 스프링의 정수와도 같은 기능이자, 안정적인 백엔드 서버를 만드는 핵심 기술입니다. 단순히 '데이터 작업을 묶어주는 것' 이상으로 그 원리를 이해하면 훨씬 강력하게 사용할 수 있습니다.

### 왜 필요한가? - 원자성(Atomicity) 보장

`@Transactional`의 가장 근본적인 존재 이유는 **여러 데이터베이스 작업을 하나의 덩어리처럼 취급하여 '전부 성공'하거나 '전부 실패'하게 만들기 위함**입니다.

가장 고전적인 예시는 계좌 이체입니다.

1. A의 계좌에서 10,000원을 차감한다. (`UPDATE ...`)
2. B의 계좌에 10,000원을 추가한다. (`UPDATE ...`)

만약 1번 작업만 성공하고, 시스템이 다운되어 2번 작업이 실패한다면 어떻게 될까요? A의 돈 10,000원은 공중으로 증발해 버립니다. 이런 데이터 불일치는 서비스에 치명적이죠.

`@Transactional`은 이 두 작업을 하나의 **트랜잭션**으로 묶어, 2번 작업까지 모두 성공해야만 최종적으로 데이터베이스에 결과를 저장(커밋, Commit)하고, 중간에 하나라도 실패하면 모든 작업을 없었던 일로 되돌립니다(롤백, Rollback). **이것이 바로 원자성(All-or-Nothing) 보장입니다.**

### 어떻게 동작하는가? - AOP와 프록시의 마법

스프링은 이 마법 같은 일을 AOP(관점 지향 프로그래밍)와 프록시(Proxy) 패턴을 이용해 처리합니다.

`@Transactional`이 붙은 서비스 클래스를 스프링이 Bean으로 등록할 때, 스프링은 **그 서비스의 '가짜 프록시(Proxy) 객체'를 대신 만들어서 컨테이너에 등록**합니다. 우리가 서비스 객체를 주입받아 사용할 때, 사실은 이 프록시 객체를 사용하게 되는 겁니다.

이 프록시 객체는 다음과 같은 순서로 동작합니다.

1. 클라이언트가 서비스의 메소드(`transferMoney()`)를 호출하면, 프록시 객체가 먼저 호출을 가로챕니다.
2. 프록시는 데이터베이스 커넥션을 가져와 트랜잭션을 시작합니다. (`BEGIN TRANSACTION`)
3. 트랜잭션이 시작되면, 프록시는 실제 서비스 객체의 `transferMoney()` 메소드를 호출합니다.
4. 실제 메소드의 로직이 모두 실행됩니다.
5. 메소드 실행 중 예외(Exception)가 발생하지 않았다면, 프록시는 트랜잭션을 **커밋(Commit)**합니다.
6. 만약 예외가 발생했다면, 프록시는 트랜잭션을 **롤백(Rollback)**합니다.

이처럼 우리는 비즈니스 로직에만 집중해서 코드를 짜면 되고, 트랜잭션 처리라는 공통 부가 기능(Aspect)은 스프링의 AOP 프록시가 알아서 처리해 주는 것입니다.

### 어떻게 잘 쓰는가? - 주요 옵션과 주의점

그냥 `@Transactional`만 붙여도 동작하지만, 몇 가지 옵션을 알면 훨씬 정교하게 사용할 수 있습니다.

#### `readOnly = true`

- 용도: 데이터 변경 없이 **조회(SELECT)만 하는 메소드에 사용**합니다.
- 효과: JPA(하이버네이트) 사용 시, 이 옵션을 주면 불필요한 데이터 변경 감지(Dirty Checking) 로직을 생략하고, 읽기 전용으로 최적화하여 조회 성능을 향상시킬 수 있습니다. 서비스 클래스의 모든 조회 메소드에는 붙여주는 것이 좋습니다.

```java
@Transactional(readOnly = true)
public MemberDto findMemberById(Long id) { ... }
```

#### `propagation` (트랜잭션 전파)

- 용도: `@Transactional` 메소드 안에서 다른 `@Transactional` 메소드를 호출할 때, 트랜잭션을 어떻게 처리할지 결정합니다.
- REQUIRED (기본값): 가장 많이 사용합니다. 부모 메소드에 이미 진행 중인 트랜잭션이 있으면 거기에 참여하고, 없으면 새로 시작합니다. "일단 트랜잭션에 포함시켜줘" 라는 의미로, 대부분의 경우 이 기본값으로 충분합니다.

#### `rollbackFor` (롤백 규칙)

- ⚠️ 매우 중요: 기본적으로 스프링은 `RuntimeException`과 그 하위 예외, 그리고 `Error`가 발생했을 때만 롤백을 수행합니다. `Exception`(Checked Exception)은 롤백시키지 않습니다.
- 용도: 특정 Checked Exception이 발생했을 때도 롤백을 하고 싶다면 이 옵션을 사용해야 합니다.

```java
// IOException이 발생해도 롤백하도록 설정
@Transactional(rollbackFor = IOException.class)
public void processFile(String filePath) throws IOException { ... }
```

#### 꼭 알아야 할 주의점

`@Transactional`은 **public 메소드에만 적용**됩니다. `protected`, `private` 메소드에 붙이거나, public이더라도 클래스 내부에서 다른 메소드를 호출(`this.someMethod()`)하는 경우에는 프록시를 거치지 않기 때문에 트랜잭션이 적용되지 않습니다.

## 커스텀 예외 클래스 만들기

### 1단계: 커스텀 예외 클래스 만들기

먼저, 비즈니스 상황을 명확하게 표현하는 예외 클래스를 직접 만듭니다. RuntimeException을 상속받아 Unchecked Exception으로 만드는 것이 일반적입니다.

더 나아가, 에러 코드와 메시지를 체계적으로 관리하기 위해 ErrorCode라는 열거형(Enum)을 함께 사용하는 것이 베스트 프랙티스입니다.

```java
// global/error/ErrorCode.java
import lombok.Getter;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;

@Getter
@RequiredArgsConstructor
public enum ErrorCode {

    // Member
    MEMBER_NOT_FOUND(HttpStatus.NOT_FOUND, "M001", "해당 회원을 찾을 수 없습니다.");

    // 추가적인 에러 코드들...
    // INVALID_INPUT(HttpStatus.BAD_REQUEST, "C001", "잘못된 입력입니다.");

    private final HttpStatus status;
    private final String code;
    private final String message;
}

// domain/member/error/MemberNotFoundException.java
import com.example.myapp.global.error.ErrorCode;
import com.example.myapp.global.error.BusinessException;

// BusinessException은 모든 커스텀 예외의 부모 클래스로 만들어두면 편리합니다.
public class MemberNotFoundException extends BusinessException {

    public MemberNotFoundException() {
        super(ErrorCode.MEMBER_NOT_FOUND);
    }
}

// global/error/BusinessException.java
import lombok.Getter;

@Getter
public class BusinessException extends RuntimeException {

    private final ErrorCode errorCode;

    public BusinessException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }
}
```

**핵심 포인트:**

- ErrorCode Enum: 에러에 대한 모든 정보(HTTP 상태, 고유 코드, 메시지)를 한 곳에서 관리하여 일관성을 유지합니다.
- BusinessException: 모든 비즈니스 예외가 ErrorCode를 갖도록 강제하는 부모 클래스 역할을 합니다.

### 2단계: 서비스 계층에서 예외 던지기 (Throw)

이제 서비스 로직에서 특정 조건에 맞을 때, 방금 만든 커스텀 예외를 던집니다. `Optional`의 `orElseThrow()`를 사용하면 코드를 매우 간결하게 작성할 수 있습니다.

```java
// domain/member/app/MemberService.java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional(readOnly = true)
    public MemberInfoResult findMemberById(Long memberId) {
        Member member = memberRepository.findById(memberId)
                .orElseThrow(MemberNotFoundException::new); // 람다로 예외 생성

        return MemberInfoResult.from(member);
    }
}
```

**핵심 포인트:**

- `orElseThrow(MemberNotFoundException::new)`: `findById`의 결과가 비어있을 경우(`Optional.empty()`), 우리가 만든 `MemberNotFoundException`을 발생시킵니다.
- 서비스 코드는 "**회원이 없으면, 회원 없음 예외를 던진다**"는 비즈니스 로직에만 집중합니다. 이 예외를 어떻게 처리해서 클라이언트에게 보여줄지는 서비스의 관심사가 아닙니다.

### 3단계: 글로벌 예외 핸들러에서 예외 잡기 (Catch)

마지막으로, 애플리케이션 전역에서 발생하는 예외를 한 곳에서 잡아 처리하는 글로벌 예외 핸들러를 만듭니다. `@RestControllerAdvice` 어노테이션을 사용합니다.

```java
// global/error/GlobalExceptionHandler.java
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 우리가 만든 BusinessException을 처리하는 핸들러
    @ExceptionHandler(BusinessException.class)
    protected ResponseEntity<ErrorResponse> handleBusinessException(BusinessException e) {
        ErrorCode errorCode = e.getErrorCode();
        log.warn("BusinessException occurred: {}", e.getMessage(), e);

        ErrorResponse response = ErrorResponse.of(errorCode);
        return new ResponseEntity<>(response, errorCode.getStatus());
    }

    // ... 다른 종류의 예외를 처리하는 핸들러들 ...
}

// global/error/ErrorResponse.java
public record ErrorResponse(String code, String message) {
    public static ErrorResponse of(ErrorCode errorCode) {
        return new ErrorResponse(errorCode.getCode(), errorCode.getMessage());
    }
}
```

**핵심 포인트:**

- `@RestControllerAdvice`: 모든 `@RestController`에서 발생하는 예외를 이 클래스가 가로채도록 합니다.
- `@ExceptionHandler(BusinessException.class)`: `BusinessException` 또는 그 자식 예외들(`MemberNotFoundException` 등)이 발생하면 이 메소드가 실행됩니다.
- 역할 분리: 서비스는 비즈니스 규칙에 맞는 예외를 던지기만 하고, 예외를 실제 HTTP 응답으로 변환하는 책임은 GlobalExceptionHandler가 전담하게 됩니다.

이 3단계 패턴을 따르면, Controller나 Service 코드에 `try-catch`가 난무하는 것을 막고, 예외 처리를 한 곳에서 일관되게 관리할 수 있어 매우 깔끔한 코드를 유지할 수 있습니다.
