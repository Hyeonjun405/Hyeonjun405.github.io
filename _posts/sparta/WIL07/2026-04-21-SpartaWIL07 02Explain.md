---
title: 02 Explain
date: 2026-04-21 10:00:00 +09:00
categories: [Sparta, SpartaWIL07]
tags: [ Java, Spring Framework ]
---

## 1. Note
- 어떻게 데이터를 빠르고 정확하게 조회할 것인가?
- 조회 성능을 향상시키기 위해서
  - 인덱스를 거는건 좋은데, 난무해도 문제고
  - 너무 넓은 범위를 조회하게 하면 속도가 느려짐.
  - 단순히 백엔드 개발자의 문제가 아니라 
  - 여러가지를 고려하고 판단해야하는 문제! 

## 2. Explain
### 1. Explain
- 쿼리가 실제로 어떻게 실행될지 “미리 실행 계획을 보여주는 기능
- 쿼리를 실행할때 내부적으로 어떻게 조회할지 보여주는 것
  - 어떤 인덱스를 사용할지 
  - 테이블을 어떤 순서로 조인할지
  - 풀 스캔 할지, 인덱스 스캔 할지 

### 2. 흐름
```
EXPLAIN SELECT * FROM user WHERE id = 10;

-- 앞에 EXPLAIN 표기하면 
-- 1. type 조회 방식 (ALL = 풀스캔, index, range, ref 등)
-- 2. key	사용된 인덱스
-- 3. rows	예상 조회 row 수
-- 4. Extra	추가 정보 (Using index, Using where 등)
```

### 3. 얻을 수 있는 부분
#### 1. 왜 쿼리가 느린지 “정확히” 알 수 있음
- EXPLAIN을 보면 확인 가능한 것
  - 풀스캔인지 (type = ALL)
  - 인덱스를 타는지 (key)
  - 몇 건을 읽는지 (rows)
- 이 데이터를 통해서 어떻게 조회해서 이런 조회 속도가 나오는지 등 확인이 가능함.
  
#### 2. 인덱스가 실제로 먹히는지 검증 가능
- EXPLAIN의
  - key = null → 인덱스 안 씀
  - type = ALL → 풀스캔
- 인덱스 만들었는데 효과가 없다면 Explain을 통해서 이유를 알 수 있음

#### 3. JOIN 성능 문제 파악 가능
- JOIN이 느릴 때 EXPLAIN을 확인 가능한 부분
  - 어떤 테이블부터 읽는지
  - 조인 순서
  - 어떤 인덱스 사용하는지
- 발견 가능한 문제들
  - N+1 문제
  - 잘못된 조인 순서
  - 불필요한 테이블 스캔

#### 4. “튜닝 전 vs 튜닝 후” 비교 가능
```
EXPLAIN 실행 → rows = 100만
인덱스 추가
다시 EXPLAIN → rows = 100
```

## 3.Explain 결과
### 1. Mysql(테이블 형태)

| id | type  | key      | rows | Extra       |
| -- | ----- | -------- | ---- | ----------- |
| 1  | range | idx_name | 100  | Using where |


### 2. Oracle(실행 순서가 트리 구조로 나옴)

| Id | Operation         | Name     | Rows | Cost |
| -- | ----------------- | -------- | ---- | ---- |
| 0  | SELECT STATEMENT  |          |      |      |
| 1  | TABLE ACCESS FULL | USERS    | 1000 | 10   |
| 2  | INDEX RANGE SCAN  | IDX_NAME | 100  | 5    |


### 3. PostgreSQL
- 결과값
  ```
  Seq Scan on categories  (cost=0.00..1.81 rows=4 width=69)
    Filter: (id < 10)
  ```
  
- 테이블 형태로 변형 (별도로 해석 필요함.)

  | Node Type | Relation   | Index | Cost       | Rows | Width | Filter  | Extra |
  | --------- | ---------- | ----- | ---------- | ---- | ----- | ------- | ----- |
  | Seq Scan  | categories | -     | 0.00..1.81 | 4    | 69    | id < 10 | -     |


## 4. Explain의 정보
### 1. Type(조회 방식)

| 개념         | MySQL  | Oracle                    | PostgreSQL |
| ---------- | ------ | ------------------------- | ---------- |
| 전체 테이블 스캔  | ALL    | TABLE ACCESS FULL         | Seq Scan   |
| 인덱스 전체 스캔  | index  | INDEX FULL SCAN           | Index Scan |
| 인덱스 범위 스캔  | range  | INDEX RANGE SCAN          | Index Scan |
| 인덱스 단건 조회  | ref    | INDEX RANGE / UNIQUE SCAN | Index Scan |
| PK 단건 조회   | const  | INDEX UNIQUE SCAN         | Index Scan |
| 조인 (PK 기준) | eq_ref | INDEX UNIQUE SCAN         | Index Scan |


### 2. key (사용된 인덱스)

| 개념      | MySQL      | Oracle            | PostgreSQL                     |
| ------- | ---------- | ----------------- | ------------------------------ |
| 사용된 인덱스 | key        | INDEX             | Index Scan / Bitmap Index Scan |
| PK 사용   | PRIMARY    | INDEX UNIQUE SCAN | Index Scan (PK 포함)             |
| 인덱스 없음  | key = null | TABLE ACCESS FULL | Seq Scan                       |

### 3. rows (예상 조회 row 수)

| 개념       | MySQL   | Oracle | PostgreSQL |
| -------- | ------- | ------ | ---------- |
| 예상 row 수 | rows    | Rows   | rows       |
| 비용       | 없음 (간접) | Cost   | cost       |
| row 크기   | 없음      | Bytes  | width      |

### 4. Extra (추가 정보)

| 개념       | MySQL (Extra)   | Oracle          | PostgreSQL         |
| -------- | --------------- | --------------- | ------------------ |
| WHERE 필터 | Using where     | FILTER          | Filter             |
| 정렬 발생    | Using filesort  | SORT ORDER BY   | Sort               |
| 임시 테이블   | Using temporary | TEMP TABLE      | Hash / Materialize |
| 커버링 인덱스  | Using index     | INDEX ONLY SCAN | Index Only Scan    |

### 5. 조인방식

| 개념          | MySQL | Oracle       | PostgreSQL  |
| ----------- | ----- | ------------ | ----------- |
| Nested Loop | 기본    | NESTED LOOPS | Nested Loop |
| Hash Join   | 제한적   | HASH JOIN    | Hash Join   |
| Merge Join  | 없음    | MERGE JOIN   | Merge Join  |
