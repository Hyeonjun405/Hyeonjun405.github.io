---
title: 02 Data and Object Patterns
date: 2025-09-24 10:00:00 +09:00
categories: [Java, JavaBase]
tags: [Java]
---

## 1. 데이터 전달 / 저장 객체

| 종류                             | 역할 / 특징                          | Servlet 환경                        | Spring 환경                               |
| ------------------------------ | -------------------------------- | --------------------------------- | --------------------------------------- |
| VO (Value Object)              | 읽기 중심, DB 데이터 값 객체               | DB 조회 후 HttpServletRequest에 담아 전달 | DB 조회 후 Model/DTO로 전달 가능                |
| DTO (Data Transfer Object)     | 계층 간 데이터 전달, 읽기/쓰기 가능            | 요청 파라미터 묶음, JSP로 전달               | Controller ↔ Service ↔ Repository 사이 전달 |
| Entity                        | DB 테이블과 1:1 매핑, ORM 관리 대상        | JDBC 직접 접근 시 사용 빈도 낮음             | JPA/Hibernate @Entity, 영속성 관리           |
| JavaBean                       | getter/setter, 기본 생성자, 재사용 객체 규약 | HttpSession, JSP EL에서 활용          | ModelAttribute, Form 객체 등               |
| Form / Command Object | 사용자 입력 데이터 수집                    | 요청 파라미터 수집용                       | @ModelAttribute, Form 객체로 바인딩           |


## 2. 데이터 접근/비즈니스 객체

| 종류                | 역할 / 특징                            | Servlet 환경                                  | Spring 환경                        |
| ----------------- | ---------------------------------- | ------------------------------------------- | -------------------------------- |
| DAO (Data Access Object) | DB CRUD 처리, SQL 실행                 | JDBC 직접 사용, Connection/PreparedStatement 관리 | Repository로 대체, JPA/Hibernate 사용 |
| Repository        | DAO 역할 + ORM 지원, SQL/객체 매핑         | 거의 없음                                       | Spring Data JPA, MyBatis Mapper  |
| BO (Business Object) | 비즈니스 로직 담당, 트랜잭션 처리 가능             | Service 객체로 구현 가능                           | Service 계층에서 비즈니스 로직 처리          |
| Service          | Controller와 Repository 사이, 트랜잭션 관리 | Servlet에서 직접 로직 처리 가능                       | @Service로 계층화, 트랜잭션 관리           |

## 3. 기타 객체/패턴

| 종류          | 역할 / 특징            | Servlet 환경           | Spring 환경              |
| ----------- | ------------------ | -------------------- | ---------------------- |
| Mapper      | 객체 ↔ DB 매핑, SQL 정의 | MyBatis 사용 시         | MyBatis Mapper 인터페이스   |
| Singleton   | 애플리케이션 단일 객체 유지    | Application Scope 객체 | Spring Bean Scope 싱글톤  |
| Builder / Factory | 생성 책임 분리, 가독성 향상   | 객체 생성 패턴             | 복잡한 DTO/VO/Entity 생성 시 |
