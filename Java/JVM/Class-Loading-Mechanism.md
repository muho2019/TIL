# 클래스 로딩 메커니즘(Class Loading Mechanism)

JVM은 클래스를 설명하는 데이터를 클래스 파일로부터 메모리로 읽어 들이고 그 데이터를 검증, 변환, 초기화하고 나서 최종적으로 가상 머신이 곧바로 사용할 수 있는 자바 타입을 생성합니다. 이 과정을 가상 머신의 **클래스 로딩 메커니즘**이라고 합니다.

- [클래스 로딩 메커니즘(Class Loading Mechanism)](#클래스-로딩-메커니즘class-loading-mechanism)
  - [1. 클래스 로딩 시점](#1-클래스-로딩-시점)
  - [2. 클래스 로딩 처리 과정](#2-클래스-로딩-처리-과정)
    - [2.1 로딩](#21-로딩)
    - [2.2 검증](#22-검증)
    - [2.3 준비](#23-준비)
    - [2.4 해석](#24-해석)
    - [2.5 초기화](#25-초기화)
      - [`<clinit>()` 메서드란?](#clinit-메서드란)
      - [초기화가 발생하는 조건](#초기화가-발생하는-조건)
      - [초기화의 동시성 보장](#초기화의-동시성-보장)

## 1. 클래스 로딩 시점

타입의 생애 주기는 다음과 같습니다. 각 단계는 이전 단계가 완전히 끝난 후가 아니라, 진행 중인 과정에 다음 단계가 '시작'될 수 있음을 이해하는 것이 중요합니다.

1. **로딩(Loading)**
2. **링킹(Linking)**
   1. **검증(Verification)**
   2. **준비(Preparation)**
   3. **해석(Resolution)**
3. **초기화(Initialization)**
4. **사용(Using)**
5. **언로딩(Unloading)**

이 순서 중 **로딩**, **검증**, **준비**, **초기화**, **언로딩** 단계의 시작 순서는 반드시 지켜져야 합니다.

하지만 **해석** 단계는 때에 따라서 **초기화** 단계 이후에 시작될 수 있습니다. 이는 자바 언어의 **런타임 바인딩**(**동적 바인딩** 또는 **늦은 바인딩**)을 지원하기 위함입니다.

## 2. 클래스 로딩 처리 과정

### 2.1 로딩

JVM은 로딩 단계에서 다음 세 가지 작업을 수행합니다.

1. 클래스의 **정규화된 이름**(**Fully Qualified Name**)을 통해 해당 클래스를 정의하는 **바이너리 바이트 스트림**을 가져옵니다.
2. 바이트 스트림이 나타내는 정적 저장 구조를 **메서드 영역**에서 사용하는 **런타임 데이터 구조**로 변환합니다.
3. 로딩된 클래스를 표현하는 `java.lang.Class` 객체를 **힙 메모리**에 생성합니다. 이 `Class` 객체는 애플리케이션이 메서드 영역에 저장된 타입 데이터를 활용할 수 있게 하는 **통로(entry point)** 역할을 합니다.

JVM 명세는 바이너리 바이트 스트림을 어디서, 어떻게 가져올지에 대해 구체적으로 명시하지 않았습니다. 이러한 유연성 덕분에 다양한 소스로부터 클래스를 로딩하는 기술이 발전할 수 있었습니다.

- ZIP/JAR/WAR 파일로부터 로딩
- 네트워크로부터 로딩 (e.g. 웹 애플릿)
- 런타임에 동적으로 생성 (e.g. 동적 프록시)
- 다른 파일로부터 생성 (e.g. JSP 파일 -> 클래스 파일)
- 데이터베이스로부터 로딩
- 암호화된 파일을 해독하여 로딩

로딩 단계, 특히 '바이너리 바이트 스트림을 얻는 동작'은 개발자가 커스텀 **클래스 로더**(`ClassLoader`)를 만들어 가장 쉽게 제어할 수 있는 부분입니다. `ClassLoader`의 `findClass()` 또는 `loadClass()` 메서드를 오버라이딩하여 원하는 로직을 구현할 수 있습니다.

단, 배열 클래스는 다릅니다. 배열 클래스는 클래스 로더가 아닌 JVM이 직접 메모리에 동적으로 생성합니다. 하지만 배열을 구성하는 원소 타입(Element Type. 배열에서 모든 차원을 제거한 타입)은 클래스 로더를 통해 로딩되므로 클래스 로더와 완전히 무관하지 않습니다.

- 컴포넌트 타입(Component Type. 배열에서 첫 번째 차원이 제거된 타입)이 참조 타입일 경우 (e.g. `int[][]`): 재귀적으로 로딩 과정을 수행하여 컴포넌트 타입을 먼저 로딩합니다. 배열 클래스는 컴포넌트 타입을 로드하는 클래스 로더의 이름 공간(namespace)에 자리하게 됩니다.
- 컴포넌트 타입이 기본 타입일 경우 (e.g. `int[]`): JVM은 해당 배열 클래스를 부트스트랩 클래스 로더(Bootstrap ClassLoader)에 배치합니다.

로딩 단계가 끝나면 클래스의 정보는 메서드 영역에 저장되고 이 정보에 접근할 수 있는 `java.lang.Class` 객체는 힙에 생성됩니다.

### 2.2 검증

검증은 링킹 과정의 첫 번째 단계이며 클래스 파일 바이트 스트림이 JVM 명세에 맞는지 그리고 실행 시 JVM의 보안을 해치지 않는지 확인하는 과정입니다.

클래스 파일은 반드시 Java 컴파일러를 통해 생성되지 않을 수도 있으므로 이 검증 단계는 악의적이거나 잘못된 바이트코드로부터 JVM을 보호하는 필수적인 안전장치입니다.

검증은 보통 다음 네 단계의 검사를 통해 이루어집니다.

1. **파일 형식 검증 (File Format Verification)**

바이트 스트림이 기본적인 클래스 파일 형식에 부합하는지 확인합니다.

- 예시
  - 매직 넘버가 `0xCAFEBABE`로 시작하는가?
  - 파일에 명시된 주/부 버전 번호를 현재 JVM에서 처리할 수 있는가?
  - 상수 풀의 각 태그가 명세에 정의된 형식에 맞는가?

2. **메타데이터 검증 (Metadata Verification)**

바이트코드가 담고 있는 클래스의 정보(메타데이터)가 Java 언어 명세에 부합하는지 의미론적으로 분석합니다.

- 예시
  - `final`로 선언된 클래스를 다른 클래스가 상속하고 있지는 않은가?
  - 추상 클래스가 아닌데 구현되지 않은 메서드가 존재하는가?
  - 부모 클래스에 존재하지 않는 메서드를 오버라이딩하고 있지는 않은가?

3. **바이트코드 검증 (Bytecode Verification)**

검증 과정 중 가장 복잡한 단계로, 메서드 본문의 바이트코드가 논리적으로 유효하고 안전한지 데이터 흐름과 제어 흐름을 분석합니다.

- 예시
  - 메서드 스택의 데이터 타입과 명령어의 요구 타입이 일치하는가? (e.g. `int`를 기대하는 명령어에 객체 참조를 사용하지 않는가?)
  - 메서드 실행 중 스택이 언더플로우 또는 오버플로우 되지 않는가?
  - 접근할 수 없는 `private` 멤버에 접근하는 코드는 없는가?

4. **심벌 참조 검증 (Symbolic Reference Verification)**

**해석(Resolution)** 단계에서 심벌 참조를 직접 참조로 변환할 때 발생하며 참조하는 대상이 실제로 존재하고 접근 가능한지 확인합니다.

- 예시
  - 심벌 참조를 통해 특정 클래스, 필드, 메서드를 찾을 수 있는가?
  - 참조하는 대상에 접근할 수 있는 권한(e.g. `private`, `protected`)을 가지고 있는가?

검증 단계는 매우 중요하지만 모든 코드를 신뢰할 수 있는 환경에서는 `-Xverifiy:none` 매개변수를 통해 이 단계를 건너뛸 수 있습니다. 이 경우 클래스 로딩 시간을 단축하는 효과를 볼 수 있습니다.

### 2.3 준비

준비 단계는 클래스의 **정적 변수**(static variables)를 위해 메모리를 할당하고 해당 데이터 타입의 **제로값**(zero-value)으로 **초기화하는 단계**입니다.

- **대상**: **클래스 변수**(`static`)만이 대상입니다. 인스턴스 변수는 객체가 생성될 때 힙 영역에 할당됩니다.
- **메모리 영역**: 정적 변수들을 위한 메모리는 **메서드 영역**에 할당됩니다.
- **초깃값**: 명시적으로 선언한 값이 아닌 각 데이터 타입의 **제로값**(`0`, `null`, `false` 등)이 할당됩니다.
  - 예시: `public static in value = 123;`
  - 준비 단계에서 `value`는 0으로 초기화됩니다.
  - 실제 값 `123`을 할당하는 코드(`putstatic` 명령어)는 이후 **초기화 단계**에서 클래스 생성자인 `<clinit>()` 메서드가 실행될 때 동작합니다.

일반적인 경우와 달리, 정적 변수가 `final` 키워드와 함께 선언되고 컴파일 시점에 값이 결정되는 상수라면 준비 단계에서부터 제로값이 아닌 명시된 값으로 초기화됩니다.

- 조건: 클래스 필드의 속성 테이블에 `ConstantValue` 속성이 존재할 경우
- 예시: `public static final int value = 123;`
- 컴파일러는 `value` 필드에 `123`이라는 값을 가진 `ConstantValue` 속성을 추가합니다.
- JVM은 준비 단계에서 이 속성을 확인하고 `value` 변수에 `123`을 직접 할당합니다.

### 2.4 해석

해석은 링킹의 마지막 단계로 상수 풀(Constant Pool) 내의 **심벌 참조**(Symbolic Reference)를 실제 메모리 주소인 **직접 참조**(Direct Reference)로 교체하는 과정입니다.

- **심벌 참조**: 클래스, 필드, 메서드 등을 이름, 타입과 같은 **리터럴**(literal)로 식별하는 방식입니다. JVM의 메모리 구조와는 무관합니다.
- **직접 참조**: 실제 메모리상의 **주소**(포인터), **오프셋**, 또는 **이를 가리키는 핸들** 등입니다. 이 주소를 통해 대상에 직접 접근할 수 있습니다.

JVM 명세는 해석이 언제 일어나야 하는지 구체적으로 강제하지 않으므로 JVM 구현체는 로딩 직후에 미리 처리하거나(**Eager Resolution**) 해당 참조를 처음 사용할 때 처리(**Lazy Resolution**)할 수 있습니다.

해석은 주로 다음과 같은 참조들에 대해 이루어집니다.

1. **클래스 또는 인터페이스 해석**: 특정 클래스나 인터페이스를 가리키는 심벌 참조를 `java.lang.Class` 객체를 가리키는 직접 참조로 변환합니다. 이 과정에서 필요하다면 대상 클래스의 로딩이 트리거될 수 있습니다.
2. **필드 해석**: 특정 필드를 가리키는 심벌 참조를, 해당 필드의 메모리상 오프셋을 가리키는 직접 참조로 변환합니다.
3. **메서드 해석**: 특정 메서드를 가리키는 심벌 참조를, 해당 메서드의 실제 메모리 주소를 가리키는 직접 참조로 변환합니다.
4. **인터페이스 메서드 해석**: 인터페이스의 특정 메서드를 가리키는 심벌 참조를 직접 참조로 변환합니다.

이 과정에서 JVM은 해당 필드나 메서드에 접근할 수 있는 권한(`private`, `public` 등)이 있는지도 함께 확인합니다.

동일한 심벌 참조에 대한 해석 요청은 여러 번 발생할 수 있습니다. 따라서 대부분의 JVM은 **첫 해석 결과를 캐시**(Cache)하여 이후에는 빠르게 직접 참조를 사용합니다.

하지만 `invokedynamic` 명령어는 예외입니다. 이 명령어의 목적은 동적 언어(dynamic language)를 지원하는 것으로, 호출될 때마다 다른 결과를 반환할 수 있습니다. 따라서 `invokedynamic`이 가리키는 대상은 '동적으로 계산된 호출 사이트 지정자(dynamically computed call site specifier)'라고 부릅니다. 이 명령어는 프로그램이 **실행되는 시점**에 비로소 해석이 이루어지기 때문에 '동적'이라고 하며 나머지 명령어들은 상대적으로 '정적'이라고 할 수 있습니다.

### 2.5 초기화

초기화는 **클래스 로딩 절차의 마지막 단계**로, 이 단계에 들어서면 JVM이 드디어 사용자가 작성한 **Java 코드를 실행하기 시작**합니다.

준비 단계에서 정적 변수들이 제로값으로 할당된 것과 달리 초기화 단계에서는 클래스 변수와 기타 자원들이 개발자가 의도한 값으로 채워집니다. 더 직관적으로 말하면, 초기화 단계란 클래스의 정적 초기화 메서드인 `<clinit>()` 메서드를 실행하는 단계입니다.

#### `<clinit>()` 메서드란?

`<clinit>()` 메서드는 개발자가 직접 작성하는 것이 아니라, Java 컴파일러가 자동으로 생성하는 특별한 메서드입니다. 컴파일러는 클래스 내의 모든 정적 변수 할당문과 static {} 블록을 소스 코드에 나타난 순서대로 모아서 하나의 `<clinit>()` 메서드로 만들어 냅니다.

#### 초기화가 발생하는 조건

JVM 명세는 클래스의 초기화가 반드시 일어나야 하는 경우를 다음과 같이 엄격하게 규정하고 있습니다. 이 조건에 해당할 때 최초 한 번 `<clinit>()` 메서드가 호출됩니다.

1. `new` 키워드를 통해 클래스의 인스턴스를 생성할 때
2. 클래스에 선언된 정적 메서드를 호출할 때
3. 클래스에 선언된 정적 필드를 사용하거나 값을 할당할 때 (단, static final 상수는 예외)
4. 리플렉션(`Class.forName()` 등)을 사용하여 클래스를 동적으로 로딩할 때
5. 하위 클래스가 초기화될 때 (이 경우 그 부모 클래스가 먼저 초기화되어야 함)
6. JVM 시작 시 지정된 메인 클래스가 실행될 때

#### 초기화의 동시성 보장

JVM은 멀티스레드 환경에서도 클래스의 `<clinit>()` 메서드가 **단 한 번만 실행되는 것을 보장**합니다. 여러 스레드가 동시에 특정 클래스의 초기화를 시도하더라도 오직 하나의 스레드만 `<clinit>()` 메서드를 실행할 수 있고 나머지 스레드들은 실행이 끝날 때까지 대기해야 합니다. 이는 클래스 초기화 과정에서의 동시성 문제를 원천적으로 방지해 줍니다.
