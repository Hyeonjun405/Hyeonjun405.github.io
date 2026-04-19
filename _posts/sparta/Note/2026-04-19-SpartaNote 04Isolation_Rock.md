---
title: 04 Isolation&Rock
date: 2026-04-19 10:00:00 +09:00
categories: [Sparta, SpartaNote]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - 격리수준과 락
   - 격리수준은 다른 트랜잭션의 변경이 내 트랜잭션에 어떻게 보일지
   - 락은 다른 트랜잭션이 해당 데이터에 대해 읽기/쓰기 행동을 못 하게 제한

## 2. Isolation&Rock
### 1. Isolation (격리 수준)
- “트랜잭션끼리 서로 얼마나 안 보이게 할 것인가”
- 종류

  | 격리수준             | Dirty Read | Non-Repeatable Read | Phantom Read | 특징           |
  | ---------------- | ---------- | ------------------- | ------------ | ------------ |
  | READ UNCOMMITTED | O          | O                   | O            | 거의 사용 안 함    |
  | READ COMMITTED   | X          | O                   | O            | 대부분 DB 기본    |
  | REPEATABLE READ  | X          | X                   | O*           | MySQL 기본     |
  | SERIALIZABLE     | X          | X                   | X            | 가장 강력 (성능 ↓) |


### 2. Lock (락)
- “데이터를 실제로 누가 건드릴 수 있는지 막는 기술”
- 종류

  | 항목    | OPTIMISTIC   | PESSIMISTIC_READ | PESSIMISTIC_WRITE |
  | ----- | ------------ | ---------------- | ----------------- |
  | 조회    | 자유롭게 가능      | 가능               | 일반 조회는 가능         |
  | 수정    | 가능 (충돌 시 실패) | 제한               | 완전 차단             |
  | 동시 실행 | 가능           | 가능               | 불가능 (대기)          |
  | 충돌 처리 | 예외 발생        | 없음               | 대기                |


### 3. Note
- 스프링에서 Isolation의 기본값은 DB를 따라감
- 두가지 다 필수적인 항목은 아님.
- 상황에 따라서, 필요에따라서 선언하고 사용하는 과정.

## 3. 케이스1 (조회 → 검증 → 수정)
### 1. 소스
- Service
```
// 내가 읽은 데이터를 트랜잭션 동안 일관되게 유지하기 위한 것
// 지금은 Rock이 걸려서 크게 막 어마어마한 효과 X
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void reserve(Long seatId) {
    // 좌석 조회
    Seat seat = seatRepository.findForUpdate(seatId)
            .orElseThrow();

    // 예약좌석 여부 검증
    if (seat.isReserved()) {
        throw new IllegalStateException();
    }

    // 예약좌석으로 수정
    seat.reserve();
}
```

- Repository
```
// 내가 자업중에 다른 트랜잭션이 그 데이터를 수정하지 못하게 막기 위한 것
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select s from Seat s where s.id = :id")
Optional<Seat> findForUpdate(Long id);
```

### 2. Memo
- Lock만 있고 Isolation 없으면
  - 내가 두 번 조회했는데 값이 바뀔 수도 있음 (특히 DB마다 다름)
  - range query에서는 깨짐 (phantom)
- Isolation만 있고 Lock 없으면
  - 다른 트랜잭션이 값을 바꿔버림
  - 내가 outdated 값으로 update (Lost Update)
- 둘다 없다면
  - 트랜잭션 A와 B가 돌 때, 
  - A가 조회후에 B가 커밋을 한 경우에
  - A는 캐싱되어있는것을 조회하기때문에 별도 확인X
  - 그냥 초기 조회 상태로 업데이트 진행함.

## 4. 케이스2 (범위 조건 + 팬텀 리드 방지)
### 1. 소스
```
@Transactional(isolation = Isolation.SERIALIZABLE)
public void createReservation(LocalDateTime time) {

    // 특정 시간대에 예약이 있는지 조회
    boolean exists = reservationRepository.existsByTimeRange(time);

    // 예약이 있으면 예외발생
    if (exists) {
        throw new IllegalStateException();
    }

    // 아니면 막기
    reservationRepository.save(new Reservation(time));
}
```

### 2. memo
- Repository에서 락을 걸 수 없는 이유
  - query가 `where start_time <= ? and end_time >= ?` 이런 패턴
  - 여기에서 데이터가 있다면 특정 로우 범위에 락이 가능하기는 한데,
  - 데이터 자체가 존재하지 않는다면 락을 걸 대상 자체가 없음.
  - 그렇다고 전체 테이블에 락을 거는건 비효율적임.
- 대안
  - SERIALIZABLE로 Gap-rock형태로 만들어서 그 공간을 제어할 수 없도록 함. 
  - Time 컬럼에 인덱스추가 + PESSIMISTIC_WRITE을 사용해 gapRock 유도


## 5. 케이스3(읽기 일관성 + 외부 시스템 연동)
### 1. 소스
- Service
```
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void process(Long id) {

    //주문 조회하고
    Order order = orderRepository.findByIdForUpdate(id)
            .orElseThrow();

    // 외부 API 호출
    externalApi.call(order.getAmount());

    //상태 변경처리
    order.complete();
}
```
- Repository
```
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select o from Order o where o.id = :id")
Optional<Order> findByIdForUpdate(Long id);
```

### 2. Memo
- 1번 케이스와 매우 유사함
  - 일관성 유지

## 6. 케이스4(읽기 안정성 + 충돌 시도 자체 차단)
### 1. 소스
```
@Transactional
@Lock(LockModeType.PESSIMISTIC_WRITE)
public void issueCoupon(Long userId) {

    // 1. 발급 여부 체크
    boolean exists = couponRepository.existsByUserId(userId);
    if (exists) {
        throw new IllegalStateException("이미 발급됨");
    }

    // 2. 발급
    couponRepository.save(new Coupon(userId));

    // 3. 로그 저장
    couponLogRepository.save(new CouponLog(userId));
}
```

### 2. Memo
- 순차적으로 여러레파지토리 작업을 진행해야하는 경우
  - 서비스에 락을 걸면 순서대로 락을 걸면서 진행하게됨
  - 만약에 중간에 락 대기 또는 예외가 터지는 경우 전체 롤백 처리 
