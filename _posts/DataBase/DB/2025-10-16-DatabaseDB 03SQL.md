---
title: 03 SQL
date: 2025-10-16 10:00:00 +09:00
categories: [Database, DB]
tags: [ Database, DB, SQL ]
---

## 1. SQL(Structured Query Language)
### 1. SQL
 - SQL은 관계형 데이터베이스(Relational Database) 에서 데이터를 다루기 위한 언어로,
 - 사용자는 SQL을 통해 데이터베이스에 질의(Query) 를 보내고,
 - 데이터베이스는 그 결과를 반환

### 2. SQL 주요 특징
 - 표준화된 언어
   - ANSI(미국표준협회)와 ISO(국제표준화기구)에서 표준화함.
   - MySQL, Oracle, PostgreSQL 등 거의 모든 DBMS에서 사용 가능 (약간의 방언 차이는 있음).
 - 선언형 언어
   - “어떻게”가 아니라 “무엇을” 하고 싶은지를 기술함.
   - 예: SELECT name FROM user WHERE age > 20;
 - 데이터 정의와 조작 모두 가능
   - 테이블 구조를 만드는 것(DDL)부터,
   - 데이터를 넣거나 수정하고 조회하는 것(DML)까지 하나의 언어로 처리 가능.
 - 관계형 모델 기반
   - 데이터를 행(Row)과 열(Column)로 구성된 테이블 형태로 관리.

### 3. SQL 종류

| 구분  | 명칭                           | 주요 기능     | 예시                             |
| --- | ---------------------------- | --------- | ------------------------------ |
| DDL | Data Definition Language     | 데이터 구조 정의 | CREATE, ALTER, DROP            |
| DML | Data Manipulation Language   | 데이터 조회·조작 | SELECT, INSERT, UPDATE, DELETE |
| DCL | Data Control Language        | 권한 제어     | GRANT, REVOKE                  |
| TCL | Transaction Control Language | 트랜잭션 제어   | COMMIT, ROLLBACK, SAVEPOINT    |

## 1. DDL
### 1. DDL (Data Definition Language)
 - DDL은 데이터 정의 언어로,
 - 데이터베이스의 구조(테이블, 스키마, 인덱스 등) 를 정의·변경·삭제하기 위한 SQL 명령어 집합

### 2. 특징

| 구분       | 설명                          |
| -------- | --------------------------- |
| 구조 중심 언어 | 테이블, 인덱스, 뷰 등 객체의 형태를 다룸    |
| 자동 커밋    | 실행 즉시 DB에 반영되며 ROLLBACK 불가능 |
| 트랜잭션 비지원 | DML과 달리 트랜잭션 제어 불가          |
| 데이터 미포함  | 데이터(내용)가 아니라 스키마(형태)를 조작    |
| 복구 불가    | DROP, TRUNCATE 후에는 복원 불가    |

### 3. 명령어 

| 명령어      | 기능        | 기본 형식                          | 비고               |
| -------- | --------- | ------------------------------ | ---------------- |
| CREATE   | 새 객체 생성   | `CREATE TABLE 테이블명 (...);`     | 테이블, 인덱스, 뷰 등 생성 |
| ALTER    | 기존 구조 변경  | `ALTER TABLE 테이블명 ADD 컬럼명 타입;` | 컬럼 추가·수정·삭제      |
| DROP     | 객체 삭제     | `DROP TABLE 테이블명;`             | 구조 자체 제거         |
| TRUNCATE | 데이터 전체 삭제 | `TRUNCATE TABLE 테이블명;`         | 구조는 유지, 데이터만 삭제  |
| RENAME   | 이름 변경     | `RENAME old_name TO new_name;` | 테이블명 변경          |


## 2. DML
### 1. DML (Data Manipulation Language)
 - DML은 데이터베이스 내의 데이터를 조회·추가·수정·삭제하기 위한 명령어 집합
 - 테이블 구조는 그대로 두고, 그 안의 실제 데이터를 다루는 언어

