---
title: 04 Rock
date: 2026-04-16 10:00:00 +09:00
categories: [Sparta, SpartaWIL06]
tags: [ Java, Spring Framework ]
---

## 1. Note
- Rock Remind!
  - Rock은 내부적인 기능이기는 하지만
  - AP에서 뭔가 대안이나 로직을 진행해줘야 하는 부분도 있음
  - 무조건 DB에서만 작업하는 것이 아님!
- isolation랑 Rock랑 구분해서 파악 필요!

## 2. 락
### 1. 락(Rock)
- 여러 트랜잭션이 동시에 같은 데이터를 건드릴 때 충돌을 막기 위해 걸어두는 제어 장치
- 트랜잭션이 데이터에 접근할 때 락을 걸어 다른 트랜잭션이 동일 데이터에 동시에 접근하지 못하도록 제한
- 이를 통해 데이터 손상이나 무결성 문제가 발생하는 것을 방지

### 2. 락의 역할
- 동시성 제어
  - 여러 트랜잭션이 동시에 접근해도 데이터 충돌을 방지
  - 하나의 트랜잭션이 수정 중인 데이터는 다른 트랜잭션이 접근하지 못하게 함
  - 예시) 재고가 1개일 때 동시에 주문이 들어와도 한 명만 구매되도록 만듬
- 데이터 정합성 유지
  - 데이터가 항상 올바른 상태를 유지하도록 보장
  - 중간 상태나 잘못된 값이 저장되는 것을 방지
  - 예시) 계좌 이체 시 출금과 입금이 순서대로 안전하게 처리
- 시스템 안정성 보장
  - 충돌 상황에서도 예측 가능한 결과를 유지
  - 동시 요청으로 인한 데이터 오류를 방지
  - 예시) 포인트 차감 중 오류가 나면 일부만 반영되지 않고 전체가 취소됨

### 3. 락의 종류
#### 1. S락 (Shared Lock, 공유 락)
- 읽기용 락
- 여러 트랜잭션이 동시에 한 데이터에 대해 S-Lock을 획득할 수 있음 
- 하지만 S-Lock이 걸려 있는 동안 다른 트랜잭션이 해당 데이터를 수정(Update/Delete)할 수는 없음
- 흐름
  ```
  # 읽기는 같이, 쓰기는 막음
  A: 데이터 조회 (S락)
  B: 데이터 조회 (S락) → 가능
  C: 데이터 수정 → 대기  
  ```

#### 2. X락 (Exclusive Lock, 배타 락)
- 쓰기용 락
- 특정 트랜잭션이 X-Lock을 획득하면,
- 해당 트랜잭션이 끝날 때까지 다른 트랜잭션은 이 데이터에 접근(읽기/쓰기 모두)할 수 없음
- "내가 고치는 동안 아무도 건드리지 마"라는 독점적인 권한을 가짐
- 흐름
  ```
  # 완전히 혼자 사용
  A: 데이터 수정 (X락)
  B: 데이터 조회 → 대기
  C: 데이터 수정 → 대기  
  ```

#### 3. S락과 X락에서 확장된 락

| 락 종류           | 역할        | 언제 쓰이는가     | 핵심 포인트      |
| -------------- | --------- | ----------- | ----------- |
| Intent Lock    | 락 의도 표시   | Row 락 전에    | 상위 충돌 방지    |
| Update Lock    | 중간 락      | 조회 후 수정 예정  | 데드락 방지      |
| Gap Lock       | 범위 잠금     | Insert 방지   | 값 사이 공간 보호  |
| Next-Key Lock  | Row + Gap | 범위 + 데이터 보호 | 팬텀 리드 방지    |
| Key Range Lock | 범위 전체     | 조건 조회 시     | 범위 자체 보호    |
| Schema Lock    | 구조 보호     | DDL 실행 시    | 테이블 변경 중 보호 |

