# 클래식 가비지 컬렉터

클래식 가비지 컬렉터는 JDK 7부터 JDK 11까지 오라클 JDK의 핫스팟 가상 머신(HotSpot Virtual Machine)에 포함된 모든 가비지 컬렉터를 포함합니다. 이들은 JVM의 메모리 관리를 담당하는 전통적인 가비지 컬렉션 알고리즘들입니다.

**클래식 가비지 컬렉터 종류**

- 시리얼 (Serial GC)
- 시리얼 올드 (Serial Old GC)
- 파뉴 (ParNew GC)
- CMS (Concurrent Mark Sweep)
- 패러렐 스캐빈지 (Parallel Scavenge GC)
- 패러렐 올드 (Parallel Old GC)
- G1 (Garbage First GC)

## CMS 컬렉터

CMS 컬렉터는 표시(Mark)와 쓸기(Sweep) 단계 모두를 사용자 스레드와 동시에(Concurrent) 수행합니다. CMS 컬렉터의 주요 목적은 가비지 컬렉션에 따른 일시 정지 시간(Stop-The-World pause time)을 최소화하는 것입니다.

### 알고리즘 구조

CMS 컬렉터는 마크-스윕 알고리즘(Mark-Sweep Algorithm)을 기초로 구현되었습니다. 전체 과정은 다음과 같은 네 단계로 구성됩니다.

1. **최초 표시 (Initial Mark)**

- **STW**(Stop-The-World) 발생
- **GC Roots**로부터 직접 참조되는 객체들을 표시
- 매우 짧은 시간 동안 수행

2. **동시 표시 (Concurrent Mark)**

- 사용자 스레드와 동시에 수행
- 최초 표시 단계에서 표시된 객체들로부터 참조 관계를 추적하여 살아있는 객체들을 표시
- 가장 시간이 오래 걸리는 단계

3. **재표시 (Remark)**

- STW 발생
- 동시 표시 단계에서 놓친 객체들을 다시 표시
- 삼색 표시(Tri-color Marking) 과정에서 발생한 누락을 수정

4. **동시 쓸기 (Concurrent Sweep)**

- 사용자 스레드와 동시에 수행
- 표시되지 않은 객체들을 메모리에서 제거
- 메모리 압축(Memory Compaction)은 수행하지 않음

### 주요 단점

1. **프로세서 자원에 대한 민감성**

- 동시 수행 단계에서 사용자 스레드를 멈추지는 않지만, 애플리케이션 성능을 저하시키고 전체 처리량(Throughput)을 떨어뜨리는 것은 피할 수 없습니다.
- CPU 집약적인 애플리케이션에서는 성능 저하가 더욱 두드러집니다.

2. **부유 쓰레기 (Floating Garbage) 문제**

- 사용자 스레드와 동시에 진행하다 보면 새로운 쓰레기가 자연스럽게 생성됩니다.
- 표시 스레드가 지나간 후에 쓰레기가 된 객체들은 다음 GC 사이클까지 정리되지 않습니다.
- 이로 인해 동시 모드 실패(Concurrent Mode Failure)가 발생할 가능성이 있습니다.

3. **메모리 단편화 (Memory Fragmentation)**

- CMS는 마크-스윕 알고리즘을 사용하기 때문에 메모리 단편화가 심각합니다.
- 압축(Compaction) 과정이 없어 메모리가 조각조각 나뉩니다.
- 단편화가 심하면 큰 객체를 할당할 때 특히 문제가 됩니다.

### 지원 종료

CMS 컬렉터는 **JDK 9**에서 **deprecated**되었고, **JDK 14**에서 **완전히 제거**되었습니다. 현재는 **G1 GC**나 **ZGC**, **Shenandoah** 등의 최신 가비지 컬렉터로 대체되고 있습니다.

## G1 컬렉터

### 기본 개념

G1 컬렉터는 **Garbage First**(가비지 우선)를 줄인 표현으로, CMS의 대체자이자 후계자를 목표로 설계되었습니다. G1은 **부분 회수**(Partial Collection)라는 컬렉터 설계 아이디어와 **리전**(Region)을 회수 단위로 하는 메모리 레이아웃 분야를 개척했습니다.

### 혁신적인 설계 특징

1. **정지 시간 예측 모델 (Pause Prediction Model)**

G1의 설계자들은 정지 시간 예측 모델을 구현하여 사용자가 원하는 정지 시간 내에서 최적의 가비지 컬렉션을 수행할 수 있도록 했습니다.

2. **회수 집합 (Collection Set, CSet)**

기존 컬렉터들은 신세대 전체(마이너 GC), 구세대 전체(메이저 GC), 또는 자바 힙 전체(전체 GC)를 회수 범위로 했습니다. 반면 G1은 힙 메모리의 어느 곳이든 회수 대상에 포함할 수 있습니다. 이를 **회수 집합**(Collection Set, **CSet**)이라 하며, 어느 세대에 속하느냐가 아니라 다음 기준으로 회수 영역을 선택합니다.

- 어느 영역에 쓰레기가 가장 많은가
- 회수했을 때 이득이 어디가 가장 큰가

이것이 G1의 **혼합 GC 모드**(Mixed GC Mode)입니다.

3. **리전 기반 힙 메모리 레이아웃**

