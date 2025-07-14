# 코드와 함께 보는 클래스 로딩 & 객체 생성

- [코드와 함께 보는 클래스 로딩 \& 객체 생성](#코드와-함께-보는-클래스-로딩--객체-생성)
  - [예제 1](#예제-1)

## 예제 1

```java
class Person {
    private static String company = "ABC Company";
    private String name = "홍길동";
    private int age = 25;

    public Person() {
        System.out.println("Person 생성자 호출");
    }
}
```

메인 메서드에서 `Person person = new Person();` 코드가 실행된다면 어떻게 될까요?

1. Person 클래스가 메서드 영역에 로딩되어 있는지 확인을 합니다. 로딩되어 있지 않다면 클래스 로딩 과정을 진행합니다.
2. 클래스 로딩 과정
   1. 로딩 단계: `Person.class` 파일로부터 클래스의 메타데이터를 가져와 메서드 영역에 저장합니다.
   2. 링킹 단계
      1. 검증: 바이트코드 유효성 검사
      2. 준비: 정적 변수를 기본값으로 초기화
         1. `company`의 데이터 타입은 `String`이므로 `null`로 초기화
      3. 해석: 심볼릭 참조를 실제 참조로 변환
   3. 초기화 단계
      1. `company` 변수에 문자열 "ABC Company" 문자열 객체의 참조값 할당
      2. 클래스 로딩이 완료된 상태의 메모리 상태
         1. 메서드 영역: Person 클래스의 메타데이터와 정적 변수가 위치
         2. 힙 영역: "ABC Company" 문자열 객체
3. 객체 생성 과정
   1. 힙 영역에 Person 객체를 위한 메모리 공간 할당
   2. 인스턴스 필드 초기화
      1. 기본값으로 초기화: `name`은 `null`, `age`는 0
      2. 그 다음 명시적 값으로 초기화: `name`은 `"홍길동"`, `age`는 `25`
   3. 생성자 호출
      1. `Person()` 생성자 실행
      2. `"Person 생성자 호출"` 출력
4. 최종 메모리 상태
   1. 메서드 영역: Person 클래스 메타데이터, 정적 변수 `company`
   2. 힙 영역: Person 객체, "ABC Company", "홍길동", "Person 생성자 호출"
   3. 스택 영역: 지역변수 `person`(힙의 객체 참조값 저장)

**`Person` 클래스 메타데이터**

- 클래스 기본 정보
  - 클래스 이름: Person
  - 접근 제어자: default
  - 클래스 타입: 일반 클래스
  - 부모 클래스 정보: `java.lang.Object`(모든 클래스의 기본 부모)
- 상수 풀
  - 문자열 리터럴 "ABC Company", "홍길동", "Person 생성자 호출"
  - 클래스 참조: String, System, PrintStream
  - 메서드 참조: System.out.println
- 필드 정보
  - 정적 필드: company(String 타입, private)
  - 인스턴스 필드: name(String 타입, private), age(int 타입, private)
- 메서드 정보: 기본 생성자
