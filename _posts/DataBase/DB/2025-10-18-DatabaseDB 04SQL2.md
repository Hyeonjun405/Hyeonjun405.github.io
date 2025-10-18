---
title: 04 SQL2
date: 2025-10-18 10:00:00 +09:00
categories: [Database, DB]
tags: [ Database, DB, SQL, 내장함수 ]
---

## 1. 내장함수
### 1. 내장함수
 - SQL 엔진이 기본적으로 제공하는 데이터 조작용 함수로, 별도의 사용자 정의 없이 바로 사용할 수 있는 기능
 - 데이터를 변환, 요약, 비교, 형식 변경 등에 사용

### 2. 내장함수 분류
#### 1. 구분
  
  | 구분                                             | 설명                      | 예시                                  |
  | ---------------------------------------------- | ----------------------- | ----------------------------------- |
  | 단일행 함수 (Single-row function)               | 각 행(row)마다 독립적으로 적용     | UPPER(), LENGTH(), ROUND(), NVL()   |
  | 집계 함수 (Group function, Aggregate function) | 여러 행(row)을 묶어 하나의 결과 반환 | SUM(), AVG(), COUNT(), MAX(), MIN() |

#### 2. 구분 별 함수
  
  | 구분     | 카테고리    | 함수 종류                                                                   |
  | ------ | ------- | ----------------------------------------------------------------------- |
  | 단일행 함수 | 문자열     | UPPER, LOWER, LENGTH, SUBSTR, CONCAT, INSTR, TRIM                       |
  |        | 숫자      | ABS, ROUND, CEIL, FLOOR, MOD, POWER                                     |
  |        | 날짜/시간   | CURRENT_DATE, CURRENT_TIMESTAMP, SYSDATE, EXTRACT, ADD_MONTHS, DATEDIFF |
  |        | 형 변환    | CAST, TO_CHAR, TO_NUMBER, TO_DATE                                       |
  |        | NULL 처리 | NVL, COALESCE, NULLIF, ISNULL                                           |
  | 집계 함수  | 요약/통계   | COUNT, SUM, AVG, MIN, MAX                                               |

#### 3. Note
- 집계함수 Nulll 처리 구분

  | 연산 방식        | 같은 컬럼 (집계)                            | 다른 컬럼 (연산)                                    |
  | ------------ | ------------------------------------- | --------------------------------------------- |
  | NULL 포함 시 결과 | NULL 무시                               | NULL 있으면 결과 NULL                              |
  | SUM          | SUM(column) → 10 + NULL + 20 = 30     | column1 + column2 → 10 + NULL = NULL          |
  | AVG          | AVG(column) → (10 + 20)/2 = 15        | column1 - column2 → 10 - NULL = NULL          |
  | MIN          | MIN(column) → 5                       | column1 * column2 → 10 * NULL = NULL          |
  | MAX          | MAX(column) → 20                      | column1 / column2 → 10 / NULL = NULL          |
  | COUNT        | COUNT(column) → NULL 제외, 값 있는 행만 계산   | COUNT(*) → 모든 행 계산                            |
  | NULL을 0으로 처리 | SUM(NVL(column,0)) → 10 + 0 + 20 = 30 | NVL(column1,0) + NVL(column2,0) → 10 + 0 = 10 |

- PostgreSQL - ORacle - Mysql
  
  | 설명                          | PostgreSQL                         | Oracle                                | MySQL |
  |-------------------------------|-----------------------------------|--------------------------------------|-------|
  | NULL일 때 대체값 반환           | COALESCE(expr1, expr2)            | NVL(expr1, expr2)                     | IFNULL(expr1, expr2) |
  | 조건문 (if문처럼 분기)          | CASE WHEN condition THEN val1 ELSE val2 END | DECODE(expr, val1, res1, val2, res2, ...) | IF(condition, true_val, false_val) |
  | 현재 날짜·시간 반환              | CURRENT_TIMESTAMP / NOW()         | SYSDATE                               | NOW() / CURDATE() |
  | 날짜 형식 변환 (문자 → 날짜)    | TO_DATE(str, format)              | TO_DATE(str, format)                   | STR_TO_DATE(str, format) |
  | 날짜 형식 변환 (날짜 → 문자)    | TO_CHAR(date, format)             | TO_CHAR(date, format)                  | DATE_FORMAT(date, format) |
  | 문자열 길이 반환                | LENGTH(str)                       | LENGTH(str)                            | CHAR_LENGTH(str) / LENGTH(str) |
  | 문자열 연결                     | || / CONCAT(str1, str2)           | || / CONCAT(str1, str2)                | CONCAT(str1, str2) |
  | 반올림                           | ROUND(num [, n])                  | ROUND(num [, n])                        | ROUND(num [, n]) |
  | 버림                             | TRUNC(num [, n])                  | TRUNC(num [, n])                        | TRUNCATE(num, n) |
  | 올림                             | CEIL(num)                          | CEIL(num)                               | CEILING(num) |
  | 나머지 계산                       | MOD(a, b) / a % b                 | MOD(a, b)                              | MOD(a, b) / a % b |
  | 현재 날짜만 반환                  | CURRENT_DATE                      | TRUNC(SYSDATE)                          | CURDATE() |
  | 행 제한 / 순번 기능               | LIMIT n / OFFSET n                | ROWNUM                                  | LIMIT n |
  | NULL 아닌 첫 번째 값 반환         | COALESCE(expr1, expr2, ...)      | COALESCE(expr1, expr2, ...)            | COALESCE(expr1, expr2, ...) |
  | 데이터 형 변환                    | CAST(expr AS type)                | TO_NUMBER(expr) / TO_CHAR(expr)        | CAST(expr AS type) |
  | 랜덤 값 생성                     | RANDOM()                          | DBMS_RANDOM.VALUE                        | RAND() |


