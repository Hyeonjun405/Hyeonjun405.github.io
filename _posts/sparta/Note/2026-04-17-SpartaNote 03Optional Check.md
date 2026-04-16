---
title: 03 Optional Check
date: 2026-04-17 10:00:00 +09:00
categories: [Sparta, SpartaNote]
tags: [ Java, Spring Framework ]
---

## 1. Note
- "이 메소드는 값을 반환할 수도 있고, 없을 수도 있다"라는 표현
- 내부에서 JPA save
  - 역할 자체가 무너지는 상황이 생김
  - 옵셔널이 값의 유무를 확인하는데, 
  - 값자기 레파지토리 저장을 하는건 안맞음
- Optional 내부에서는 “값 존재 여부 판단과 변환까지만” 하고,
  - JPA save 같은 상태 변경은 서비스 계층에서 처리하는 게 가장 안전한 구조

## 2. 값 생성

| 메소드                            | 설명                       | 특징            |
| ------------------------------ | ------------------------ | ------------- |
| `Optional.of(T value)`         | null이 아닌 값으로 Optional 생성 | null이면 NPE 발생 |
| `Optional.ofNullable(T value)` | null 허용 생성               | 가장 많이 사용      |
| `Optional.empty()`             | 빈 Optional 생성            | 값 없음 표현       |


## 3. 값 유무 확인

| 메소드           | 설명          | 반환                 |
| ------------- | ----------- | ------------------ |
| `isPresent()` | 값이 있으면 true | boolean            |
| `isEmpty()`   | 값이 없으면 true | boolean (Java 11+) |


## 3. 값 꺼내기

| 메소드                     | 설명                | 특징                        |
| ----------------------- | ----------------- | ------------------------- |
| `get()`                 | 값 반환              | 값 없으면 예외 (위험)             |
| `orElse(T other)`       | 값 없으면 기본값 반환      | 항상 평가됨                    |
| `orElseGet(Supplier)`   | 값 없으면 Supplier 실행 | lazy 실행 (권장)              |
| `orElseThrow()`         | 값 없으면 예외 발생       | 기본 NoSuchElementException |
| `orElseThrow(Supplier)` | 커스텀 예외            | 실무에서 많이 사용                |

## 4. 조건처리

| 메소드                                   | 설명                       |
| ------------------------------------- | ------------------------ |
| `ifPresent(Consumer)`                 | 값 있으면 실행                 |
| `ifPresentOrElse(Consumer, Runnable)` | 값 있으면 A, 없으면 B (Java 9+) |

## 5. 변환&가공

| 메소드                 | 설명             | 특징                       |
| ------------------- | -------------- | ------------------------ |
| `map(Function)`     | 값 변환           | null이면 그대로 empty         |
| `flatMap(Function)` | 중첩 Optional 제거 | Optional<Optional<T>> 방지 |
| `filter(Predicate)` | 조건 만족 시 유지     | 아니면 empty                |

## 6. 기타

| 메소드            | 설명                             |
| -------------- | ------------------------------ |
| `stream()`     | Optional → Stream 변환 (Java 9+) |
| `or(Supplier)` | 값 없으면 대체 Optional 제공           |

