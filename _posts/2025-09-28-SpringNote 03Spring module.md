---
title: 03 Spring module
date: 2025-09-28 10:00:00 +09:00
categories: [Spring Framework, SpringNote]
tags: [ Java, Spring Framework, Spring Note ]
---

## 1. 스프링 모듈

| 구분                                        | 모듈                                                                                                                                      | 주요 기능                                                                               |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Core Container (핵심 컨테이너)              | - spring-core<br>- spring-beans<br>- spring-context<br>- spring-expression (SpEL)                                       | IoC/DI 컨테이너, 빈 생성/관리, ApplicationContext, SpEL(표현식 언어)                              |
| AOP & Instrumentation                 | - spring-aop<br>- spring-aspects<br>- spring-instrument                                                                     | 관점 지향 프로그래밍(AOP), 트랜잭션 처리, 로깅, 보안 적용                                                |
| Data Access / Integration (데이터 접근 계층) | - spring-jdbc<br>- spring-tx (트랜잭션)<br>- spring-orm (JPA/Hibernate)<br>- spring-oxm (XML 바인딩)<br>- spring-jms (메시징) | JDBC 템플릿, 트랜잭션 관리, ORM 연동, XML 데이터 처리, 메시징 지원                                       |
| Web                                   | - spring-web<br>- spring-webmvc<br>- spring-webflux                                                                         | Servlet API 통합, Multipart 처리, REST 지원, MVC 아키텍처, DispatcherServlet, 리액티브 웹(WebFlux) |
| Messaging                             | - spring-messaging                                                                                                                  | STOMP, WebSocket, 메시징 프로그래밍 모델                                                      |
| Test                                  | - spring-test                                                                                                                       | 단위/통합 테스트 지원, JUnit/Jupiter 연동, Mock 객체 제공                                          |

## 2. 스프링의 변화
### 1.스프링 프레임워크 단계
- 모듈화 설계
  - Core, MVC, DB, AOP, Messaging, Test 등 기능별로 모듈을 분리
  - 장점: 필요 없는 기능을 포함하지 않아도 됨, 영향도 최소화, 유지보수 용이
  - 단점: 개발자가 필요한 모듈을 일일이 추가하고 설정해야 해서 초기 설정 부담이 큼
- 목적
  - 관심사(웹, DB, AOP 등)를 분리해서 모듈 단위로 독립적 개발 가능
  - 모듈 간 의존성을 최소화 → 시스템 복잡도 관리

### 2. 스프링 부트 단계
- Starter와 자동 설정(Auto-Configuration) 제공
  - 스타터: 관련 모듈을 묶음으로 제공 → Core + MVC, Core + DB 등
  - 자동 설정: 최소한의 설정만으로도 모듈이 바로 작동
- 장점: 개발 초기 설정 부담을 대폭 줄임, 필요한 기능만 선택적 사용 가능
- 단점: 내부 모듈을 완전히 세세하게 제어하고 싶으면 별도 설정 필요

## 3. 스프링 모듈 사용

| 활용 형태             | 뷰 필요 여부 | 사용 모듈/기술                                     | 주요 기능/설명                          |
| ----------------- | ------- | -------------------------------------------- | --------------------------------- |
| 웹 애플리케이션          | 필요      | Spring MVC, Thymeleaf, JSP                   | 요청 → Controller → View → Response |
| REST API 서버       | 불필요     | Spring MVC, WebFlux                          | JSON/XML 등 데이터 반환, 모바일/SPA 통신     |
| 마이크로서비스 / 백엔드 서비스 | 불필요     | Spring Boot, Spring Cloud                    | 서비스 간 REST/gRPC/API 호출, 메시징 처리    |
| 배치 작업 / 스케줄러      | 불필요     | Spring Batch, @Scheduled                     | 대용량 데이터 처리, 주기적 반복 작업             |
| 메시징 / 이벤트 처리      | 불필요     | Spring Messaging, RabbitMQ, Kafka, WebSocket | 실시간 이벤트 처리, 메시지 브로커 연동            |
| 순수 애플리케이션 로직      | 불필요     | Spring Core (DI/IoC)                         | 객체 관리, 의존성 주입, 비즈니스 로직 수행         |

## 4. 스프링부트 의존성 / 모듈

| 선택 상태                                 | 포함 모듈/스타터                                                    | 주요 기능/설명                                                                                                             |
| ------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| 아무것도 선택 안함                        | spring-core, spring-context, spring-beans, spring-expression | IoC/DI, Bean 관리, 컨텍스트 관리, SpEL 등 코어 기능만 가능. MVC, AOP, DB, Validation 등 없음                                            |
| 웹(starter-web) 선택                 | spring-web, spring-webmvc, Jackson, 톰캣 내장                    | DispatcherServlet, HandlerMapping, HandlerAdapter, 메시지 컨버터, REST/JSON 처리, MVC 구조 제공. JSP는 별도 설정 필요, Thymeleaf는 별도 선택 |
| DB(starter-data-jpa) 선택           | spring-data-jpa, spring-orm, hibernate, spring-jdbc          | JPA Repository, Entity 매핑, JDBC Template, ORM 지원. DataSource 자동 설정 포함                                                |
| AOP(starter-aop) 선택               | spring-aop, aspectj                                          | @Aspect, @EnableAspectJAutoProxy 지원. 트랜잭션, 로깅, 모니터링 등 가능                                                             |
| Validation(starter-validation) 선택 | javax.validation / jakarta.validation                        | @Valid, @Validated, @NotNull, @Size 등 JSR-303/380 표준 검증 지원                                                           |
