---
title: 03 Transactional
date: 2025-09-28 10:00:00 +09:00
categories: [00Spring Framework, SpringData]
tags: [ Java, Spring Framework, SpringData, Transactional]
---

## 1. Transactional
### 1. Transactional
- 스프링에서 제공하는 선언적 트랜잭션 관리 어노테이션
- 메서드나 클래스에 붙이면 트랜잭션 범위를 정의할 수 있음
- 핵심 역할: DB 작업을 하나의 트랜잭션 단위로 묶어서 모두 성공하거나 모두 실패하게 관리

### 2. 동작의 흐름
- 메서드 시작 → 트랜잭션 시작
- 메서드 정상 종료 → 트랜잭션 커밋
- 메서드에서 RuntimeException 발생 → 트랜잭션 롤백
- 체크 예외(Exception)는 기본적으로 롤백하지 않음 (설정 변경 가능)

### 3. 특징

| 항목          | 내용                            |
| ----------- | ----------------------------- |
| 선언적         | 메서드/클래스에 어노테이션만 붙이면 트랜잭션 적용   |
| AOP 기반      | 프록시를 통해 메서드 실행 전후로 트랜잭션 시작/종료 |
| 자동 커밋/롤백    | 정상 시 커밋, 예외 발생 시 롤백 (설정 가능)   |
| propagation | 메서드 간 호출 시 트랜잭션 전파 전략 설정 가능   |


## 2. Propagation
### 1. 트랜잭션 전파
 - 트랜잭션은 DB 작업 단위를 하나의 묶음으로 관리하는 것
 - 스프링에서 @Transactional을 메서드에 붙이면 그 메서드가 트랜잭션 안에서 실행됨
 - 전파(Propagation)는 이미 트랜잭션이 존재할 때 새로운 메서드 호출이 그 트랜잭션과 어떻게 관계를 맺을지를 정의한 것
 - 즉, 메서드 간 트랜잭션 연결 방식을 설정하는 옵션.

### 2. 트랜잭션 전파옵션

| Propagation   | 설명                                        |
| ------------- | ----------------------------------------- |
| REQUIRED (기본값) | 현재 트랜잭션이 있으면 참여, 없으면 새로 생성                |
| REQUIRES_NEW  | 무조건 새 트랜잭션 생성, 기존 트랜잭션은 잠시 보류             |
| SUPPORTS      | 현재 트랜잭션이 있으면 참여, 없으면 트랜잭션 없이 실행           |
| NOT_SUPPORTED | 현재 트랜잭션이 있으면 일시 중단하고 트랜잭션 없이 실행           |
| MANDATORY     | 현재 트랜잭션이 반드시 있어야 함, 없으면 예외                |
| NEVER         | 트랜잭션이 있으면 예외, 없으면 실행                      |
| NESTED        | 현재 트랜잭션 안에서 **중첩 트랜잭션** 생성 (savepoint 사용) |

### 3. 사용
 - 어노테이션 옵션으로 사용함
 ```
 @Service
 public class PaymentService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void pay() {
        // 항상 새로운 트랜잭션 생성
        // 독립적으로 commit/rollback 됨
    }
 }
 ```