### 2. 특징

| 구분       | 설명                               |
| -------- | -------------------------------- |
| 트랜잭션 지원  | COMMIT, ROLLBACK을 통해 작업 단위 제어 가능 |
| 데이터 중심   | 테이블 구조가 아닌 ‘데이터’ 조작에 초점          |
| 복구 가능    | COMMIT 전에는 ROLLBACK으로 원상복구 가능    |
| 가장 많이 사용 | 실제 애플리케이션에서 가장 자주 사용되는 SQL 유형    |

### 3. 명령어
#### 1. 기본명령어

| 명령어    | 기능     | 기본 형식                                         | 비고                  |
| ------ | ------ | --------------------------------------------- | ------------------- |
| SELECT | 데이터 조회 | `SELECT 컬럼 FROM 테이블 WHERE 조건;`                | 조회 전용, 원본 데이터 변경 없음 |
| INSERT | 데이터 삽입 | `INSERT INTO 테이블 (컬럼1, 컬럼2) VALUES (값1, 값2);` | 새 행(Row) 추가         |
| UPDATE | 데이터 수정 | `UPDATE 테이블 SET 컬럼=값 WHERE 조건;`               | 조건 없으면 전체 수정 주의     |
| DELETE | 데이터 삭제 | `DELETE FROM 테이블 WHERE 조건;`                   | 조건 없으면 전체 삭제 주의     |

## 3. DCL
### 1. DCL (Data Control Language)
 - 데이터베이스 사용자에게 권한을 부여하거나 회수하는 명령어 집합
 - 누가 어떤 데이터에 접근할 수 있는지를 제어하는 언어

### 2. 특징 

| 구분              | 설명                        |
| --------------- | ------------------------- |
| 권한 제어 언어        | 사용자에게 데이터 접근 권한을 부여하거나 회수 |
| 보안 관련           | 데이터 무단 접근이나 조작을 방지        |
| DDL과 유사하게 자동 커밋 | 실행 즉시 반영, ROLLBACK 불가능    |
| 데이터 조작 불가       | 데이터 내용은 변경하지 않음           |
| DBA(관리자) 중심 사용  | 일반 사용자보다는 관리자 계정에서 주로 사용  |

### 3. 명령어
#### 1. 기본 명령어

| 명령어    | 기능    | 기본 형식                       | 비고                                       |
| ------ | ----- | --------------------------- | ---------------------------------------- |
| GRANT  | 권한 부여 | `GRANT 권한 ON 객체 TO 사용자;`    | 예: `GRANT SELECT ON member TO user1;`    |
| REVOKE | 권한 회수 | `REVOKE 권한 ON 객체 FROM 사용자;` | 예: `REVOKE SELECT ON member FROM user1;` |

#### 2. 권한 종류

| 권한         | 설명            |
| ---------- | ------------- |
| SELECT     | 데이터 조회 권한     |
| INSERT     | 데이터 삽입 권한     |
| UPDATE     | 데이터 수정 권한     |
| DELETE     | 데이터 삭제 권한     |
| ALL        | 모든 권한 부여      |
| REFERENCES | 외래 키 참조 권한    |
| EXECUTE    | 프로시저/함수 실행 권한 |

## 4. TCL
### 1. TCL (Transaction Control Language)
 - 트랜잭션(Transaction) 을 제어하기 위한 SQL 명령어 집합으로,
 - 데이터 변경 작업(DML)의 처리 결과를 확정하거나 되돌리는 역할

### 2. 특징

| 구분           | 설명                                           |
| ------------ | -------------------------------------------- |
| DML과 함께 사용   | SELECT, INSERT, UPDATE, DELETE 등 데이터 조작 시 적용 |
| 일관성 보장       | 중간 단계에서 오류 발생 시 데이터 불일치 방지                   |
| 수동 커밋 가능     | 자동 커밋 모드를 해제하면 명시적으로 제어 가능                   |
| 복구 기능        | 잘못된 변경은 ROLLBACK으로 원상 복구                     |
| SAVEPOINT 지원 | 중간 저장 지점을 만들어 부분 롤백 가능                       |

