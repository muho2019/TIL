# 테스트 코드

## 테스트 코드가 필요한 이유

### 1. 안정성 확보: 버그를 미리 막아주는 자동 탐지기

코드를 수정하거나 새로운 기능을 추가할 때, 가장 무서운 건 **사이드 이펙트**(side effect), 즉 의도치 않은 곳에서 버그가 터지는 겁니다.

- **수동 테스트의 한계**: 기능이 몇 개 없을 땐 직접 눌러보고 확인(수동 테스트)하면 되지만, 기능이 100개, 200개가 되면 모든 걸 다 확인하는 건 불가능에 가깝습니다.
- **자동화된 검증**: 테스트 코드는 내가 수정한 코드 때문에 기존의 다른 기능이 망가졌는지 아닌지를 **수 초~수 분 내에 자동으로 검증**해 줍니다. 새로운 기능 추가 후 전체 테스트를 돌렸을 때 모두 통과한다면, "적어도 기존 기능은 안전하구나!"하는 심리적 안정감을 얻을 수 있죠.

### 2. 자신감 있는 리팩토링: 과감한 코드 개선의 발판

프로젝트가 오래될수록 코드는 복잡해지고 지저분해집니다. 이걸 개선하는 작업을 **리팩토링**이라고 하죠.

- **테스트가 없다면?** 낡고 복잡한 코드를 발견해도, 이걸 고쳤을 때 무슨 일이 벌어질지 몰라 아무도 건드리지 못하는 '유령 코드'가 됩니다. 코드는 점점 썩어가죠.
- **테스트가 있다면?** 든든한 테스트 코드가 있다면 이야기가 다릅니다. 코드 구조를 과감하게 바꾸고, 성능을 개선한 뒤에 테스트를 실행해 보세요. 모든 테스트가 통과한다면, "기능은 그대로 유지하면서 코드 품질은 높였다!"는 확신을 가질 수 있습니다. 즉, **코드를 건강하게 유지할 수 있는 가장 강력한 무기**가 됩니다.

### 3. 살아있는 문서: 가장 정확한 기능 명세서

잘 작성된 테스트 코드는 그 자체로 훌륭한 **문서**가 됩니다.

- **문서의 단점**: 별도의 문서(Word, Wiki 등)는 코드가 변경될 때 함께 수정되지 않으면 금방 낡은 정보가 되어버립니다.
- **테스트 코드의 장점**: 테스트 코드는 실제 동작하는 코드와 함께 있습니다. 예를 들어 `createOrder_with_invalid_coupon_should_throw_exception()` (유효하지 않은 쿠폰으로 주문 시 예외 발생) 같은 테스트 케이스를 보면, **주석이나 문서를 찾아보지 않아도 이 기능이 어떻게 동작해야 하는지 명확하게 이해**할 수 있습니다. 새로운 팀원이 와도 테스트 코드를 보면 빠르게 기능을 파악할 수 있죠.

### 4. 더 좋은 설계 유도: 테스트하기 좋은 코드가 좋은 코드다

역설적으로, 테스트 코드를 작성하려고 노력하면 코드 설계(디자인)가 더 좋아집니다.

- **테스트하기 어려운 코드**: 어떤 클래스가 너무 많은 일을 하거나(단일 책임 원칙 위배), 다른 클래스와 너무 복잡하게 얽혀있으면(강한 결합도) 테스트하기가 매우 어렵습니다.
- **테스트 가능한 설계**: 코드를 테스트하려면 자연스럽게 기능별로 역할을 나누고(모듈화), 의존성을 외부에서 주입받는(DI) 구조로 설계하게 됩니다. 즉, 테스트 용이성(Testability)을 높이는 과정이 자연스럽게 객체지향 원칙을 지키는 좋은 설계로 이어지는 경우가 많습니다.

물론, 테스트 코드를 작성하는 데는 분명 시간이 더 듭니다. 하지만 장기적으로 봤을 때, 버그를 잡고, 수동으로 테스트하고, 잘못된 코드를 이해하느라 낭비하는 시간을 훨씬 더 많이 줄여주는 현명한 투자입니다.

## 단위 테스트