## 3. 낙관적락, 비관적락
### 1. 비관적 락 
#### 1. 비관적락 (Pessimistic Lock)
- 데이터 충돌이 발생할 가능성을 고려해, 작업 시작 시점부터 락을 걸어 다른 트랜잭션의 접근을 차단하는 방식
- 정합성은 강하게 보장되지만, 대기 시간이 발생해 성능이 떨어질 수 있음
- 예시) 결제, 재고처럼 동시에 수정되면 안 되는 경우

#### 2. 동작 방식
- 데이터를 조회하는 시점에 바로 락을 걸어 다른 트랜잭션의 접근을 차단
- 락이 해제될 때까지 다른 요청은 대기하게 됨

#### 3. 특징
- 충돌을 원천적으로 방지하여 데이터 정합성이 강하게 보장됨
- 대신 대기 시간이 발생해 성능 저하나 락 경합이 발생할 수 있음

#### 4. 흐름
```
-- [요청 A]
START TRANSACTION;
SELECT stock FROM product WHERE id = 1 FOR UPDATE;
-- A: X락 획득 (이 시점부터 다른 요청 접근 차단)

-- [요청 B]
START TRANSACTION;
SELECT stock FROM product WHERE id = 1 FOR UPDATE;
-- B: 여기서 대기 (A가 락 잡고 있어서 접근 불가)

-- [요청 A]
UPDATE product SET stock = stock - 1 WHERE id = 1;
COMMIT;
-- A: 락 해제

-- [요청 B]
-- A가 커밋한 이후 실행됨
SELECT stock FROM product WHERE id = 1 FOR UPDATE;

-- B: 이제 락 획득 가능
UPDATE product SET stock = stock - 1 WHERE id = 1;
COMMIT;
```

### 2. 낙관적 락
#### 1. 낙관적 락 (Optimistic Lock)
- 충돌이 드물다고 가정하고, 락 없이 작업을 진행한 뒤 마지막에 데이터 변경 여부를 검증하는 방식
- 충돌이 발생하면 작업을 실패시키거나 재시도함
- 예시 ) 게시글 수정, 사용자 정보 변경

#### 2. 동작 방식
- 락 없이 데이터를 조회하고 작업을 수행한 뒤, 수정 시점에 version 등을 비교하여 변경 여부를 확인
- 값이 변경되었으면 충돌로 판단하고 작업을 실패시키거나 재시도함

#### 3. 특징
- 락을 사용하지 않아 성능이 좋고 동시 처리에 유리함
- 대신 충돌 발생 시 별도의 예외 처리나 재시도 로직이 필요함

#### 4. 흐름
```
-- [요청 A]
START TRANSACTION;
SELECT stock, version FROM product WHERE id = 1;
-- A: version = 1 조회
-- 버전 컬럼(버전을 알 수 있는 컬럼)을 두고 그거에 맞게 작업을 진행함. 

-- [요청 B]
START TRANSACTION;
SELECT stock, version FROM product WHERE id = 1;
-- B: version = 1 동일하게 조회 (락 없음, 동시에 진행)

-- [요청 A]
UPDATE product
SET stock = stock - 1,
    version = version + 1
WHERE id = 1 AND version = 1;
-- A: 성공 (version 2로 변경)

COMMIT;

-- [요청 B]
UPDATE product
SET stock = stock - 1,
    version = version + 1
WHERE id = 1 AND version = 1;
-- B: 실패 (이미 version이 2라 조건 불일치)

-- 처리 로직
-- B: 실패 감지 → 재조회 후 재시도 또는 에러 처리

-- 충돌 자체는 DB가 만들고 충돌로 판단하고 처리하는 건 Application(AP)
-- AP에서 업데이트 건수보고 뭔가 작업을 처리함. 

COMMIT;
```

## 4. JPA와 Rock락
### 1. 비관적 락 (Pessimistic Lock)
#### 1. 종류