## 2. 부속질의
### 1. 부속질의
 - 다른 SQL 쿼리 안에 포함된 쿼리
 - 메인 쿼리에서 조건, 값, 임시 테이블 등을 제공
 - 일반적으로 안쪽부터 실행 → 바깥쪽 메인 쿼리에 전달

### 2. 부속질의 종류
 
  | 종류       | 반환 형태       | 특징                   | 예시 형태                                                             |
  | -------- | ----------- | -------------------- | ----------------------------------------------------------------- |
  | 단일행 부속질의 | 1행 1열 이상    | 조건 비교용, 1행 1열이면 스칼라  | `WHERE col = (SELECT col FROM table WHERE ...)`                   |
  | 다중행 부속질의 | 여러 행 1열     | IN, ANY, ALL 조건에서 사용 | `WHERE col IN (SELECT col FROM table WHERE ...)`                  |
  | 다중열 부속질의 | 1행 이상, 여러 열 | 복합 컬럼 비교용            | `WHERE (col1, col2) = (SELECT col1, col2 FROM table ...)`         |
  | 상관 부속질의  | 메인 쿼리 행별 반복 | 메인 쿼리 컬럼 참조          | `WHERE col > (SELECT AVG(col) FROM table t2 WHERE t2.id = t1.id)` |
  
  - 스칼라쿼리 : 1행 1열(= 단일값)을 반환하는 부속질의
  - 인라인 뷰 (Inline View) : FROM 절에 들어가는 부속질의 전체

### 3. 위치 기준 분류
#### 1. where절 부속질의
 - 용도: 조건 비교용
 - 사용 가능 형태
   - 단일행 부속질의 → = 등으로 비교
   - 다중행 부속질의 → IN, ANY, ALL과 함께 비교
 - 특징
   - 메인 쿼리의 조건에 따라 부속질의가 한 번 또는 행별로 실행
   - 상관 부속질의라면 메인 쿼리의 각 행 값을 참조해 반복 실행
 - 형태
   ```
   WHERE salary > ANY (SELECT salary FROM emp WHERE dept_id = 20)
   ```
#### 2. SELECT 절 부속질의
 - 용도: 각 행별 계산 결과를 컬럼처럼 반환
 - 사용 가능 형태
    - 단일값(스칼라)만 가능
    - 다중값/다중열은 오류 발생
 - 특징
    - 메인 쿼리의 각 행에 대해 부속질의가 실행
    - 계산 결과를 새로운 컬럼으로 보여주는 용도
 - 형태
   ```
   SELECT emp_id,
       (SELECT AVG(salary) FROM emp WHERE dept_id = e.dept_id) AS avg_salary
   FROM emp e;
   ```

#### 3. FROM 절 부속질의 (인라인 뷰)
 - 용도: 임시 테이블처럼 사용
 - 사용 가능 형태
    - 여러 행, 여러 열 가능
    - 조인, 그룹화, 집계 자유롭게 가능
 - 특징
    - 메인 쿼리에서 마치 테이블처럼 참조 가능
    - 결과를 별칭(alias)으로 지정해야 함
 - 형태
   ```
   SELECT dept_id, avg_salary
   FROM (
     SELECT dept_id, AVG(salary) AS avg_salary
     FROM emp
     GROUP BY dept_id
   ) AS dept_avg;
   ```
   
### 4. Note
 - 부속질의 종류는 행태기준으로 단일행 부속질의/다중행 부속질의/다중열 부속질의/상관 부속질의
 - 단일열/단일행으로 쓰면 스칼라 쿼리
 - From절 위치에서 쓰이는 모든 부속질의는 인라인뷰
 - 나머지 위치에서는 부속질의 행태에 따라서 종류가 결정됨.

## 4. 뷰
### 1. View
 - 실제 데이터를 직접 저장하지 않고,SELECT 쿼리 결과를 가상의 테이블처럼 정의한 객체
 - 뷰 = “SELECT 문을 이름 붙여서 저장한 가짜 테이블”
 - 내부적으로는 항상 원본 테이블(기본 테이블, Base Table) 을 참조
 - 실행 시마다 SELECT문이 수행되므로 대량 데이터 조회 시 느릴 수 있음

