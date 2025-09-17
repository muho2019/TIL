# 목차

- [목차](#목차)
- [기본](#기본)
  - [1. 핵심 아키텍처: 필터 체인 (Filter Chain)](#1-핵심-아키텍처-필터-체인-filter-chain)
  - [2. 인증 (Authentication): "당신은 누구신가요?"](#2-인증-authentication-당신은-누구신가요)
  - [3. 인가 (Authorization): "무엇을 할 수 있나요?"](#3-인가-authorization-무엇을-할-수-있나요)
  - [4. 주요 구성 요소 요약](#4-주요-구성-요소-요약)
- [JWT 생성 방법](#jwt-생성-방법)
  - [1단계: 라이브러리 추가](#1단계-라이브러리-추가)
  - [2단계: Secret Key 생성 및 관리](#2단계-secret-key-생성-및-관리)
  - [3단계: JWT 생성 코드 작성](#3단계-jwt-생성-코드-작성)
- [JWT 인증 확인 방법](#jwt-인증-확인-방법)
  - [1. 가장 간편한 방법: @AuthenticationPrincipal (Controller에서)](#1-가장-간편한-방법-authenticationprincipal-controller에서)
  - [2. 정석적인 방법: `SecurityContextHolder` (Service 등 어디서든)](#2-정석적인-방법-securitycontextholder-service-등-어디서든)
  - [3. 동작 원리: JWT 필터와 `SecurityContextHolder`](#3-동작-원리-jwt-필터와-securitycontextholder)
- [세션 저장소](#세션-저장소)
  - [왜 레디스를 사용할까?](#왜-레디스를-사용할까)
  - [스프링 세션을 레디스로 연동하는 방법](#스프링-세션을-레디스로-연동하는-방법)
    - [1단계: 의존성 추가](#1단계-의존성-추가)
    - [2단계: `application.yml` 설정](#2단계-applicationyml-설정)
    - [3단계: `@EnableRedisHttpSession` 어노테이션 추가](#3단계-enableredishttpsession-어노테이션-추가)
    - [(선택) 로컬 테스트를 위한 Docker Compose 설정](#선택-로컬-테스트를-위한-docker-compose-설정)
- [역할 제한 방법](#역할-제한-방법)
  - [1. 메서드 보안: `@PreAuthorize` 사용 (강력 추천)](#1-메서드-보안-preauthorize-사용-강력-추천)
    - [1단계: 메서드 보안 활성화](#1단계-메서드-보안-활성화)
    - [2단계: 컨트롤러 메서드에 어노테이션 추가](#2단계-컨트롤러-메서드에-어노테이션-추가)
  - [2. 설정 기반: `SecurityFilterChain`에서 URL 패턴으로 제어](#2-설정-기반-securityfilterchain에서-url-패턴으로-제어)

# 기본

스프링 시큐리티의 기본기는 '인증(Authentication)'과 '인가(Authorization)'라는 두 가지 핵심 개념을 중심으로 한 아키텍처를 이해하는 것입니다.

가장 쉽게 비유하자면, 스프링 시큐리티는 'VIP 클럽의 보안 시스템'과 같습니다.

- 인증 (Authentication): 클럽 입구에서 "당신이 누구인지" 신분증을 확인하는 과정.
- 인가 (Authorization): 클럽 안에 들어온 후, "당신이 VIP 룸에 들어갈 자격이 있는지" 권한을 확인하는 과정.

## 1. 핵심 아키텍처: 필터 체인 (Filter Chain)

스프링 시큐리티의 가장 핵심적인 구조는 '필터 체인'입니다. 클라이언트로부터 들어오는 모든 요청(Request)은 애플리케이션의 컨트롤러에 도달하기 전에, 스프링 시큐리티가 설치한 여러 보안 필터들을 순서대로 통과해야 합니다.

마치 클럽에 들어가기 위해 신분증 검사, 복장 검사, 소지품 검사 등 여러 단계를 거치는 것과 같습니다. 이 필터 체인은 `SecurityFilterChain` Bean을 등록하여 구성하며, 각 필터는 보안과 관련된 특정 작업을 수행합니다.

## 2. 인증 (Authentication): "당신은 누구신가요?"

사용자가 로그인을 시도할 때 일어나는 과정입니다. 스프링 시큐리티는 이 과정을 여러 전문 컴포넌트들의 협력으로 처리합니다.

**[인증 흐름]**

1. 사용자 로그인 시도: 사용자가 아이디와 비밀번호를 입력하고 `/login` 같은 주소로 요청을 보냅니다.
2. `AuthenticationFilter`: 이 필터는 아이디/비밀번호 요청을 가로채서, `UsernamePasswordAuthenticationToken`이라는 '미인증된 신분증' 객체를 만듭니다.
3. `AuthenticationManager` (인증 관리자): 필터는 이 '미인증 신분증'을 '인증 관리자'에게 보냅니다. 인증 관리자는 실제 인증을 수행할 전문가(`AuthenticationProvider`)를 찾아 일을 위임합니다.
4. `AuthenticationProvider` (인증 전문가):
   1. `UserDetailsService`를 통해 입력된 아이디에 해당하는 실제 회원 정보를 DB에서 찾아옵니다.
   2. `PasswordEncoder`를 사용해, 사용자가 입력한 비밀번호와 DB에 저장된 암호화된 비밀번호가 일치하는지 비교합니다.
5. 인증 성공: 비밀번호가 일치하면, `AuthenticationProvider`는 사용자 정보와 권한(`Authorities`)이 모두 담긴 '인증된 신분증(`Authentication` 객체)'을 생성하여 반환합니다.
6. `SecurityContextHolder`: 최종적으로, 이 '인증된 신분증'은 `SecurityContextHolder`라는 '세션 보관소'에 저장됩니다. 이제 이 사용자는 현재 요청뿐만 아니라, 로그아웃하기 전까지 계속 인증된 상태로 유지됩니다.

## 3. 인가 (Authorization): "무엇을 할 수 있나요?"

인증이 완료된 사용자가 특정 페이지나 기능에 접근할 때 일어나는 과정입니다.

1. 사용자 요청: 인증된 사용자가 `/admin/dashboard` 같은 페이지에 접근을 시도합니다.
2. `AuthorizationFilter`: 인가 필터가 이 요청을 가로챕니다.
3. 권한 확인: 필터는 `SecurityContextHolder`에서 현재 사용자의 '인증된 신분증'을 꺼내, 그 안에 담긴 권한 목록(`Authorities`, 예: `ROLE_ADMIN`)을 확인합니다.
4. 접근 결정: 미리 설정된 규칙(예: `/admin/**` 경로는 `ROLE_ADMIN` 권한이 필요)과 사용자의 권한을 비교합니다.
   1. 권한 있음: 필터를 통과시켜 요청이 컨트롤러로 전달되도록 허용합니다.
   2. 권한 없음: 접근을 차단하고 `403 Forbidden` 에러를 반환합니다.

## 4. 주요 구성 요소 요약

- `Authentication`: 사용자의 인증 정보를 담는 객체. 인증 전에는 자격 증명(아이디/비번), 인증 후에는 사용자 상세 정보와 권한을 담습니다.
- `UserDetails`: DB 등에서 조회한 사용자 정보를 스프링 시큐리티 형식에 맞게 감싸주는 '어댑터' 클래스입니다.
- `UserDetailsService`: `UserDetails` 객체를 불러오는(load) 역할을 하는 핵심 인터페이스입니다. 보통 DB에서 사용자를 조회하는 로직을 여기에 구현합니다.
- `PasswordEncoder`: 비밀번호를 안전하게 암호화하고, 입력된 비밀번호와 암호화된 비밀번호를 비교하는 역할을 합니다.
- `SecurityContextHolder`: 인증된 사용자 정보를 현재 스레드 내에 저장하여, 애플리케이션의 어느 곳에서든 해당 사용자 정보에 접근할 수 있게 해주는 저장소입니다.

# JWT 생성 방법

JWT(JSON Web Token)는 전용 라이브러리(JJWT 등)를 사용하여, 토큰에 담을 정보(Claims), 만료 시간 등을 설정하고, 서버만 아는 비밀 키(Secret Key)로 서명하여 생성합니다.

마치 위조 방지 홀로그램 스티커가 붙은 'VIP 이벤트 티켓'을 만드는 것과 같습니다.

- Claims (정보): 티켓에 적힌 내용 (이름: user123, 등급: ROLE_USER, 좌석: A1)
- Expiration (만료 시간): 티켓의 유효 기간 ("~9월 18일 자정까지 유효")
- Secret Key (비밀 키): 오직 주최 측만 가진 '비밀 홀로그램 스탬프'
- Signature (서명): 스탬프로 티켓에 찍은 '위조 방지 홀로그램'

이 티켓은 누구나 내용을 볼 수 있지만, 비밀 스탬프가 없으면 똑같이 만들 수 없으며, 홀로그램이 훼손되면 위조된 것임을 바로 알 수 있습니다.

## 1단계: 라이브러리 추가

Java에서는 `io.jsonwebtoken (JJWT)` 라이브러리가 사실상 표준입니다. `build.gradle`에 의존성을 추가합니다.

**[build.gradle]**

```groovy
dependencies {
    // JJWT 라이브러리
    implementation 'io.jsonwebtoken:jjwt-api:0.12.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.12.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.12.5' // Jackson이 필수는 아니지만, 가장 일반적입니다.
}
```

## 2단계: Secret Key 생성 및 관리

JWT의 서명을 위해 사용할 비밀 키를 생성합니다. 이 키는 절대 외부에 노출되어서는 안 됩니다.

- ⚠️ 중요: 비밀 키를 소스 코드에 하드코딩하면 안 됩니다. `application.yml`이나 환경 변수를 통해 외부에서 주입받아야 합니다.
- 키는 충분히 길고 복잡한 문자열(Base64 인코딩된)을 사용하는 것이 좋습니다.

**[JwtProvider.java - 키 초기화]**

```java
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import javax.crypto.SecretKey;
import java.util.Date;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.temporal.ChronoUnit;

@Component
public class JwtProvider {

    private final SecretKey secretKey;

    // application.yml 등에서 설정한 비밀 키 값을 주입받음
    public JwtProvider(@Value("${jwt.secret-key}") String secretString) {
        // Base64 문자열을 디코딩하여 SecretKey 객체로 변환
        byte[] keyBytes = Decoders.BASE64.decode(secretString);
        this.secretKey = Keys.hmacShaKeyFor(keyBytes);
    }

    // ... 토큰 생성 메서드 ...
}
```

**[application.yml]**

```yaml
jwt:
  # 64바이트 이상의 긴 랜덤 문자열을 Base64 인코딩하여 사용
  secret-key: aG90ZWwtYm9va2luZy1wbGF0Zm9ybS1zcHJpbmctYm9vdC1qd3Qtc2VjcmV0LWtleS1mb3ItcHJvamVjdC1iYWNra3UgMTIzNDU2Nzg5MA==
```

## 3단계: JWT 생성 코드 작성

이제 사용자 정보를 받아 JWT를 생성하는 메서드를 만듭니다.

**[JwtProvider.java - 토큰 생성]**

```java
public String createToken(String userId, String role, String email) {
    Date now = new Date();
    // 토큰 만료 시간: 1시간 후
    Date expiryDate = new Date(now.getTime() + 3600000);

    return Jwts.builder()
        // 1. Subject: 토큰의 주체를 설정 (보통 사용자의 고유 식별자)
        .subject(userId)
        // 2. Claims: 토큰에 담을 추가 정보 (Key-Value)
        .claim("role", role)
        .claim("email", email)
        // 3. Issued At: 토큰 발급 시간
        .issuedAt(now)
        // 4. Expiration: 토큰 만료 시간
        .setExpiration(expiryDate)
        // 5. Signature: 사용할 서명 알고리즘과 Secret Key 설정
        .signWith(secretKey)
        // 6. 최종적으로 JWT 문자열 생성
        .compact();
}
```

**코드 해설**

- `subject(userId)`: 이 토큰이 누구에 대한 것인지 나타냅니다. 보통 사용자의 이메일이나 ID를 사용합니다.
- `claim("role", role)`: 비공개 클레임(Private Claim)입니다. role이라는 이름으로 사용자의 권한 정보를 저장하는 등, 원하는 데이터를 자유롭게 추가할 수 있습니다.
- `issuedAt(now)`: 토큰이 언제 발급되었는지 기록합니다. (Issued At)
- `expiration(expiryDate)`: 가장 중요한 설정 중 하나입니다. 이 토큰이 언제까지 유효한지 나타냅니다. (Expiration Time) 이 시간이 지나면 토큰은 유효하지 않게 됩니다.
- `signWith(secretKey)`: 2단계에서 만든 SecretKey 객체를 사용하여 서명을 생성합니다. jjwt 0.12.x 버전부터는 키 자체에 알고리즘 정보가 포함되어 있어, 알고리즘을 별도로 명시하지 않아도 됩니다.
- `compact()`: 위에서 설정한 모든 내용을 바탕으로 xxxxx.yyyyy.zzzzz 형태의 JWT 문자열을 최종적으로 만들어 반환합니다.

이렇게 생성된 토큰을 클라이언트에게 전달하면, 클라이언트는 다음 요청부터 이 토큰을 `Authorization` 헤더에 담아 보내게 됩니다.

# JWT 인증 확인 방법

가장 쉬운 방법은 컨트롤러에서 `@AuthenticationPrincipal` 어노테이션을 사용하는 것이고, 어디서든 접근할 수 있는 정석적인 방법은 `SecurityContextHolder`를 이용하는 것입니다.

마치 놀이공원에 비유할 수 있습니다.

- `JWT 토큰`: 입장할 때 발급받은 바코드가 찍힌 '자유이용권 팔찌'
- `JWT 인증 필터`: 놀이기구 입구에서 팔찌의 바코드를 스캔하는 '직원'
- `SecurityContextHolder`: 직원이 스캔 후 "이 사람은 OOO이고, VIP 회원입니다"라고 기록해두는 '임시 방문 기록부'
- `@AuthenticationPrincipal`: 방문 기록부에서 내 이름(사용자 정보)을 바로 꺼내보는 가장 간편한 방법

## 1. 가장 간편한 방법: @AuthenticationPrincipal (Controller에서)

컨트롤러 메서드의 파라미터에 `@AuthenticationPrincipal` 어노테이션을 붙이면, 스프링 시큐리티가 현재 로그인된 사용자의 상세 정보 객체(`Principal`)를 알아서 주입해 줍니다. 가장 깔끔하고 권장되는 방법입니다.

**준비물: 사용자 상세 정보 클래스 (`UserPrincipal`)**

먼저, JWT 토큰의 정보를 바탕으로 생성될, 사용자 정보를 담는 클래스가 필요합니다. 보통 `UserDetails`를 구현하여 만들며, 여기에 `id` 같은 추가 정보를 담을 수 있습니다.

```java
// UserPrincipal.java
// record 타입을 사용하면 간결하게 작성 가능
public record UserPrincipal(
    Long id,
    String email,
    Collection<? extends GrantedAuthority> authorities
) implements UserDetails {
    // ... UserDetails 인터페이스의 다른 메서드 구현 ...
}
```

**컨트롤러 사용 예시**

```java
@RestController
@RequestMapping("/api/members")
public class MemberController {

    @GetMapping("/me")
    public ResponseEntity<MemberInfoResponse> getMyInfo(
        @AuthenticationPrincipal UserPrincipal userPrincipal
    ) {
        // userPrincipal 객체에 현재 로그인된 사용자의 정보가 담겨있습니다.
        Long memberId = userPrincipal.id();
        String email = userPrincipal.email();

        // 이 정보를 바탕으로 비즈니스 로직을 처리합니다.
        MemberInfoResult result = memberService.findMemberById(memberId);
        return ResponseEntity.ok(MemberInfoResponse.from(result));
    }
}
```

## 2. 정석적인 방법: `SecurityContextHolder` (Service 등 어디서든)

`SecurityContextHolder`는 현재 요청 스레드의 보안 정보를 담고 있는 저장소입니다. 이 클래스의 static 메서드를 이용하면 애플리케이션의 어느 계층에서든 사용자 정보에 접근할 수 있습니다.

```java
@Service
public class MemberService {

    public void someBusinessLogic() {
        // 1. SecurityContextHolder에서 Authentication 객체를 가져온다.
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        // 2. Authentication 객체에서 Principal(사용자 정보)을 가져온다.
        Object principal = authentication.getPrincipal();

        // 3. 가져온 Principal을 실제 사용자 상세 정보 객체로 형변환(cast)한다.
        if (principal instanceof UserPrincipal userPrincipal) {
            Long currentUserId = userPrincipal.id();
            // ... 이 사용자의 ID를 가지고 로직 수행 ...
        }
    }
}
```

- Pro Tip: 서비스 계층이 `SecurityContextHolder`에 직접 의존하는 것은 테스트 용이성 등을 해칠 수 있습니다. 가급적 Controller에서 `@AuthenticationPrincipal`로 사용자 ID를 꺼낸 후, 서비스 메서드에는 파라미터로 명시적으로 넘겨주는 방식이 더 좋은 설계입니다.

## 3. 동작 원리: JWT 필터와 `SecurityContextHolder`

이 모든 것이 가능한 이유는 우리가 직접 구현하는 `JwtAuthenticationFilter` 덕분입니다.

1. 클라이언트가 요청 헤더에 JWT를 담아 보냅니다.
2. JwtAuthenticationFilter가 요청을 가로채 JWT를 검증합니다.
3. 토큰이 유효하면, 토큰 안의 payload(claims)에서 사용자 ID, 이메일, 역할(Role) 등의 정보를 꺼냅니다.
4. 꺼낸 정보로 위에서 만든 UserPrincipal 객체와 Authentication 객체(보통 UsernamePasswordAuthenticationToken)를 생성합니다.
5. 생성된 Authentication 객체를 SecurityContextHolder에 저장합니다.
6. 이후 해당 요청이 끝날 때까지 애플리케이션의 어느 곳에서든 SecurityContextHolder를 통해 방금 저장한 사용자 정보에 접근할 수 있게 됩니다.

# 세션 저장소

스프링 부트의 내장 웹 서버(Tomcat 등)는 기본적으로 서버의 메모리에 세션 정보를 저장합니다.

메모리에 저장하는 방식은 간단하지만, 서버를 재시작하면 모든 로그인 정보가 사라지고, 서버를 여러 대로 늘리면(로드 밸런싱) 세션 불일치 문제가 발생하는 치명적인 단점이 있습니다. 이를 해결하기 위해 레디스(Redis)와 같은 외부 세션 저장소를 사용합니다.

## 왜 레디스를 사용할까?

레디스는 메모리 기반의 Key-Value 데이터 저장소로, 디스크 기반의 데이터베이스보다 훨씬 빠릅니다. 따라서 수시로 읽고 써야 하는 세션 데이터를 저장하기에 매우 적합합니다.

마치 서버마다 각자 '개인 수첩'에 고객 정보를 적어두는 대신, 모든 직원이 함께 볼 수 있는 '중앙 관제실 화이트보드' 칠판에 정보를 기록하는 것과 같습니다. 어느 직원이든(어느 서버든) 이 화이트보드(레디스)만 보면 고객(사용자)의 상태를 알 수 있고, 관제실 전원이 꺼지지 않는 한 정보는 안전하게 유지됩니다.

## 스프링 세션을 레디스로 연동하는 방법

스프링 부트에서는 아주 간단한 설정 몇 단계만으로 세션 저장소를 레디스로 변경할 수 있습니다.

### 1단계: 의존성 추가

`build.gradle`에 Spring Data Redis와 Spring Session Data Redis 의존성을 추가합니다.

**[build.gradle]**

```groovy
dependencies {
    // Spring Data Redis
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'

    // Spring Session Data Redis
    implementation 'org.springframework.session:spring-session-data-redis'
}
```

### 2단계: `application.yml` 설정

`application.yml` 파일에 세션 저장소 타입을 redis로 지정하고, 접속할 레디스 서버의 정보를 입력합니다.

**[application.yml]**

```yaml
spring:
  # 1. 세션 저장소 타입을 redis로 지정
  session:
    store-type: redis
    # 세션 타임아웃 설정 (예: 30분)
    timeout: 30m

  # 2. 접속할 Redis 서버 정보
  data:
    redis:
      host: localhost # 로컬 PC에 설치된 Redis
      port: 6379
      # password: your-redis-password # 비밀번호가 있다면 추가
```

### 3단계: `@EnableRedisHttpSession` 어노테이션 추가

스프링 부트 애플리케이션에 레디스를 세션 저장소로 사용하겠다고 명시적으로 알려주는 어노테이션을 추가합니다. 보통 메인 애플리케이션 클래스에 붙입니다.

**[MyApplication.java]**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;

@EnableRedisHttpSession // <<< 이 어노테이션을 추가!
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

**참고**: 최신 스프링 부트 버전에서는 자동 설정(Auto-configuration)이 매우 강력해서, 의존성과 store-type: redis 설정만으로도 이 어노테이션 없이 동작하는 경우가 많습니다. 하지만 명시적으로 붙여주는 것이 의도를 드러내는 좋은 습관입니다.

### (선택) 로컬 테스트를 위한 Docker Compose 설정

로컬 PC에 Redis를 직접 설치하기보다, Docker Compose로 간단히 띄워서 사용하는 것이 편리합니다.

**[docker-compose.yml]**

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: my-redis
    ports:
      # 내 PC의 6379 포트와 컨테이너의 6379 포트를 연결
      - '6379:6379'
```

이제 docker-compose up 명령어로 레디스를 실행하고, 스프링 부트 애플리케이션을 localhost:6379를 바라보게 설정하면 됩니다.

이 설정이 끝나면, 사용자가 로그인할 때 생성되는 `HttpSession` 정보는 더 이상 서버 메모리가 아닌 레디스에 자동으로 저장됩니다. 이제 서버를 여러 대로 늘리거나 재시작해도 사용자의 로그인 상태는 안전하게 유지됩니다.

# 역할 제한 방법

특정 역할(Role)을 가진 사용자만 컨트롤러에 접근하도록 제한하는 것은 스프링 시큐리티의 핵심 '인가(Authorization)' 기능입니다. 두 가지 방법이 있으며, 메서드에 직접 어노테이션을 붙이는 `@PreAuthorize` 방식이 가장 많이 쓰이고 권장됩니다.

마치 클럽의 VIP 룸 앞에 "VIP만 출입 가능" 팻말을 붙이는 것과 같습니다. 팻말을 붙이는 두 가지 방법이 있는 셈이죠.

1. 메서드 보안 (`@PreAuthorize`): VIP 룸 문에 직접 팻말을 붙이는 방식 (가장 명확하고 유연함)
2. 설정 기반 (`SecurityFilterChain`): 클럽의 전체 출입 규칙서에 "VIP 룸은 VIP만 들어갈 수 있다"고 기록하는 방식

## 1. 메서드 보안: `@PreAuthorize` 사용 (강력 추천)

이 방식은 각 컨트롤러 메서드에 직접 필요한 권한을 명시하는 방법입니다. 코드가 직관적이고, 메서드별로 세밀한 제어가 가능합니다.

### 1단계: 메서드 보안 활성화

먼저, `SecurityConfig` 클래스에 `@EnableMethodSecurity` 어노테이션을 추가하여 이 기능을 활성화해야 합니다.

**[SecurityConfig.java]**

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity // <<< 이 어노테이션을 추가!
public class SecurityConfig {
    // ... SecurityFilterChain Bean 설정 ...
}
```

### 2단계: 컨트롤러 메서드에 어노테이션 추가

이제 보호하고 싶은 컨트롤러 메서드 위에 `@PreAuthorize`를 붙이고, SpEL(Spring Expression Language)을 사용하여 조건을 명시합니다.

**[AdminController.java]**

```java
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/admin")
public class AdminController {

    // 1. ADMIN 역할(Role)이 있어야만 접근 가능
    @GetMapping("/dashboard")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<String> getAdminDashboard() {
        return ResponseEntity.ok("관리자 대시보드 정보입니다.");
    }

    // 2. 여러 역할 중 하나라도 있으면 접근 가능
    @GetMapping("/notice")
    @PreAuthorize("hasAnyRole('ADMIN', 'MANAGER')")
    public ResponseEntity<String> getAdminNotice() {
        return ResponseEntity.ok("관리자 또는 매니저만 볼 수 있는 공지사항입니다.");
    }

    // 3. 인증된 사용자라면 누구나 접근 가능 (역할 무관)
    @GetMapping("/profile")
    @PreAuthorize("isAuthenticated()")
    public ResponseEntity<String> getAdminProfile() {
        return ResponseEntity.ok("인증된 관리자/매니저의 프로필 정보입니다.");
    }
}
```

- `hasRole('ROLE_NAME')`: 특정 역할을 가졌는지 확인합니다.
  - **주의**: 스프링 시큐리티는 기본적으로 역할 이름 앞에 `ROLE_` 접두사를 붙여서 비교합니다. 따라서 DB에 `ADMIN`으로 저장되어 있다면, `ROLE_ADMIN` 권한을 가지고 있는지 확인해야 합니다.
- `hasAnyRole('ROLE_1', 'ROLE_2')`: 여러 역할 중 하나라도 만족하는지 확인합니다.
- `isAuthenticated()`: 인증된 사용자인지 (로그인했는지) 여부만 확인합니다.

## 2. 설정 기반: `SecurityFilterChain`에서 URL 패턴으로 제어

`SecurityConfig` 파일에서 URL 패턴별로 접근 규칙을 중앙에서 관리하는 방식입니다.

**[SecurityConfig.java]**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
// ...
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // ... (csrf, cors, httpBasic, formLogin 등 다른 설정) ...
            .authorizeHttpRequests(auth -> auth
                // /api/admin/** 패턴의 모든 요청은 "ADMIN" 역할을 가진 사용자만 허용
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                // /api/manager/** 패턴은 "MANAGER" 또는 "ADMIN" 역할을 가진 사용자만 허용
                .requestMatchers("/api/manager/**").hasAnyRole("MANAGER", "ADMIN")
                // /api/users/me 패턴은 인증만 되면 누구나 허용
                .requestMatchers("/api/users/me").authenticated()
                // 나머지 모든 요청은 허용
                .anyRequest().permitAll()
            );
        return http.build();
    }
}
```

이 방식은 특정 그룹의 URL에 대한 규칙을 한 곳에서 관리하기 편하지만, 각 컨트롤러 메서드가 어떤 권한을 필요로 하는지 파악하려면 `SecurityConfig` 파일을 항상 확인해야 하는 번거로움이 있습니다.

결론적으로, 코드의 가독성과 유지보수 측면에서 `@PreAuthorize`를 사용하는 것이 훨씬 더 현대적이고 효율적인 방법입니다.
