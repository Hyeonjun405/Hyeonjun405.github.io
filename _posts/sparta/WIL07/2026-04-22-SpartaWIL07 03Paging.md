---
title: 03 Paging
date: 2026-04-22 10:00:00 +09:00
categories: [Sparta, SpartaWIL07]
tags: [ Java, Spring Framework ]
---

## 1. Note
- 페이징처리..!

## 2. Paging
### 1. Paging
- 데이터를 한 번에 전부 가져오지 않고, 일정 단위(페이지)로 나눠서 가져오는 방식
- 사용자는 필요한 데이터만 한 번에 조회할 수 있으며,
- 전체 데이터를 나누어 제공함으로써 성능을 최적화가능함.

### 2. 주요목적
#### 1. 성능최적화
- 대량 데이터를 한 번에 조회하면 DB와 서버에 큰 부담이 생김
- 조회 시간도 길어지고 메모리 사용량도 급격히 증가함
- 페이징은 필요한 개수만 조회하도록 제한해서 이런 문제를 줄임
- 결과적으로 시스템 전체의 안정성과 응답 속도를 유지할 수 있음

#### 2. 사용자 경험 개선
- 사용자는 많은 데이터를 한 번에 보는 것보다 나눠 보는 것이 편함
- 페이징을 적용하면 화면이 깔끔해지고 원하는 정보를 찾기 쉬워짐
- 또한 초기 로딩 속도가 빨라져 체감 성능이 좋아짐
- 무한 스크롤이나 페이지 이동 같은 UI 구현도 가능해짐

#### 3. 네트워크 비용 절약
- 한 번에 많은 데이터를 전송하면 응답 크기가 커짐
- 이는 특히 모바일 환경에서 속도 저하로 이어질 수 있음
- 페이징을 사용하면 필요한 데이터만 전송하게 되어 효율적
- 그 결과 트래픽 사용량과 응답 지연을 줄일 수 있음

### 3. 동작원리
#### 1. LIMIT
- 조회할 데이터의 최대 개수를 제한
- 한 번에 몇 개의 데이터를 가져올지를 결정
- SQL
  ```
  -- 최대 10개의 데이터만 조회
  SELECT * FROM user LIMIT 10;
  ```

#### 2. OFFSET
- 조회 시작 위치를 지정하며, 앞의 데이터를 건너뜀
- 몇 번째 데이터부터 가져올지를 결정함
- SQL
  ```
  -- 앞의 20개를 건너뛰고 이후부터 조회
  SELECT * FROM user OFFSET 20;
  ```

#### 3. 혼합
```
// 21번째 데이터부터 10개 조회 (21 ~ 30번째)
SELECT * FROM user LIMIT 10 OFFSET 20;
```

## 3. 페이징 처리의 한계
### 1. OFFSET이 큰 경우 성능 저하 발생 이유
- 문제 발생 원리
  - 데이터베이스는 OFFSET에 지정된 값만큼 데이터를 건너뛰기 위해
    - 결과 집합을 메모리에 로드하고,
    - 해당 범위의 데이터를 모두 스캔(Discard)
  - OFFSET의 값이 커질수록 건너뛰어야 하는 데이터가 많아져서 처리 시간이 급격히 증가함
- 예시
  ```
  // 처음 100,000개의 데이터를 스캔하여 버린 후, 100,001번째 데이터부터 반환
  // 데이터베이스는 100,000개의 데이터를 읽고 정렬한 후 버리므로 CPU와 메모리를 낭비
  SELECT *
  FROM products
  ORDER BY id
  LIMIT 10 OFFSET 100000;
  ```

### 2. 데이터 양에 따른 성능 차이 분석
- 소규모 데이터
  - OFFSET을 사용하는 페이징은 데이터 양이 적을 때 성능 문제가 거의 발생하지 않음.
  - 사용자가 페이지 이동을 하더라도 결과가 빠르게 반환.
- 대규모 데이터
  - 데이터 양이 많아질수록 OFFSET 기반의 페이징은 점점 비효율적이 됨.
  - 10,000번째 페이지를 요청하면 데이터베이스는 999,990개의 데이터를 스캔 후 폐기해야 하므로 성능이 급격히 저하.


### 4. 페이징 최적화 전략
#### 1. Keyset Pagination (커서 기반 페이징)
- Keyset Pagination은 OFFSET을 사용하지 않고,
- 이전 요청의 마지막 데이터 값을 기준으로 다음 데이터를 가져오는 방식
- 작동 방식
  - 특정 정렬 기준(Key)을 사용하여 이전 요청의 마지막 데이터를 기준으로 이후 데이터를 가져옴
  - OFFSET이 필요하지 않아 불필요한 데이터 스캔을 방지.
- 장단점
  - 장점
    - 대규모 데이터에서도 일정한 성능 유지.
    - 고정된 Key를 기반으로 필요한 데이터만 조회 가능.
  - 단점
    - 정렬 기준(Key) 설정이 복잡할 수 있으며, 페이지 번호 기반 탐색이 어려움.
- SQL
  ```
  -- 첫 번째 페이지 요청
  SELECT id, name, price, created_at
  FROM products
  ORDER BY id DESC 
  LIMIT 10; 
  
  -- 다음 페이지 요청
  SELECT id, name, price, created_at
  FROM products
  WHERE id < 마지막_id값 -- PK기반
  ORDER BY id DESC
  LIMIT 10;
  ```

#### 2. 인덱스를 활용한 페이징 성능 개선
- 정렬 및 검색 조건에 맞는 인덱스를 설계하여 페이징 성능을 최적화
- 복합 인덱스를 활용하면 쿼리의 검색과 정렬 성능을 대폭 개선할 수 있음
- SQL
  ```
  -- 검색 조건 (WHERE)과 정렬 조건 (ORDER BY)을 인덱스가 모두 활용
  CREATE INDEX idx_category_created_at ON products(category_id, created_at);
  
  SELECT id, name, price, created_at
  FROM products
  WHERE category_id = 100 -- 인덱스 컬럼1
  ORDER BY created_at DES C -- 인덱스 컬럼2 (꼭 where이 아니여도 order에도 인덱스 걸림)
  OFFSET 0
  LIMIT 10; -- LIMIT은 한 번에 가져올 데이터의 수를 제한하며, OFFSET은 결과의 시작 지점을 지정
  ```

#### 3. 필요한 데이터만 가져오는 최적화 기법
- 조회 시 필요한 컬럼만 SELECT하여 네트워크와 메모리 사용량을 줄임
- SQL 비교
  ```
  -- 모든 컬럼을 조회하는 비효율적인 쿼리
  SELECT *
  FROM products
  WHERE category_id = 100
  ORDER BY created_at DESC
  OFFSET 0
  LIMIT 10;
  
  -- 필요한 컬럼만 조회하여 최적화
  SELECT id, name, price, created_at
  FROM products
  WHERE category_id = 100
  ORDER BY created_at DESC
  OFFSET 0
  LIMIT 10;
  ```