- **테스트 대상**: 클래스나 메소드 하나처럼, 테스트할 수 있는 가장 작은 코드 단위(Unit)입니다.
- **핵심 목적**: 특정 로직(예: 이 메소드에 10을 넣으면 20이 나오는가?)이 독립적으로 잘 동작하는지 검증하는 것입니다.
- **가장 중요한 특징**: 격리 (Isolation)
  - 테스트하려는 코드 외의 **다른 모든 것(DB, 외부 API, 다른 클래스 등)은 가짜(Mock)로 대체**합니다.
  - 예를 들어, `UserService`의 `join()` 메소드를 테스트한다면, 이 메소드가 호출하는 `UserRepository`는 실제 DB에 연결된 객체가 아니라, "이런 데이터가 들어오면 무조건 true를 반환해라"라고 우리가 정해준 가짜 객체를 사용합니다.
- **장점**:
  - 속도가 매우 빠릅니다. (DB나 네트워크 연결이 없으므로)
  - 어떤 테스트가 실패하면, **버그가 어디에 있는지 즉시 특정**할 수 있습니다. (딱 그 메소드에 문제가 있다는 뜻이니까요)
- **단점**:
  - 실제 환경과 다르기 때문에, 다른 컴포넌트와 연결되었을 때 발생하는 문제는 잡아낼 수 없습니다.
- **작성 시 가져야 할 의도**:
  - 단위 테스트를 작성할 때는 스스로를 그 메소드나 클래스를 설계한 사람이라고 생각해야 합니다. 관심사는 오직 '이 부품(Unit)이 명세서대로 정확하게 동작하는가?' 뿐입니다. 다른 부품과의 연결은 전혀 신경 쓰지 않습니다.
  - 1. **명세(Specification)를 검증하겠다.**
    - **정상 케이스 (Happy Path)**: 의도한 입력값을 넣었을 때, 기대하는 결과가 정확히 나오는가?
      - (의도) "이 `calculateInterest` 메소드는 원금 100만원과 이율 5%를 받으면, 이자 5만원을 정확히 반환해야 한다."
    - **예외 케이스 (Sad Path)**: 잘못된 입력값을 넣었을 때, 약속된 오류(Exception)를 올바르게 발생시키는가?
      - (의도) "여기에 음수(-) 원금을 넣으면, `IllegalArgumentException`을 터뜨리는 게 이 메소드의 올바른 동작이다."
    - **경계값 케이스 (Boundary Cases)**: 경계에 해당하는 값(0, null, 빈 리스트, 최댓값 등)을 넣었을 때, 논리적 오류 없이 처리되는가?
      - (의도) "가입할 회원의 이름으로 `null`이나 빈 문자열이 들어오면, 어떻게 처리하기로 약속했지? 그것을 검증해야겠다."
  - 2. **구현이 아닌 '결과'를 테스트하겠다.**
    - 메소드 내부가 `for`문으로 짜여있든 `stream`으로 짜여있든 그건 중요하지 않습니다. 중요한 것은 "무엇을 넣으면 무엇이 나오는가" 라는 입력과 출력의 관계입니다.
    - 이런 관점은 나중에 내부 코드를 리팩토링할 때 엄청난 자신감을 줍니다. 내부 구현을 어떻게 바꾸든, 이 테스트만 통과하면 "기능은 그대로구나!" 확신할 수 있으니까요.

### 기술

#### JUit

JUnit은 자바의 표준 테스트 프레임워크입니다. 테스트 코드를 실행하고, 생명주기를 관리하며, 구조화하는 역할을 하죠. 핵심 어노테이션 몇 개만 알면 충분합니다.

- `@Test`: 가장 기본이죠. "이 메소드는 테스트 케이스입니다"라고 알려주는 표시입니다.
- `@DisplayName("설명")`: 테스트에 한글 등으로 알기 쉬운 이름을 붙여줍니다. 예를 들어 `@DisplayName("존재하지 않는 ID로 조회 시 예외 발생")` 처럼요. 테스트가 실패했을 때 이 이름이 보이므로 매우 유용합니다.
- `@BeforeEach` / `@AfterEach`: 각각의 `@Test` 메소드가 실행되기 전/후에 항상 실행되는 코드를 정의합니다. 여러 테스트에서 중복되는 객체 생성이나 데이터 초기화 코드를 여기에 넣으면 코드가 깔끔해집니다.
- `@Nested`: 관련 있는 테스트들을 하나의 내부 클래스로 묶어 구조화할 수 있습니다. 예를 들어 `회원가입_성공_케이스` 클래스와 `회원가입_실패_케이스` 클래스로 나눌 수 있죠.