G1은 크기와 수가 고정된 세대 단위 영역 구분에서 벗어나, 연속된 자바 힙을 동일 크기의 여러 독립 리전으로 나눕니다. 각 리전은 필요에 따라 다음과 같이 역할을 바꿀 수 있습니다.

- 신세대의 **에덴**(Eden) 공간
- **생존자**(Survivor) 공간
- **구세대**(Old Generation) 공간
- **거대 리전**(Humongous Region): 리전 용량의 절반보다 큰 객체를 저장

### 핵심 동작 원리

G1은 각 리전의 **쓰레기 누적값**을 추적합니다. 여기서 값이란 가비지 컬렉션으로 회수할 수 있는 공간의 크기와 회수에 드는 시간의 경험값입니다. 그리고 **우선순위 목록**을 관리하며 허용하는 한도 내에서 회수 효과가 가장 큰 리전부터 회수합니다. 이것이 '가비지 우선'이라는 이름이 탄생한 이유입니다.

### 주요 해결 과제

1. **리전 간 참조 문제 해결**

- **기억 집합**(Remembered Set)을 도입하여 GC 루트부터 힙 전체를 스캔하는 일을 방지
- 모든 리전이 각자의 기억 집합을 관리하며, 다른 리전으로부터의 모든 참조 정보를 기록
- G1의 기억 집합은 기본적으로 **해시 테이블** 구조로 구현

2. **동시 표시 중 스레드 간섭 방지**

- **시작 단계 스냅숏 알고리즘**(Snapshot-At-The-Beginning, **SATB**) 채택 (CMS의 증분 업데이트와 대비)
- 각 리전을 위해 **TAMS**(Top at Mark Start) 포인터 설계
- 동시 회수 동안 새로 생성되는 객체는 TAMS 포인터보다 높은 주소에 할당
- 이 주소보다 높이 있는 객체는 암묵적으로 표시된 것으로 간주하여 회수 대상에서 제외

3. **정지 시간 예측 모델 구현**

- 이론적 기초: **감소 평균**(Decaying Average)
- 리전별 회수 시간, 기억 집합의 더려워진 카드 개수 등을 기록
- 평균, 표준 편차, 신뢰도 등의 통계 분석
- 감소 평균은 새로운 데이터에 더 민감하여 '최근'의 평균적인 상태를 더 정확하게 반영

### G1 동작 과정 (4단계)

1. **최초 표시 (Initial Mark)**

- **STW** 발생
- **GC 루트**가 직접 참조하는 객체들을 표시
- **TAMS 포인터** 값을 수정하여 **시작 단계 스냅숏** 생성
- **마이너 GC**와 동시에 수행되어 추가 정지 시간이 거의 없음

2. **동시 표시 (Concurrent Mark)**

- 사용자 스레드와 동시에 수행
- GC 루트로부터 객체들의 **도달 가능성**(Reachability) 분석
- 전체 힙의 객체 그래프를 재귀적으로 스캔
- **시작 단계 스냅숏**과 비교하여 참조가 변경된 객체들을 재스캔

3. **재표시 (Remark)**

- **STW** 발생
- **시작 단계 스냅숏** 이후 변경된 소수의 객체만 처리
- 매우 빠르게 완료

4. **복사 및 청소 (Copy & Clean)**

- **STW** 발생
- 통계 데이터를 기초로 리전들을 회수 가치와 비용에 따라 정렬
- 목표 정지 시간에 맞춰 회수 계획 수립
- 선별된 리전의 **생존 객체를 빈 리전으로 이주**(Migration)
- 다수의 GC 스레드가 **병렬 처리**

### 정지 시간 제어

#### 기본 설정 및 권장사항

- 목표 정지 시간 기본값: 200밀리초
- 일반적인 적정 범위: 100~300밀리초
- 너무 짧게 설정(예: 20밀리초)하면 힙의 극소량만 회수되어 전체 GC 유발 위험

#### 처리량과 지연 시간의 균형

G1의 공식 목표는 단순한 짧은 지연 시간 추구가 아니라 지연 시간을 제어하는 동시에 처리량을 최대화하는 것입니다.

### 메모리 관리 알고리즘

G1은 광역적으로는 **마크-컴팩트**(Mark-Compact), 지엽적으로는 **마크-카피**(Mark-Copy)를 활용합니다. 이로 인해 메모리 단편화가 발생하지 않아 가비지 컬렉션 후 남은 공간 전체를 효율적으로 활용할 수 있습니다.

#### 주요 장점

- **예측 가능한 정지 시간**: 사용자가 목표 정지 시간을 설정할 수 있음
- **메모리 단편화 방지**: 연속된 메모리 공간 확보로 큰 객체 할당 용이
- **효율적인 회수**: 가비지가 많은 리전 우선 회수로 효율성 극대화
- **처리량과 지연시간 균형**: 두 성능 지표의 최적 균형점 제공

#### 단점 및 주의사항

- **메모리 오버헤드**: 각 리전의 기억 집합 유지를 위한 추가 메모리 필요
- **전체 GC 위험**: 메모리 회수 속도가 할당 속도를 따라가지 못할 경우 발생
- **복잡성**: CMS 대비 더 복잡한 내부 구조와 동작 방식
