# 인터페이스 작성 방법

## 1. 기본 (The Basics): `JpaRepository` 상속

가장 기본은 `JpaRepository` 인터페이스를 상속받는 것부터 시작합니다. 이것만으로도 기본적인 데이터 처리(CRUD) 기능 대부분이 자동으로 구현됩니다.

- `JpaRepository<T, ID>`
  - `T`: 이 Repository가 다룰 엔티티(Entity) 클래스를 지정합니다.
  - `ID`: 해당 엔티티의 Primary Key(PK) 필드 타입을 지정합니다.

**[MemberRepository.java]**

```java
import org.springframework.data.jpa.repository.JpaRepository;

// Member 엔티티를 다루고, Member의 PK 타입은 Long 이다.
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

이 인터페이스를 만들기만 하면, 우리는 아래의 핵심 메서드들을 구현 없이 바로 주입받아 사용할 수 있습니다.

- `save(S entity)`: 새로운 엔티티를 저장하거나, 기존 엔티티를 수정 (INSERT, UPDATE)
- `findById(ID id)`: PK로 엔티티 하나를 조회. `Optional<T>` 반환 (SELECT)
- `findAll()`: 모든 엔티티를 조회 (SELECT)
- `count()`: 엔티티의 총 개수를 조회 (SELECT)
- `delete(T entity)`: 특정 엔티티를 삭제 (DELETE)
- `deleteById(ID id)`: PK로 엔티티를 삭제 (DELETE)

## 2. 중급 (Intermediate): 쿼리 메소드 (Query Methods)

Spring Data JPA의 가장 강력한 기능입니다. 정해진 규칙에 따라 메서드 이름을 짓기만 하면, 스프링이 그 이름을 분석해서 JPQL 쿼리를 자동으로 생성하고 실행해 줍니다.

- 주요 키워드:
  - FindBy...: 조회 쿼리임을 나타내는 접두사 (생략 가능)
  - And, Or: 여러 조건을 조합
  - LessThan, GreaterThan, Between: 숫자나 날짜 범위 조건
  - Like, Containing, StartingWith, EndingWith: 문자열 패턴 조건
  - IsNull, IsNotNull: Null 체크
  - OrderBy...Asc, OrderBy...Desc: 정렬 조건

**[MemberRepository.java 에 추가]**

```java
import java.util.List;
import java.util.Optional;

public interface MemberRepository extends JpaRepository<Member, Long> {

    // 1. 이메일(email)로 회원 찾기
    Optional<Member> findByEmail(String email);

    // 2. 닉네임(nickname)에 특정 문자열을 포함(Containing)하는 회원 목록 찾기
    List<Member> findByNicknameContaining(String keyword);

    // 3. 특정 나이(age)보다 많고(GreaterThan), 이름을 기준으로 내림차순(OrderByNameDesc) 정렬
    List<Member> findByAgeGreaterThanOrderByNameDesc(int age);

    // 4. 이메일과 이름을 모두 만족(And)하는 회원이 존재하는지(Exists) 확인
    boolean existsByEmailAndName(String email, String name);
}
```

## 3. 심화 (Advanced): 더 복잡한 쿼리 다루기

메서드 이름만으로 표현하기 어렵거나, 더 정교한 제어가 필요할 때 사용하는 고급 기술들입니다.

### 직접 쿼리 작성: `@Query`

메서드 이름이 너무 길어지거나 복잡한 로직(JOIN, 서브쿼리 등)이 필요할 때, JPQL(Java Persistence Query Language)을 직접 작성할 수 있습니다.

```java
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

// ...
    // JPQL을 사용하여 직접 쿼리 작성
    // :name은 파라미터 바인딩을 의미
    @Query("SELECT m FROM Member m WHERE m.name = :name AND m.age > :age")
    List<Member> findMembersByNameAndAge(@Param("name") String name, @Param("age") int age);
```

### 데이터 수정: `@Modifying`

`@Query`는 기본적으로 조회(SELECT)만 가능합니다. 데이터를 수정(UPDATE)하거나 삭제(DELETE)하는 쿼리를 실행하려면 `@Modifying` 어노테이션을 함께 사용해야 합니다. 수정 쿼리는 보통 트랜잭션 안에서 실행되어야 하므로 서비스 계층에서 `@Transactional`을 붙여줘야 합니다.

```java
import org.springframework.data.jpa.repository.Modifying;
// ...
    @Modifying
    @Query("UPDATE Member m SET m.status = 'INACTIVE' WHERE m.lastLoginAt < :date")
    int updateInactiveMembers(@Param("date") LocalDateTime date);
```

### 페이징과 정렬: `Pageable` & `Sort`

대량의 데이터를 조회할 때, 페이징 처리는 필수입니다. `Pageable` 인터페이스를 파라미터로 넘기기만 하면, 스프링이 알아서 페이징 쿼리(LIMIT, OFFSET)를 적용해 줍니다.

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

// ...
    // 이름에 키워드를 포함하는 회원을 페이징 처리하여 조회
    // Pageable 객체에는 페이지 번호, 페이지 크기, 정렬 정보가 담겨있음
    Page<Member> findByNameContaining(String name, Pageable pageable);
```

- 반환 타입 `Page<Member>`: 조회된 데이터 목록 외에도, 전체 데이터 수, 전체 페이지 수, 현재 페이지 번호 등 페이징 처리에 필요한 모든 정보를 담고 있어 매우 편리합니다.

### 필요한 데이터만 골라내기: 프로젝션 (Projections)

엔티티의 모든 필드가 아닌, 몇 개의 특정 필드만 조회하고 싶을 때 사용합니다. 전체 데이터를 가져오는 것보다 성능상 이점이 큽니다.

**1. 조회할 필드만 담는 인터페이스를 정의합니다.**

```java
// MemberSummary.java (인터페이스 기반 프로젝션)
public interface MemberSummary {
    String getName();
    String getEmail();
}
```

**2. Repository 메서드의 반환 타입을 이 인터페이스로 지정합니다.**

```java
// MemberRepository.java
List<MemberSummary> findByStatus(MemberStatus status);
```

이렇게 하면, 스프링 데이터 JPA가 자동으로 `name`과 `email` 필드만 조회하는 쿼리를 생성하여 `MemberSummary` 객체 목록으로 반환해 줍니다.
