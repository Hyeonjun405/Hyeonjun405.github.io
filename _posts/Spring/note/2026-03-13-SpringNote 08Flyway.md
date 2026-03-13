---
title: 08 Flyway
date: 2026-03-13 10:00:00 +09:00
categories: [00Spring Framework, SpringNote]
tags: [ Java, Spring Framework, Spring Note ]
---

## 1. Note
### 1. Flyway
 - 간단히 표현하면, 깃 처럼 SQL을 관리해서 공통된 쿼리를 사용해서 테이블을 관리하도록 하는 Lib
 - Local에서 개발자마다 DB를 두고 사용한다면
   - 연결되지 않은 다른 컴퓨터에서도 table이 일정하게 유지됨
   - V버전만 잘맞추면 다른 누군가 테이블을 생성해도
   - 차이없이 모두 공통 스키마를 가짐.
 - Local 에서 DEV DB를 연결해서 사용한다고 하면
   - 상대적으로 필요성이 굉장히 떨어지기는 하지만,
   - DEV에서 형상변경을 어떻게 했는지 파악하는 용도 또는
   - 누가 이걸 언제 추가했는지 확인하는 용도 등 활용가능함.
   - 그리고 DEV에 반영한거 정리해서 PRD에 바로 반영하기에는 이 방법이 현명해보임.
 - 보통 PRD에서는 사용하면 안되기 때문에
   - 환경별 프로퍼티스에서 실행여부(T/F) 처리를 해서 안하도록
   - PRD도 DEV와 공통적으로 가져가기는 하지만,
   - 일반적으로 별도로 확인함..!

### 2. JPA & Flyway
 - JPA를 사용하는 경우에는
   - JPA가 엔티티생성 과정에서 테이블을 생성하는 설정이 있음.
   - 이때 Flyway에 버전관리가 되지 않는 테이블이 생성될 경우 충돌 / 오류가 날수도 있음.
   - 따라서 JPA에서 생성하지 않도록 변경 필요 `spring.jpa.hibernate.ddl-auto=create`
 - 작업의 순서
  ```
   1 Entity 변경
   2 migration SQL 작성
   3 애플리케이션 실행 
   4 validate로 구조 검증 # 여기서 테이블 생성
  ```
 
## 2. Flyway
### 1. Flyway
 - 데이터베이스 스키마 버전 관리(DB Migration) 라이브러리
 - DB 구조 변경을 코드처럼 관리하는 도구
 - 자동으로 반영해서 DB 전체의 통일성을 맞춰줌
 - DB에 migration 이력을 기록하는 테이블이 자동으로 생성
 - 기본적은로 `DataSource`에 등록된 DB 바라보지만, 다른걸로도 변경가능함.

### 2. 사용하는 포인트
 - 개발자마다 DB 상태가 다름
 - 배포할 때 어떤 SQL을 먼저 실행해야 하는지 헷갈림
 - 운영/개발 DB 스키마가 서로 달라짐

### 3. 흐름
  ```
  Spring Boot start
  → Flyway 실행
  → DB migration
  → Application start
  ```

### 4. 이력 테이블 구조
  
  | installed_rank | version | description        | type | script                     | success |
  | -------------- | ------- | ------------------ | ---- | -------------------------- | ------- |
  | 1              | 1       | create_user_table  | SQL  | V1__create_user_table.sql  | true    |
  | 2              | 2       | add_email_column   | SQL  | V2__add_email_column.sql   | true    |
  | 3              | 3       | create_order_table | SQL  | V3__create_order_table.sql | true    |

 - success가 false가 뜨면 테이블 내의 row지우고 다시 반영하는게 현명함.
 - 어쨋든 DEV환경이고 중요한 포인트가 아니면 테스트 이력을 남겨둘 이유가 없음.
 - false는 보통 SQL 오류니까.

## 3. Flyway 사용법
### 1. 의존성
  ```
  implementation 'org.flywaydb:flyway-core'
  ```

### 2. 패턴
 - 파일 경로
  ```
  src/main/resources/db/migration
  ```
 - 파일 패턴
  ```
  V1__create_user_table.sql
  V2__add_email_column.sql
  V3__create_order_table.sql
  ```
   - 파일명 : 패턴 V1__{이름}.sql
   - memo
     - 언더바 2개
     - V{number} 버전 순서대로 반영됨.
  
### 3. 프로퍼티스 설정
#### 1. 기본 동작

| 설정                           | 의미                         | 예시 값                   |
| ---------------------------- |----------------------------| ---------------------- |
| spring.flyway.enabled        | Flyway 실행 여부(PRD는 false처리) | true                   |
| spring.flyway.locations      | migration SQL 파일 경로        | classpath:db/migration |
| spring.flyway.table          | migration 이력 테이블 이름        | flyway_schema_history  |
| spring.flyway.schemas        | 사용할 DB 스키마                 | app_schema             |
| spring.flyway.default-schema | 기본 스키마                     | public                 |

#### 2. 실행 제어 옵션

| 설정                                 | 의미                          | 예시 값             |
| ---------------------------------- | --------------------------- | ---------------- |
| spring.flyway.baseline-on-migrate  | 기존 DB에 Flyway 도입 시 기준 버전 생성 | true             |
| spring.flyway.baseline-version     | 시작 버전                       | 1                |
| spring.flyway.baseline-description | baseline 설명                 | initial baseline |
| spring.flyway.validate-on-migrate  | migration 파일 검증             | true             |
| spring.flyway.clean-disabled       | clean 명령 금지                 | true             |

#### 3. SQL 파일 관련

| 설정                                            | 의미                  | 예시 값  |
| --------------------------------------------- | ------------------- | ----- |
| spring.flyway.sql-migration-prefix            | 버전 migration prefix | V     |
| spring.flyway.repeatable-sql-migration-prefix | 반복 migration prefix | R     |
| spring.flyway.sql-migration-separator         | 파일명 구분자             | __    |
| spring.flyway.sql-migration-suffixes          | SQL 파일 확장자          | .sql  |
| spring.flyway.encoding                        | SQL 파일 인코딩          | UTF-8 |

#### 4. 디비 연결설정

| 설정                              | 의미               | 예시 값                                   |
| ------------------------------- | ---------------- | -------------------------------------- |
| spring.flyway.url               | Flyway 전용 DB URL | jdbc:postgresql://localhost:5432/appdb |
| spring.flyway.user              | DB 사용자           | app_user                               |
| spring.flyway.password          | DB 비밀번호          | app_password                           |
| spring.flyway.driver-class-name | JDBC 드라이버        | org.postgresql.Driver                  |

#### 5. 실행전략

| 설정                                      | 의미                       | 예시 값  |
| --------------------------------------- | ------------------------ | ----- |
| spring.flyway.group                     | migration을 하나의 트랜잭션으로 실행 | true  |
| spring.flyway.out-of-order              | 버전 순서 어긋나도 실행            | false |
| spring.flyway.mixed                     | SQL + Java migration 혼용  | false |
| spring.flyway.ignore-missing-migrations | 누락 migration 무시          | false |
