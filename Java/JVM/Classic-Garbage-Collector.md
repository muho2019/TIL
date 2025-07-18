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

- STW(Stop-The-World) 발생
- GC Roots로부터 직접 참조되는 객체들을 표시
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

3. **메모리 파편화 (Memory Fragmentation)**

- CMS는 마크-스윕 알고리즘을 사용하기 때문에 메모리 파편화가 심각합니다.
- 압축(Compaction) 과정이 없어 메모리가 조각조각 나뉩니다.
- 파편화가 심하면 큰 객체를 할당할 때 특히 문제가 됩니다.

### 지원 종료

CMS 컬렉터는 **JDK 9**에서 **deprecated**되었고, **JDK 14**에서 **완전히 제거**되었습니다. 현재는 **G1 GC**나 **ZGC**, **Shenandoah** 등의 최신 가비지 컬렉터로 대체되고 있습니다.
