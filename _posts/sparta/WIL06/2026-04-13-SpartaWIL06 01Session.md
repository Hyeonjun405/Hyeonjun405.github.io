---
title: 01 Transaction
date: 2026-04-13 10:00:00 +09:00
categories: [Sparta, SpartaWIL06]
tags: [ Java, Spring Framework ]
---

## 1. Note
- remind!
- 마이바티스&JPA
  - 마이바티스와 JPA가 동일한 데이터소스를 사용하고
  - 스프링부트에서 동일한 @Transactional 안에 있다면 트랜잭션이 공유됨.
  - 이건 쫌 신기한 접근인데

## 2. Transaction
### 1. 트랜잭션
- 트랜잭션은 데이터베이스에서 하나의 논리적인 작업 단위를 의미
- 이 작업 단위는 여러 개의 쿼리로 구성될 수 있지만, 모두 함께 성공하거나 모두 함께 실패해야 함
- 이를 보장하기 위해 원자성, 일관성, 고립성, 지속성이라는 ACID 특성을 만족해함
- 트랜잭션이 정상적으로 완료되면 commit을 통해 변경 사항이 영구 저장됨
- 문제가 발생하면 rollback을 통해 이전 상태로 되돌려 데이터 무결성을 유지함

### 2. 트랜잭션의 흐름
```
-- 트랜잭션 시작 (데이터베이스에 따라 BEGIN이나 START TRANSACTION 사용)
START TRANSACTION;

-- 작업수행 1
UPDATE account
SET balance = balance - 10000
WHERE user_id = 1;

-- 작업수행 2
UPDATE account
SET balance = balance + 10000
WHERE user_id = 2;

-- 커밋(반영)과 롤백(트랜잭션 기점 초기화) 
COMMIT;
ROLLBACK; 
```

### 3. 트랜잭션의 흐름2
```
-- 트랜잭션 시작
START TRANSACTION;

-- 작업1
UPDATE account
SET balance = balance - 10000
WHERE user_id = 1;

-- 중간지점
SAVEPOINT sp1;

-- 작업2
UPDATE account
SET balance = balance + 10000
WHERE user_id = 2;

-- 롤백
-- sp1로 이동하게 되면, 작업1은 유지, 작업2는 초기화가됨
ROLLBACK TO SAVEPOINT sp1;

-- 커밋
COMMIT;
```

## 3. JDBC와 Transaction
### 1. JDBC Transactiion
- JDBC 트랜잭션은 자바에서 데이터베이스 작업을 직접 제어할 때 사용하는 트랜잭션 처리 방식
- 기본적으로 JDBC는 auto-commit이 true라서 SQL 한 줄이 실행될 때마다 자동으로 커밋됨
- 트랜잭션을 직접 관리하려면 
  - setAutoCommit(false)로 자동 커밋을 끄고 여러 SQL을 하나의 작업으로 묶음
  - 작업이 모두 정상적으로 끝나면 commit()을 호출해 변경 내용을 DB에 반영하고, 
  - 문제가 생기면 rollback()으로 전체를 되돌림
  - 필요한 경우 Savepoint를 사용해 중간 지점부터 부분 롤백도 가능

### 2. 흐름
```
// 커넥션 생성
Connection conn = dataSource.getConnection();

// 오토커밋 비활성화
conn.setAutoCommit(false);

// 비즈니스로직 1
PreparedStatement ps1 = conn.prepareStatement("UPDATE account SET balance = balance - 1000 WHERE id=1");
ps1.executeUpdate();

// 부분 롤백 포인트
// Savepoint sp1 = conn.setSavepoint();

// 비즈니스로직 2
PreparedStatement ps2 = conn.prepareStatement("UPDATE account SET balance = balance + 1000 WHERE id=2");
ps2.executeUpdate();

// 커밋과 롤백
conn.commit();
// conn.rollback();
// 부분롤백
// conn.rollback(sp1);

// 커넥션 종료
conn.close();
```

## 4. 트랜잭션 관리 방식
### 1. 비교

| 구분     | 프로그래밍 방식           | 선언적 방식         |
| ------ | ------------------ | -------------- |
| 제어 방식  | 직접 제어              | Spring 자동 처리   |
| 코드 복잡도 | 높음                 | 낮음             |
| 유지보수   | 어려움                | 쉬움             |
| 사용성    | 특수 상황              | 일반적인 표준        |
| 대표 방식  | TransactionManager | @Transactional |



### 2. 프로그래밍 방식 트랜잭션
#### 1. 프로그래밍 방식
- 프로그래밍 방식 트랜잭션은 개발자가 코드에서 직접 트랜잭션을 제어하는 방식
- 트랜잭션의 시작, 커밋, 롤백을 모두 명시적으로 작성
- 특징
  - 개발자가 직접 트랜잭션 경계를 제어
  - 세밀한 제어 가능
  - 코드가 길고 복잡해질 수 있음
  - Spring에서는 TransactionTemplate 또는 PlatformTransactionManager 사용


#### 2. 장단점
- 장점
  - 트랜잭션 제어를 매우 정밀하게 할 수 있음
  - 복잡한 조건 로직에 유리
- 단점
  - 코드가 장황해짐
  - 비즈니스 로직과 트랜잭션 코드가 섞임
  - 유지보수성이 떨어짐

#### 3. 예시
```
@Autowired
private PlatformTransactionManager transactionManager;

public void process() {
    TransactionStatus status =
        transactionManager.getTransaction(new DefaultTransactionDefinition());

    try {
        userRepository.save(user);
        orderRepository.save(order);

        transactionManager.commit(status);
    } catch (Exception e) {
        transactionManager.rollback(status);
    }
}
```

### 3. 선언적 트랜잭션
#### 1. 선언적 방식
- 선언적 트랜잭션은 어노테이션 기반으로 트랜잭션을 자동 처리하는 방식
- Spring에서 가장 많이 사용하는 방식
- 특징
  - @Transactional 하나로 트랜잭션 처리
  - AOP 기반으로 동작
  - 비즈니스 로직과 트랜잭션 로직 분리
  - Spring이 자동으로 commit / rollback 처리

#### 2. 장단점
- 장점
  - 코드가 매우 간결함
  - 비즈니스 로직과 트랜잭션 분리
  - 유지보수성 좋음
  - Spring 표준 방식
- 단점
  - 내부 동작이 보이지 않아 이해 필요
  - 복잡한 트랜잭션 제어에는 한계 (세부 제어 어려움)
  - self-invocation 문제 등 AOP 특성 주의 필요

#### 4. 예시
```
@Transactional
public void process() {
    userRepository.save(user);
    orderRepository.save(order);
}
```
