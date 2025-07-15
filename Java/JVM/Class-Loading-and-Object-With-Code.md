# 코드와 함께 보는 클래스 로딩 & 객체 생성

- [코드와 함께 보는 클래스 로딩 \& 객체 생성](#코드와-함께-보는-클래스-로딩--객체-생성)
  - [예제 1](#예제-1)
  - [예제 2](#예제-2)

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

## 예제 2

```java
class Animal {
    private static String species = "Mammal";
    private static int totalCount = 0;

    private String name;
    private int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
        totalCount++;
        System.out.println("Animal 생성자 호출: " + name);
    }
}

class Dog extends Animal {
    private static String dogType = "Domestic";

    private String breed;

    public Dog(String name, int age, String breed) {
        super(name, age);
        this.breed = breed;
        System.out.println("Dog 생성자 호출: " + breed);
    }
}

public class Main {
    public static void main(String[] args) {
        Animal animal = new Animal("동물", 5);
        Dog dog = new Dog("멍멍이", 3, "골든리트리버");
    }
}
```

각 과정의 명칭이나 설명은 제외하고 설명해보겠습니다. 그리고 `Main` 클래스의 로딩은 끝난 상태이고 `main` 메서드가 실행되었다고 가정하겠습니다.

1. `Animal` 클래스가 로딩되어 있는지 확인
2. 없다면 `Animal` 클래스 로딩 과정 진행
   1. 클래스 메타데이터를 메서드 영역에 저장
   2. 정적 변수 기본값으로 초기화: `species: null`, `totalCount: 0`
   3. 정적 변수 명시된 값으로 초기화: `species: "Mammal"`, `totalCount: 0`
3. 스택에 지역 변수 `animal` 저장
4. `new Animal("동물", 5)` 실행
   1. 힙에 `Animal` 객체 저장
   2. 지역 변수 `animal`에 `Animal` 객체 주소값 할당
   3. 멤버 변수 기본값으로 초기화: `name: null`, `age: 0`
   4. 생성자 실행
   5. 멤버 변수 매개변수값으로 초기화: `name: "동물"`, `age: 5`
   6. `totalCount` + 1 연산
   7. `Animal 생성자 호출: 동물` 출력
5. `Dog` 클래스는 `Animal` 클래스를 상속하였으므로 부모 클래스인 `Animal` 클래스가 로딩되어 있는지 확인
6. `Animal` 클래스는 로딩되어 있으므로 `Dog` 클래스 로딩되어 있는지 확인.
7. `Dog` 클래스는 로딩되어 있지 않으므로 로딩 과정 진행
   1. 클래스 메타데이터를 메서드 영역에 저장
   2. 정적 변수 기본값으로 초기화: `dogType: null`
   3. 정적 변수 명시된 값으로 초기화: `dogType: "Domestic"`
8. 스택에 지역 변수 `dog` 저장
9. `new Dog("멍멍이", 3, "골든리트리버")` 실행
   1. 힙에 `Dog` 객체 저장
   2. 지역 변수 `dog`에 `Dog` 객체 주소값 할당
   3. 부모 멤버 변수 기본값으로 초기화: `name: null`, `age: 0`
   4. 멤버 변수 기본값으로 초기화: `breed: null`
   5. `Animal` 생성자 실행 (super 호출)
   6. Animal 멤버 변수 매개변수값으로 초기화: `name: "멍멍이"`, `age: 3`
   7. `totalCount` + 1 연산
   8. `Animal 생성자 호출: 멍멍이` 출력
   9. `Dog` 생성자 실행
   10. 멤버 변수 매개변수값으로 초기화: `breed: "골든리트리버"`
   11. `Dog 생성자 호출: 골든리트리버` 출력
10. 최종 메모리 상태
    1. 메서드 영역: `Animal` 클래스의 메타데이터/정적변수, `Dog` 클래스의 메타데이터/정적변수
    2. 힙 영역: `Animal` 객체, `Dog` 객체, 문자열: `"Mammal", "동물", "Animal 생성자 호출: 동물", "Domestic", "멍멍이", "Animal 생성자 호출: 멍멍이", "골든리트리버", "Dog 생성자 호출: 골든리트리버"`
    3. 스택 영역: 지역 변수 `animal`, `dog`
