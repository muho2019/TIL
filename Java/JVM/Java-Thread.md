# 자바 스레드

## 스레드란 무엇인가?

스레드는 프로세스 내에서 실행되는 독립적인 실행 흐름입니다. 하나의 프로세스는 여러 개의 스레드를 가질 수 있으며 이를 통해 동시성(Concurrency)을 구현할 수 있습니다.

- **프로세스(Process)**: 운영체제에서 실행 중인 프로그램의 인스턴스
- **스레드(Thread)**: 프로세스 내에서 실행되는 가벼운 실행 단위
- 스레드들은 같은 프로세스 내에서 메모리 공간(Memory Space)을 공유합니다.

## 자바에서의 스레드

자바는 언어 차원에서 멀티스레딩(Multithreading)을 지원합니다. JVM은 여러 스레드를 동시에 실행할 수 있으며 각 스레드는 독립적인 실행 스택(Execution Stack)을 가집니다.

- **메인 스레드(Main Thread)**: 모든 자바 프로그램은 main() 메서드에서 시작되는 메인 스레드를 가집니다.
- **데몬 스레드(Daemon Thread)**: 백그라운드에서 실행되는 보조 스레드
- **사용자 스레드(User Thread)**: 일반적인 작업을 수행하는 스레드

## 스레드 생성 방법

자바에서 스레드를 생성하는 방법은 주로 두 가지입니다.

- Thread 클래스 상속

```java
class MyThread extends Thread {
    @Override
    public void run() {
        // 스레드가 실행할 코드
    }
}

MyThread thread = new MyThread();
thread.start(); // 스레드 시작
```

- Runnable 인터페이스 구현

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        // 스레드가 실행할 코드
    }
}

Thread thread = new Thread(new MyRunnable());
thread.start();
```

## 스레드 생명주기 (Thread Lifecycle)

스레드는 다음과 같은 상태들을 가집니다.

- **NEW**: 스레드가 생성되었지만 아직 시작되지 않은 상태
- **RUNNABLE**: 실행 중이거나 실행 준비가 된 상태
- **BLOCKED**: 동기화 블록에 진입하기 위해 대기 중인 상태
- **WAITING**: 다른 스레드의 특정 작업 완료를 무한정 기다리는 상태
- **TIMED_WAITING**: 지정된 시간 동안 대기하는 상태
- **TERMINATED**: 실행이 완료된 상태

## 스레드 동기화 (Thread Synchronization)

여러 스레드가 공유 자원(Shared Resources)에 동시에 접근할 때 발생할 수 있는 문제를 해결하기 위한 메커니즘입니다.

- 주요 동기화 방법
  - **synchronized 키워드**: 메서드나 블록에서 상호 배제(Mutual Exclusion) 제공
  - **Lock 인터페이스**: 더 세밀한 제어가 가능한 동기화 메커니즘
  - **volatile 키워드**: 변수의 가시성(Visibility) 보장
- 동기화 문제들
  - **경쟁 상태(Race Condition)**: 여러 스레드가 공유 자원에 동시 접근하여 예상치 못한 결과가 발생하는 상황
  - **데드락(Deadlock)**: 두 개 이상의 스레드가 서로의 자원을 기다리며 무한정 대기하는 상황

## 스레드 풀 (Thread Pool)

스레드 풀은 미리 생성된 스레드들을 재사용하여 성능을 향상시키는 패턴입니다.

- 장점
  - 스레드 생성/소멸 비용 절약
  - 시스템 자원의 효율적 관리
  - 응답 시간 개선

```java
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(() -> {
    // 작업 내용
});
executor.shutdown();
```