#### Mockito

단위 테스트의 핵심인 **격'**를 가능하게 해주는 기술입니다. 테스트할 객체(SUT, System Under Test)가 의존하는 객체들을 가짜(Mock)로 만들고, 이 가짜 객체의 행동을 마음대로 조종할 수 있게 해줍니다.

- `@Mock`: 가짜 객체를 만들 필드에 붙여줍니다. 이 객체는 껍데기만 있고 내부는 비어있습니다.
- `@InjectMocks`: 테스트할 실제 객체 필드에 붙입니다. Mockito가 이 객체를 생성할 때, `@Mock`이 붙은 가짜 객체들을 자동으로 주입해 줍니다. (new로 생성하고 수동으로 주입할 필요가 없어져요!)
- `when(mock.method()).thenReturn(value)`: 가짜 객체의 행동을 정의하는 가장 중요한 구문입니다.
  - `when(memberRepository.findById(1L)).thenReturn(Optional.of(testMember));`
  - → "`memberRepository`의 `findById(1L)` 메소드가 호출되면, `testMember`를 반환해줘!" 라는 의미입니다.
- `verify(mock).method()`: 가짜 객체의 특정 메소드가 호출되었는지 검증합니다.
  - `verify(emailService, times(1)).sendWelcomeEmail("test@email.com");`
  - → "`emailService`의 `sendWelcomeEmail` 메소드가 "test@email.com" 인자와 함께 정확히 1번 호출되었는지 확인해줘!" 라는 의미입니다.

#### AssertJ

테스트의 마지막 단계인 '검증(Assert)'을 사람이 읽기 좋은 문장처럼 만들어주는 라이브러리입니다. JUnit의 기본 `assertEquals`보다 훨씬 직관적이고 강력합니다.

- **핵심 문법**: `assertThat(실제값).is...()`
- **가독성**: `assertThat(member.getAge()).isEqualTo(20);` ("멤버의 나이가 20과 같은지 확인해줘.") 처럼 자연스럽게 읽힙니다.
- **메소드 체이닝**: 여러 검증을 연결해서 쓸 수 있어 편리합니다.
  - `assertThat(names).hasSize(3).contains("Kim", "Lee").doesNotContain("Park");`
- **강력한 기능**: 예외 테스트, 리스트/객체 필드 검증 등 다양한 상황에 맞는 검증 메소드를 제공합니다.
  - `assertThatThrownBy(() -> memberService.join(duplicateMemberDto)).isInstanceOf(DuplicateMemberException.class);` ("이 코드를 실행했을 때, `DuplicateMemberException`이 발생하는지 확인해줘.")

#### 예시 코드