| LockMode                    | 설명                | 특징           | 사용 상황         |
| --------------------------- | ----------------- | ------------ | ------------- |
| PESSIMISTIC_WRITE           | 쓰기 락 (X락)         | 읽기/쓰기 모두 차단  | 재고, 결제        |
| PESSIMISTIC_READ            | 읽기 락 (S락)         | 읽기 허용, 쓰기 차단 | 변경 방지 조회      |
| PESSIMISTIC_FORCE_INCREMENT | 쓰기 락 + version 증가 | 낙관적 락과 혼합    | 버전 강제 증가 필요 시 |

#### 2. PESSIMISTIC_WRITE
- Repository
  ```
  @Lock(LockModeType.PESSIMISTIC_WRITE)
  @Query("select p from Product p where p.id = :id")
  Product findByIdForUpdate(Long id);
  ```
  
- Service
  ```
  @Transactional
  public void decreaseStock(Long id) {
    Product p = repository.findByIdForUpdate(id); // 여기서 락걸리고
    p.decreaseStock(1); // 작업하고
  }// 트랜잭션이 끝나면 Commit하고, 락풀림
  ```

#### 3. PESSIMISTIC_READ
```
@Lock(LockModeType.PESSIMISTIC_READ)
@Query("select p from Product p where p.id = :id")
Product findByIdForShare(Long id);
```
#### 4. PESSIMISTIC_FORCE_INCREMENT
- 흐름 파악(개발자는 별도 작업X, 락만 표기)
  - DB: “조건 안 맞으면 업데이트 안함 (0 row)”
  - JPA: “이건 충돌이네 → 예외 던짐”
  - AP: “예외 받으면 재시도 or 실패 처리”

- Entity
```
@Version
private Long version; // 컬럼추가 필요
```

- Repository
```
@Lock(LockModeType.PESSIMISTIC_FORCE_INCREMENT)
@Query("select p from Product p where p.id = :id")
Product findByIdWithVersion(Long id);
```

### 2.낙관적락
#### 1. 종류

| LockMode                   | 설명                 | 특징                     | 사용 상황          |
| -------------------------- | ------------------ | ---------------------- | -------------- |
| OPTIMISTIC                 | 기본 낙관적 락           | 수정 시 version 비교로 충돌 감지 | 일반적인 동시성 제어    |
| OPTIMISTIC_FORCE_INCREMENT | 조회 시 version 강제 증가 | 읽기만 해도 충돌 유도           | 강제 동시성 제어 필요 시 |
| READ                       | OPTIMISTIC과 동일     | 사실상 OPTIMISTIC 별칭      | 조회 기반 충돌 감지    |


#### 2. OPTIMISTIC (기본 낙관적 락)
- Entity
```
@Version
private Long version; // 컬럼추가 필요
```

- Repository
```
@Lock(LockModeType.OPTIMISTIC)
@Query("select p from Product p where p.id = :id")
Product findByIdOptimistic(Long id);
```

- Service
```
@Transactional
public void decreaseStock(Long id) {
    Product p = repository.findByIdOptimistic(id);
    p.decreaseStock(1);
}
```



#### 3. OPTIMISTIC_FORCE_INCREMENT
- Entity
```
@Version
private Long version; // 컬럼추가 필요
```

- Repository
```
@Lock(LockModeType.OPTIMISTIC_FORCE_INCREMENT)
@Query("select p from Product p where p.id = :id")
Product findByIdForce(Long id);
```

- Service
  ```
  @Transactional
  public void readAndLock(Long id) {
    Product p = repository.findByIdForce(id);// 읽기만 해도 version 증가됨
    
  } // 조회 시점에 증가하지만, 실제 DB 반영은 flush/commit 시점
  ```

#### 4. READ
- repository
```
@Lock(LockModeType.READ)
@Query("select p from Product p where p.id = :id")
Product findByIdRead(Long id);
```

- SErivce
```
@Transactional
public void update(Long id) {
    Product p = repository.findByIdRead(id);
    p.decreaseStock(1);
}
```
