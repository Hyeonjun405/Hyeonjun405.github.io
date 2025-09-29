---
title: 02 threadLock
date: 2025-09-26 10:00:00 +09:00
categories: [Java, thread]
tags: [ Java, threadLock ]
---

## 1. 스레드락 요약

| 락 이름             | 요약              | 대표 클래스                      | 언제 사용하나?        |
| ---------------- | --------------- | --------------------------- | --------------- |
| Mutex            | 가장 기본적인 배타 락    | `synchronized`, `Lock`      | 충돌 방지, 단순 구조    |
| Reentrant Lock   | 재진입 가능, 유연한 설정  | `ReentrantLock`             | 재귀, 공정성, 타임아웃 등 |
| Read-Write Lock  | 읽기 병렬화, 쓰기 단독   | `ReentrantReadWriteLock`    | 읽기 많은 구조        |
| Stamped Lock     | 낙관적 읽기, 성능 최적화  | `StampedLock`               | 고성능 읽기 중심       |
| Semaphore        | 동시 접근 수 제한      | `Semaphore`                 | 리소스 개수 제어       |
| Fair / Non-Fair  | 순서 보장 여부 선택 가능  | `ReentrantLock(true/false)` | 기아 방지 필요 시      |
| Optimistic Lock  | 먼저 읽고 나중에 충돌 검사 | `StampedLock`               | 충돌 드문 읽기        |
| Pessimistic Lock | 항상 락을 먼저 걸고 수행  | 전통 락 전반                     | 충돌 자주 발생 시      |


## 2. 스레드락 종류
### 1. Mutex (상호 배제 락) 🔍
- 가장 기본적인 락. 하나의 스레드만 접근 가능하게 해서 충돌을 방지함.
- synchronized, 또현는 Lock 인터페이스 구체 사용 (ReentrantLock)
- JVM 키워드 / java.util.concurrent.locks.Lock
- 언제 사용
    - 자원에 하나의 스레드만 접근해야 하는 경우
    - 동기화가 필요하지만 구조가 단순한 경우 (synchronized 사용)
    
### 2. Reentrant Lock (재진입 가능 락) 🔍
- 같은 스레드가 여러 번 락을 걸 수 있도록 허용하는 락.
- ReentrantLock
- java.util.concurrent.locks.ReentrantLock
- 언제 사용
    - 락을 걸고 다시 내부에서 재귀적으로 호출되거나
    - 락 해제 타이밍을 명확히 제어하고 싶을 때 (unlock()을 직접 호출)
    - 공정성 제어(Fair), 타임아웃, 인터럽트 처리가 필요한 경우

### 3. Read-Write Lock (읽기-쓰기 분리 락) 🔍
- 읽기는 동시에, 쓰기는 단독으로 처리하여 읽기 많은 작업에서 성능 향상.
- ReentrantReadWriteLock
- java.util.concurrent.locks.ReentrantReadWriteLock
- 언제 사용
    - 데이터는 자주 읽히지만, 쓰기는 드물게 발생하는 경우
    - 예: 캐시, 설정 파일, 조회 위주의 데이터 구조 보호

### 4. Stamped Lock (버전 기반 락)  
- 낙관적 락 기반으로, 읽기 성능 최적화에 초점 맞춘 락.
- StampedLock
- java.util.concurrent.locks.StampedLock (Java 8+)
- 언제 사용
    - 읽기 작업이 매우 많고, 쓰기 충돌이 거의 없는 경우
    - 고성능이 중요한 데이터 구조 (예: 통계 정보, 위치 정보 처리 등)

### 5. Semaphore (자원 개수 제한 락)
- 동시에 접근 가능한 스레드 수를 제한하는 카운팅 락.
- Semaphore.acquire(), release()
- java.util.concurrent.Semaphore
- 언제 사용
    - 제한된 수의 리소스 (예: DB 커넥션, 파일 핸들 등)를 관리할 때
    - 하나의 자원이 아니라 복수 개의 자원에 대해 접근을 제어할 때

### 6. Fair / Non-Fair Lock (공정성 조절 락)
- 락 요청 순서를 보장할지 여부를 설정할 수 있는 락.
- new ReentrantLock(true) → 공정 / false → 비공정
- ReentrantLock 생성자 옵션
- 언제 사용
    - 락 획득 순서를 보장해야 하는 경우 (예: 큐 처리, 우선순위 보장 등)
    - 경쟁이 심한 상황에서 기아(Starvation)를 방지하고 싶을 때

### 7. Optimistic Lock (낙관적 락)
- 락 없이 먼저 접근하고, 충돌이 있었는지만 나중에 검사.
- tryOptimisticRead(), validate()
- StampedLock
- 언제 사용
    - 읽기 성능이 중요하고, 쓰기 충돌이 매우 적은 경우
    - 데이터를 읽고 나서 다시 확인 가능한 구조일 때 (불일치 시 재시도 가능해야 함)

### 8. Pessimistic Lock (비관적 락) 🔍
- 항상 락을 먼저 걸고 시작하는 전통적인 방식의 락.
- synchronized, ReentrantLock, ReadLock, WriteLock 등
- 대부분의 전통적 락 구조에 포함
- 언제 사용
    - 경합이 잦고, 충돌이 발생할 가능성이 높은 경우
    - 자원에 대한 일관성을 철저히 보장해야 할 때