```java
// 1. JUnit과 Mockito 연동
@ExtendWith(MockitoExtension.class)
class MemberServiceTest {

    // 2. Mockito로 가짜 객체(Repository)와 실제 객체(Service) 준비
    @InjectMocks
    private MemberService memberService;
    @Mock
    private MemberRepository memberRepository;

    @Test
    @DisplayName("정상적으로 회원가입에 성공한다")
    void join_success() {
        // given (준비) - Arrange
        Member newMember = new Member("test@email.com", "password");
        MemberDto memberDto = new MemberDto("test@email.com", "password");

        // memberRepository.findByEmail()이 호출되면 빈 Optional을 반환하도록 행동 정의
        when(memberRepository.findByEmail("test@email.com")).thenReturn(Optional.empty());
        // memberRepository.save()가 호출되면 newMember를 반환하도록 행동 정의
        when(memberRepository.save(any(Member.class))).thenReturn(newMember);

        // when (실행) - Act
        Long savedMemberId = memberService.join(memberDto);

        // then (검증) - Assert
        // 3. AssertJ로 결과(상태) 검증
        assertThat(savedMemberId).isEqualTo(newMember.getId());
        // memberRepository의 save가 정확히 1번 호출되었는지 행위 검증
        verify(memberRepository, times(1)).save(any(Member.class));
    }
}

@ExtendWith(MockitoExtension.class)
class MemberServiceExceptionTest {

    @InjectMocks
    private MemberService memberService;
    @Mock
    private MemberRepository memberRepository;

    @Test
    @DisplayName("이미 존재하는 이메일로 가입 시 DuplicateMemberException이 발생한다")
    void join_with_duplicate_email_should_throw_exception() {
        // given (준비)
        // 테스트용 가입 정보와 이미 DB에 존재하는 회원 객체 준비
        MemberDto memberDto = new MemberDto("test@email.com", "password123");
        Member existingMember = new Member("test@email.com", "any_password");

        // "memberRepository.findByEmail()을 호출하면, existingMember를 담은 Optional을 반환해라"
        // 즉, 이미 존재하는 회원인 상황을 강제로 연출
        when(memberRepository.findByEmail("test@email.com")).thenReturn(Optional.of(existingMember));

        // when & then (실행 및 검증)
        // AssertJ의 assertThatThrownBy를 사용하여 예외 테스트
        assertThatThrownBy(() -> memberService.join(memberDto))
                .isInstanceOf(DuplicateMemberException.class) // 1. 예외 타입이 맞는지
                .hasMessageContaining("이미 존재하는 이메일입니다."); // 2. 예외 메시지가 맞는지

        // 추가 검증: 예외가 발생했으니, save 메소드는 절대 호출되면 안 된다.
        verify(memberRepository, never()).save(any(Member.class));
    }
}

@ExtendWith(MockitoExtension.class)
class MemberServiceBoundaryTest {

    @InjectMocks
    private MemberService memberService;

    // 참고: 이 테스트는 Repository까지 갈 필요가 없으므로 @Mock은 불필요.

    @ParameterizedTest // 여러 파라미터를 넣어 반복 테스트를 하겠다!
    @ValueSource(strings = {"1234567", "123456789012345678901"}) // 7자리, 21자리 비밀번호
    @DisplayName("비밀번호 길이가 7자 이하 또는 21자 이상일 때 IllegalArgumentException이 발생한다")
    void join_with_invalid_password_length_should_throw_exception(String invalidPassword) {
        // given (준비)
        MemberDto memberDto = new MemberDto("test@email.com", invalidPassword);

        // when & then (실행 및 검증)
        assertThatThrownBy(() -> memberService.join(memberDto))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessageContaining("비밀번호는 8자 이상 20자 이하이어야 합니다.");
    }
}
```

**Mock 객체 주입 기준**:

- **Mock으로 만들어야 하는 대상 (`@Mock`)**
  - **Repository 계층**: 데이터베이스와 통신하는 모든 객체.
  - **다른 Service 계층**: 테스트하려는 서비스가 의존하는 다른 비즈니스 로직 객체. (`OrderService` 테스트 시 `PaymentService` Mock 처리)
  - **외부 API 통신 객체**: `RestTemplate`, `WebClient` 등 외부 서버와 통신하는 모든 객체.
  - **유틸리티 클래스 중 로직이 복잡한 경우**: 내부에 복잡한 로직을 가진 Helper나 Util 클래스.
- **Mock으로 만들 필요 없는 대상 (실제 객체 `new`로 생성)**
  - **DTO (Data Transfer Object)나 VO (Value Object)**: 이들은 단순한 데이터 덩어리일 뿐, 자체적인 비즈니스 로직을 갖지 않습니다. `new MemberDto(...)`처럼 직접 생성해서 사용하는 것이 훨씬 간단하고 명확합니다.
  - **Enum (열거형)**: 상수 모음이므로 Mock으로 만들 이유가 없습니다.

**행위 검증(`verify`) 작성 방법**:

행위 검증은 꼭 필요한 '핵심적인 부작용(Side Effect)이나 상호작용'에만 선별적으로 사용해야 합니다. 다음의 두 가지 황금률을 기억하세요.

- **1. 테스트 대상이 '외부'와 소통하는지 확인할 때**
  - 여기서 '외부'란, 테스트 중인 서비스(클래스)의 경계를 넘어 다른 시스템과 상호작용하는 지점을 의미합니다. 이것이 verify의 가장 중요하고 올바른 사용처입니다.
  - **데이터베이스**: `repository.save(data)`, `repository.delete(id)` 호출 여부
  - **외부 API**: `apiClient.sendRequest(request)` 호출 여부
  - **메시지 큐**: `messageProducer.send("message")` 호출 여부
  - **이메일/알림**: `notificationManager.sendEmail(email)` 호출 여부
  - 이런 호출들은 테스트 중인 메소드의 중요한 결과입니다. 예를 들어, `createUser()` 메소드의 임무는 단순히 User 객체를 반환하는 것뿐만 아니라, **데이터베이스에 유저를 저장하라**는 명령을 내리는 것입니다. 이 명령이 제대로 내려졌는지 확인하는 것이 바로 행위 검증의 역할입니다.
