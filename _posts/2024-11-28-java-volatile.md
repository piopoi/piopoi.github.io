---
title: "OS 스레드의 메모리 접근방식과 Java의 volatile 키워드"
date: 2024-11-28
categories: knowledge
tags:
  - java
  - concurrency
  - study
---

이번 주제는 Java의 `volatile` 키워드로, 오랜만에 동시성(Concurrency) 문제와 관련된 주제를 다루게 되었다. 스터디를 준비하며 `volatile` 키워드에 대해 간단하게 정리해본다.

이 글은 아래와 같은 순서로 진행된다.

1. 스레드가 메모리에 접근하는 방식
   - 운영체제의 스레드의 경우.
   - Java의 volatile 키워드의 경우.
2. Java의 volatile 키워드에 대해.

<br>

# 스레드의 메모리 접근 방식 비교: OS vs Java volatile

## OS 스레드의 메모리 접근 방식

### 읽기(Read) 동작

```
[스레드] → [L1 캐시 확인] → [L2 캐시 확인] → [L3 캐시 확인] → [메인 메모리]
```

1. 스레드가 특정 메모리 주소의 값을 요청한다.
2. CPU는 먼저 L1 캐시를 확인한다.
   - 값이 있으면 (캐시 히트) → 해당 값을 반환.
   - 값이 없으면 (캐시 미스) → L2 캐시로 이동하여 값을 찾는다.
3. L2 캐시 확인:
   - 캐시 히트 → 값을 반환하고 L1 캐시에 복사한다.
   - 캐시 미스 → L3 캐시로 이동하여 값을 찾는다.
4. L3 캐시 확인:
   - 캐시 히트 → 값을 반환하고 하위 캐시(L2 및 L1)에 복사한다.
   - 캐시 미스 → 메인 메모리로 접근하여 값을 읽는다.
5. 메인 메모리에서 값을 읽어오면, L3 → L2 → L1 캐시 계층에 순차적으로 복사하며 값을 반환한다.

### 쓰기(Write) 동작

#### Write-Back 정책 (일반적인 경우)

```
[스레드] → [L1 캐시에 쓰기] → ... → [나중에 메인 메모리에 반영]
```

1. 스레드가 값을 쓰려고 할 때:
   - 먼저 L1 캐시에 기록한다.
   - 기록된 캐시 라인은 “더티(dirty)” 상태로 표시된다.
2. 캐시 동기화:
   - 다른 코어의 캐시에 동일한 데이터가 존재하면 이를 무효화(invalidate) 한다.
   - MESI(Modified, Exclusive, Shared, Invalid) 프로토콜을 통해 캐시 일관성을 유지한다.
     - 멀티코어 환경에서는 각 코어가 자체 캐시를 사용하므로, 동일한 데이터의 일관성을 유지하기 위해 MESI 프로토콜과 같은 캐시 일관성 알고리즘이 필요하다.
3. 메인 메모리에 반영:
   - 캐시 라인이 교체될 때(예: 캐시 공간 부족으로 인해 다른 데이터로 대체되는 경우).
   - 시스템이 적절한 시점을 판단하여 자동으로 동기화할 때.
   - 명시적인 동기화 요청(예: flush 또는 sync 명령)이 있을 때.

#### Write-Through 정책 (특수한 경우)

```
[스레드] → [L1 캐시에 쓰기] → [즉시 메인 메모리에 반영]
```

1. L1 캐시에 값을 기록함과 동시에, 변경 내용을 메인 메모리에도 즉시 반영한다.
2. 캐시 라인은 “클린(clean)” 상태로 유지되며, 메모리와 항상 동기화된다.
3. 메모리 일관성 유지가 중요한 특정 상황(예: 임베디드 시스템, 실시간 시스템)에서 사용된다.

## Java의 volatile 변수의 메모리 접근 방식

### 읽기(Read) 동작

- 캐시된 값을 사용하지 않고 메인 메모리에서 직접 읽는다.
- 메모리 배리어를 통해 명령어 재배치를 방지한다.

### 쓰기(Write) 동작

 - 캐시를 건너뛰고 직접 메인 메모리에 쓴다.
 - 다른 코어의 캐시를 무효화한다.

### 메모리 배리어와 명령어 재배치
CPU나 컴파일러는 성능 최적화를 위해 코드의 실행 순서를 변경할 수 있다.

```java
// 원래 코드
int a = 1;        // 1번
int b = 2;        // 2번
volatile int x = a + b;  // 3번
int c = 3;        // 4번

// CPU가 재배치한 실행 순서
int b = 2;        // 2번
int c = 3;        // 4번
int a = 1;        // 1번
volatile int x = a + b;  // 3번
```

#### 메모리 배리어의 역할
1. 명령어 재배치 방지
   - volatile 변수 전후로 메모리 배리어가 생성된다
   - volatile 읽기/쓰기 전후의 코드가 재배치되는 것을 막는다
2. 예시
    ```java
    volatile boolean flag = false;
    int data = 0;

    // Thread 1
    void writeData() {
        data = 42;            // 1번
        flag = true;          // 2번 (volatile 쓰기)
    }

    // Thread 2
    void readData() {
        while (!flag) {}      // volatile 읽기
        System.out.println(data);  // data 읽기
    }
    ```
   - 메모리 배리어가 없다면: 1번과 2번의 순서가 바뀔 수 있다
   - 메모리 배리어로 인해: volatile 변수 전후로 순서가 보장된다