### 3. 명령어

| 명령어         | 기능          | 기본 형식               | 비고                   |
| ----------- | ----------- | ------------------- | -------------------- |
| COMMIT      | 트랜잭션 확정     | `COMMIT;`           | 변경 사항을 DB에 영구 반영     |
| ROLLBACK    | 트랜잭션 취소     | `ROLLBACK;`         | 마지막 COMMIT 이전 상태로 복구 |
| SAVEPOINT   | 중간 저장 지점 설정 | `SAVEPOINT 저장점명;`   | 특정 지점까지만 되돌릴 때 사용    |
| ROLLBACK TO | 특정 지점까지 복구  | `ROLLBACK TO 저장점명;` | SAVEPOINT와 함께 사용     |


## 5. Join

### 1. Join
- 두 개 이상의 테이블을 공통된 속성(컬럼)을 기준으로 연결하여, 하나의 결과 집합(Result Set)으로 만드는 관계형 연산
- Join은 DML은 아니지만 SELECT 문 안에서 사용하는 연산자(연산식)

### 2. 종류

| 종류                            | 설명                              | 예시                                               |
| ----------------------------- | ------------------------------- | ------------------------------------------------ |
| INNER JOIN                    | 두 테이블 모두 조건을 만족하는 행만            | `SELECT * FROM A INNER JOIN B ON A.id = B.a_id;` |
| LEFT JOIN (LEFT OUTER JOIN)   | 왼쪽 테이블 기준, 조건 일치 여부와 상관없이 모든 행  | `SELECT * FROM A LEFT JOIN B ON A.id = B.a_id;`  |
| RIGHT JOIN (RIGHT OUTER JOIN) | 오른쪽 테이블 기준, 조건 일치 여부와 상관없이 모든 행 | `SELECT * FROM A RIGHT JOIN B ON A.id = B.a_id;` |
| FULL JOIN (FULL OUTER JOIN)   | 양쪽 테이블 모두, 조건 일치 여부와 상관없이 모든 행  | `SELECT * FROM A FULL JOIN B ON A.id = B.a_id;`  |
| CROSS JOIN                    | 모든 조합(Cartesian product)        | `SELECT * FROM A CROSS JOIN B;`                  |

### 3. oracle - join
#### 1. 표준 SQL(ANSI) vs Oracle 전용 문법
  - Oracle은 예전부터 자체 전용 조인 문법(Oracle Join Syntax)을 사용함.
  - 나중에 ANSI 표준 SQL JOIN 문법도 지원하게 됨.

#### 2. Oracle 전용 Join
 
   | 구분             | ANSI 표준 SQL                                    | Oracle 전용 문법 (옛 방식)                        |
   | -------------- | ---------------------------------------------- | ------------------------------------------ |
   | INNER JOIN | `SELECT  FROM A INNER JOIN B ON A.id = B.id;` | `SELECT  FROM A, B WHERE A.id = B.id;`    |
   | LEFT JOIN  | `SELECT  FROM A LEFT JOIN B ON A.id = B.id;`  | `SELECT  FROM A, B WHERE A.id = B.id(+);` |
   | RIGHT JOIN | `SELECT  FROM A RIGHT JOIN B ON A.id = B.id;` | `SELECT  FROM A, B WHERE A.id(+) = B.id;` |
   | FULL JOIN  | `SELECT  FROM A FULL JOIN B ON A.id = B.id;`  | 지원 안 함 (UNION으로 대체해야 함)                    |

#### 3. 비교
 - 오라클
    ```
    SELECT e.name, d.dept_name
    FROM employee e, department d
    WHERE e.dept_id = d.dept_id(+);
    ```
 - ANSI표준
   ```
   SELECT e.name, d.dept_name
   FROM employee e
   LEFT JOIN department d
   ON e.dept_id = d.dept_id;
   ```