- **2. 다른 방법으로 검증하기 어려울 때 (주로 반환값이 `void`일 때)**
  - 메소드가 특정 값을 반환한다면, 우리는 그 반환된 **상태**를 검증하는 것이 훨씬 좋습니다. 하지만 `void`를 반환하는 메소드는 검증할 상태가 없습니다. 이 메소드의 유일한 임무가 다른 객체의 메소드를 호출하는 것일 수 있습니다.

## 통합 테스트

- **테스트 대상**: 여러 컴포넌트(모듈)들이 서로 연결된 상태입니다.
- **핵심 목적**: "사용자가 회원가입 버튼을 누르면, 컨트롤러 -> 서비스 -> 리포지토리를 거쳐 DB에 데이터가 올바르게 저장되는가?"처럼, 전체적인 흐름과 상호작용이 문제없이 동작하는지 검증하는 것입니다.
- **가장 중요한 특징**: 실제 환경과의 유사성
  - 실제 의존성을 사용하는 경우가 많습니다. (가짜 DB가 아닌 실제 테스트용 DB에 연결, 외부 API도 실제 호출)
  - 스프링에서는 보통 `@SpringBootTest` 어노테이션을 붙여서, 실제 애플리케이션처럼 모든 Bean을 스프링 컨테이너에 띄워서 테스트합니다.
- **장점**:
  - 시스템의 전반적인 신뢰도를 높여줍니다. (단위 테스트가 모두 통과해도, 통합 테스트에서 문제가 터지는 경우가 아주 많습니다.)
  - 설정 오류, DB 스키마 문제, API 연동 오류 등 **단위 테스트로는 잡을 수 없는 문제들을 발견**할 수 있습니다.
- **단점**:
  - 속도가 매우 느립니다. (스프링 컨테이너를 띄우고 DB에 연결하는 등 준비 과정이 깁니다.)
  - 테스트가 실패했을 때, **어디가 원인인지 한 번에 파악하기 어렵습니다.** (컨트롤러 문제인지, 서비스 문제인지, DB 문제인지 다 살펴봐야 합니다.)
- **작성 시 가져야 할 의도**
  - 통합 테스트를 작성할 때는 스스로를 이 기능을 사용하는 최종 사용자 혹은 클라이언트라고 생각해야 합니다. 내부 부품 하나하나의 동작 방식보다는, "그래서 이 버튼(API)을 누르면, 내가 원하는 결과가 제대로 나오는가?" 라는 **전체 시나리오**(Scenario)에 집중합니다.
  - **컴포넌트 간의 '연결'을 의심하겠다.**
    - (의도) "`Controller`가 받은 요청을 `Service`에게 잘 전달하는가? `Service`가 처리한 데이터를 `Repository`가 DB에 잘 저장하는가? 중간에 데이터가 변질되거나 누락되지는 않는가?"
    - (의도) "A 서비스가 B 서비스를 API로 호출할 때, 둘이 약속한 데이터 형식(JSON)은 잘 맞나? 네트워크 설정이나 인증 문제는 없나?"
  - **외부 시스템과의 '연동'을 검증하겠다.**
    - (의도) "우리 애플리케이션의 설정 (`application.yml`)은 올바른가? 실제 DB(Test DB)에 연결했을 때, 테이블이나 컬럼 이름이 달라서 에러가 나지는 않는가?"
    - (의도) "JPA(ORM) 설정은 올바른가? 내가 만든 엔티티 클래스와 실제 DB 테이블이 잘 매핑되어서, 데이터가 깨지지 않고 잘 들어가고 조회되는가?"
  - **하나의 완전한 '사용자 스토리'를 테스트하겠다.**
    - (의도) "'사용자가 로그인 버튼을 누른다' → ID/PW를 검증한다 → 성공하면 토큰을 발급한다 → '발급된 토큰으로 게시글 작성을 요청한다' → 토큰이 유효한지 검증한다 → 게시글을 DB에 저장한다 → '성공 응답을 받는다'. 이 모든 흐름이 물 흐르듯 잘 이어지는가?"