<br>

# Java의 volatile 키워드

`volatile`은 **Java 메모리 모델**에서 특정 변수의 값이 스레드 간에 즉시 일관성을 유지하도록 보장하는 키워드다. 기본적으로 변수의 값을 각 스레드가 자신만의 **캐시**에서 읽고 쓰기 때문에, 변수 값이 예기치 않게 동기화되지 않을 수 있다. `volatile`을 사용하면 모든 스레드가 변수 값을 직접 **주 메모리(Main Memory)**에서 읽고 쓴다.

## volatile의 특징

### 가시성(Visibility) 보장
`volatile` 변수에 대한 읽기와 쓰기는 **항상 주 메모리**에서 이루어진다. 따라서 한 스레드가 변수 값을 변경하면, 다른 스레드에서 즉시 변경된 값을 읽을 수 있다.

```java
public class VolatileExample {
    private static volatile boolean flag = true;

    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            while (flag) {
                // flag 값 변경을 기다림
            }
            System.out.println("Thread1 종료");
        });

        Thread thread2 = new Thread(() -> {
            try {
                Thread.sleep(1000);
                flag = false; // 주 메모리에 값 변경
                System.out.println("flag 값 변경: false");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

위 코드에서 flag는 volatile로 선언되어 있으므로, thread2에서 값을 변경하면 thread1에서 즉시 변경된 값을 감지한다.

### 원자성(Atomicity) 미보장

volatile은 가시성만 보장하며, 복합 연산(예: 증가 연산, 조건부 갱신 등)의 원자성은 보장하지 않는다.

```java
public class NonAtomicExample {
    private static volatile int count = 0;

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    count++; // 복합 연산 (읽기 -> 쓰기)
                }
            }).start();
        }

        try {
            Thread.sleep(2000);
            System.out.println("최종 count 값: " + count);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

위 코드에서 count는 예상치 못한 결과를 출력할 가능성이 크다. 복합 연산은 원자성을 보장하지 않기 때문이다.

## volatile vs synchronized

사용 목적과 범위가 다르다.

### volatile

- 목적: 변수의 가시성(visibility) 보장
- 범위: 단일 변수에만 적용

```java
public class VisibilityExample {
    private volatile boolean flag = false;  // 단일 변수
    private volatile int counter = 0;       // 단일 변수
}
```

### synchronized

- 목적: 코드 블록이나 메서드의 원자성(atomicity) 보장
- 범위: 여러 문장, 여러 변수를 포함하는 블록

```java
public class AtomicityExample {
    private int counter = 0;
    private boolean flag = false;
    
    synchronized void updateState() {  // 메서드 전체
        counter++;
        flag = true;
    }
    
    void updateStateBlock() {
        synchronized(this) {  // 블록 단위
            counter++;
            flag = true;
        }
    }
}
```

## volatile 사용 사례

### 상태 플래그 관리

`volatile`은 상태 플래그를 관리하거나 이벤트를 감지할 때 유용하다.

```java
private static volatile boolean running = true;

public static void stopRunning() {
    running = false;
}
```

### 이중 검증 잠금(Double-Checked Locking)

volatile은 객체 초기화 시 발생할 수 있는 문제를 방지하는 데 사용된다.

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {                 //1
            synchronized (Singleton.class) {
                if (instance == null) {         //2
                    instance = new Singleton(); //3
                }
            }
        }
        return instance;                        //4
    }
}
```

Double-Checked Locking 방식에서 `volatile` 키워드가 빠진 경우, 인스턴스 초기화 중 발생할 수 있는 가시성 문제로 인해 예기치 않은 동작이 발생할 수 있다. 

이를 Java Memory Model(JMM) 관점에서 설명하면 다음과 같다:

문제: 잘못 초기화된 객체 참조 가능

- new Singleton()이 호출될 때 JVM은 아래 단계를 수행한다:
  1. 메모리 할당
  2. Singleton 생성자 호출
  3. instance 변수에 참조 할당
- 위 작업은 컴파일러 또는 CPU의 명령 재정렬(Instruction Reordering) 최적화에 의해 순서가 변경될 수 있다. 즉, 2번(생성자 호출)보다 3번(참조 할당)이 먼저 실행될 수 있다.

시나리오:
1. Thread A가 `getInstance()`를 호출하여 첫 번째 null 체크(//1)를 통과하고, `synchronized` 블록에 진입한다.
2. Thread A는 `new Singleton()` 호출을 시작하며, 명령 재정렬로 인해 `instance`에 미리 할당(3번)이 발생할 수 있다.
3. Thread B가 `getInstance()`를 호출한다.
   - 이때 `instance가` null이 아니므로 두 번째 null 체크(//2)를 통과하지 않고 바로 리턴(//4)된다.
4. 그러나 Thread B가 리턴한 `instance는` 아직 생성자의 초기화가 완료되지 않은 불완전한 객체이다.

결과:
- Thread B는 잘못 초기화된 `instance`를 사용할 가능성이 있다.
- 이로 인해 `NullPointerException`과 같은 런타임 에러가 발생하거나, 잘못된 상태의 객체가 반환될 수 있다.

<br>

## References.

[claude](https://claude.ai/){:target="_blank"}<br>
[chatgpt](https://chatgpt.com/){:target="_blank"}<br>