### 2. View의 예시
 - 생성
   ```
    CREATE VIEW emp_dept_view AS
    SELECT e.emp_id, e.emp_name, d.dept_name
    FROM emp e
    JOIN dept d ON e.dept_id = d.dept_id;
   ```
 - 사용
   ```
    SELECT * FROM emp_dept_view;
   ```

### 3. View의 종류

| 종류                 | 설명                         | 예시                                                                                             |
| ------------------ | -------------------------- | ---------------------------------------------------------------------------------------------- |
| 단순 뷰(Simple View)  | 한 개의 테이블 기반                | `CREATE VIEW emp_view AS SELECT empno, ename FROM emp;`                                        |
| 복합 뷰(Complex View) | 여러 테이블 JOIN, GROUP BY 등 포함 | `CREATE VIEW emp_dept AS SELECT e.ename, d.dname FROM emp e JOIN dept d ON e.deptno=d.deptno;` |
| 인라인 뷰(Inline View) | SQL문 안에서 직접 정의된 뷰 (일시적)    | `SELECT * FROM (SELECT deptno, AVG(sal) FROM emp GROUP BY deptno) tmp;`                        |

### 4. View 관련 명령어

| 명령어          | 설명                   | 예시                                               |
| ------------ | -------------------- | ------------------------------------------------ |
| CREATE VIEW  | 뷰 생성                 | `CREATE VIEW view_name AS SELECT ...`            |
| DROP VIEW    | 뷰 삭제                 | `DROP VIEW view_name;`                           |
| REPLACE VIEW | 기존 뷰 수정(덮어쓰기)        | `CREATE OR REPLACE VIEW view_name AS SELECT ...` |
| ALTER VIEW   | 일부 DBMS에서 뷰 속성 수정 가능 | `ALTER VIEW view_name COMPILE;` (Oracle)         |

## 3. 인덱스
### 1. 인덱스
 - 테이블의 검색 속도를 빠르게 하기 위해 사용하는 자료 구조
 - 책의 색인처럼, 원하는 데이터를 바로 찾아갈 수 있게 만들어주는 것
 - 테이블에 있는 데이터 자체를 바꾸는 것이 아니라, 특정 컬럼 값과 그 위치 정보를 별도로 저장해서 조회를 빠르게 함.

### 2. 인덱스의 장단점
 - 장점
    - 검색 속도 매우 빠름 (특히 WHERE, JOIN, ORDER BY, GROUP BY)
    - 중복 방지, 유니크 제약 조건 지원
 - 단점
    - INSERT/UPDATE/DELETE 성능 저하 (인덱스도 같이 갱신되어야 함)
    - 많은 인덱스 생성 시 저장 공간 증가
    - 잘못된 인덱스 설계는 성능 오히려 저하 가능

### 3. 인덱스 종류

| 종류                            | 설명                                                              |
| ----------------------------- | --------------------------------------------------------------- |
| 단일 컬럼 인덱스                 | 한 컬럼만 기준으로 생성                                                   |
| 복합 컬럼 인덱스                 | 두 개 이상의 컬럼을 묶어서 생성 (JOIN이나 WHERE 조건 복합 조회에 유리)                  |
| 유니크 인덱스                   | 중복 값을 허용하지 않는 인덱스 (Primary Key와 동일한 역할 가능)                      |
| 클러스터형 인덱스(Clustered)      | 테이블 데이터를 인덱스 순서대로 저장 (Oracle/SQL Server)                        |
| 논클러스터형 인덱스(Non-Clustered) | 별도의 인덱스만 생성, 데이터 순서는 테이블과 별개 (Oracle: 일반 인덱스, PostgreSQL/MySQL) |
| 부분 인덱스(Partial/Filtered)  | 조건을 만족하는 일부 데이터만 인덱싱 (PostgreSQL, SQL Server)                   |
| 함수 기반 인덱스(Function-based) | 컬럼에 함수를 적용한 결과를 기준으로 인덱스 생성 (Oracle, PostgreSQL)                |

### 3. 인덱스 사용
- 생성

  ```
  -- 단일 컬럼 인덱스
  CREATE INDEX idx_emp_name ON emp(ename);
  
  -- 복합 컬럼 인덱스
  CREATE INDEX idx_emp_dept_sal ON emp(deptno, sal);
  
  -- 유니크 인덱스
  CREATE UNIQUE INDEX idx_emp_id ON emp(empno);
  
  -- 함수 기반 인덱스 (PostgreSQL)
  CREATE INDEX idx_lower_name ON emp(LOWER(ename));
  ```

- 삭제 

  ```
  -- Oracle / PostgreSQL: 
  DROP INDEX index_name;
  -- MySQL
  DROP INDEX index_name ON table_name;
  ```
