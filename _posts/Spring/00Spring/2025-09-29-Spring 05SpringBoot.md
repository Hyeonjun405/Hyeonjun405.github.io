---
title: 05 SpringBoot
date: 2025-09-29 10:00:00 +09:00
categories: [00Spring Framework, Spring]
tags: [ Java, Spring Framework, SpringBoot ]
---

## 1. 스프링과 스프링부트
### 1. 스프링부트
  - 스프링부트(Spring Boot)는 스프링(Spring Framework)을 더 쉽게 사용할 수 있도록 만들어진 상위 도구이자 확장 프레임워크
  - 스프링부트 없이 스프링만 사용할 수 있지만, 스프링부트는 반드시 스프링 위에서 동작하는 구조

### 2. 스프링의 한계
 - 설정의 복잡성
    - 초기에는 XML 기반 설정을 통해 빈(bean) 등록, 의존성 주입, 트랜잭션 관리 등을 수행해야 했음
    - 애노테이션 기반 설정이 도입되었으나 여전히 세부 설정 요소가 많음
  - 개발 환경 세팅의 어려움
    - 프로젝트 시작 시 필요한 라이브러리를 개별적으로 추가해야 함
    - 라이브러리 간 버전 충돌 문제가 빈번하게 발생함
  - 서버 실행의 번거로움
    - 애플리케이션 실행을 위해 외부 WAS(Tomcat, Jetty 등)에 WAR 파일을 배포해야 함
    - 코드 수정 시 재배포 과정이 반복적으로 요구됨
  - 운영 환경 지원 부족
    - 모니터링, 헬스 체크, 로그 관리 등 운영 환경에 필요한 기능을 기본적으로 제공하지 않음
    - 개발자가 직접 외부 라이브러리를 도입하고 설정해야 함
  - 학습 곡선의 높음
    - IoC/DI, AOP, 트랜잭션 관리, MVC 패턴 등 핵심 개념과 설정 요소를 모두 학습해야 함
    - 초급 개발자가 접근하기에 진입 장벽이 높음
  
### 3. 부트에서의 해결방식

| 구분     | 기존 스프링의 단점                         | 스프링부트의 해결                                      |
| ------ | ---------------------------------- | ---------------------------------------------- |
| 설정     | XML 및 복잡한 설정 필요                    | Auto Configuration을 통한 자동 설정 제공                |
| 의존성 관리 | 라이브러리를 개별적으로 추가하고 버전 충돌 위험 존재      | Starter 패키지를 통한 일괄 의존성 관리 및 호환성 보장             |
| 서버 실행  | 외부 WAS에 WAR 배포 필요                  | 내장 서버(Tomcat, Jetty 등) 포함으로 JAR 실행만으로 서버 구동 가능 |
| 운영 환경  | 모니터링, 헬스 체크, 로그 관리 기능 미제공          | Spring Boot Actuator 등 운영 환경 지원 기능 기본 제공       |
| 학습 곡선  | IoC, AOP, 트랜잭션 관리 등 개념과 설정 학습 부담 큼 | 기본 설정 단순화 및 빠른 프로젝트 생성으로 진입 장벽 완화              |

### 4. 스프링부트와 스프링
 - `스프링부트만의 기능이 특별히 새로 추가된 건 없음(☆)`
 - 스프링부트의 핵심 가치는 "스프링 기반 개발의 복잡성을 줄이고, 설정·배포를 단순화하는 것"
 - 스프링에서 가능하지만 구현이 어려운 것을 부트가 쉽게 만들어주는 것

## 2. 스프링부트의 역할
### 1. 의존성관리
#### 1. 의존성관리
 - 스프링부트는 **BOM(Bill of Materials)**이라는 의존성 버전 관리 시스템을 제공
 - 개발자가 개별 라이브러리 버전을 일일이 맞추지 않아도 됨
 - Starter라는 묶음 의존성을 제공 → 예: spring-boot-starter-web 하나로 웹 개발에 필요한 여러 라이브러리를 포함
 - 결과적으로 의존성 설정이 훨씬 간단해지고 버전 충돌 위험이 줄어듦

#### 2. BOM
 - 스프링부트에서 의존성 버전 관리 규칙이 담긴 특별한 POM 파일
 - 스프링부트는 spring-boot-dependencies라는 BOM을 제공
 - BOM에는 각 라이브러리의 안정된 버전 세트가 정의되어 있음
 - 개발자는 버전을 직접 명시하지 않아도, BOM이 자동으로 호환되는 버전을 지정
 - 스프링부트 프로젝트를 만들 때 보통 ***spring-boot-starter-parent***나 ***spring-boot-dependencies***가 BOM 역할을 함

#### 3. 스프링부트의 의존성

| 의존성 (Starter)                  | 포함 기능 / 용도                                         |
| ------------------------------ | -------------------------------------------------- |
| spring-boot-starter            | 기본 스타터, 로깅(Logback) 등 핵심 기능 포함                     |
| spring-boot-starter-web        | Spring MVC, REST API 개발, 내장 톰캣 제공                  |
| spring-boot-starter-data-jpa   | JPA, Hibernate 기반 ORM 지원                           |
| spring-boot-starter-security   | 인증·인가 처리, 보안 기능 제공                                 |
| spring-boot-starter-thymeleaf  | 서버 사이드 템플릿 엔진 Thymeleaf 지원                         |
| spring-boot-starter-validation | 요청 데이터 검증(Bean Validation, Hibernate Validator) 지원 |
| spring-boot-starter-test       | JUnit, Mockito 등 테스트 라이브러리 제공                      |
| spring-boot-starter-actuator   | 모니터링, 헬스 체크, 메트릭 등 운영 관리 기능 제공                     |

### 2. 자동설정
#### 1. 자동설정
 - 스프링부트는 프로젝트의 클래스패스(classpath)와 선언된 의존성을 분석하여 필요한 설정을 자동으로 적용
 - 예: 데이터베이스 의존성이 있으면 자동으로 DataSource, JPA 설정을 구성
 - 개발자가 수동으로 설정할 필요를 최소화 → 빠른 개발 가능

#### 2. 자동설정 방식
  ```
  @SpringBootApplication
  public class MyApplication {
      public static void main(String[] args) {
          SpringApplication.run(MyApplication.class, args);
      }
  }
  ```
  - @SpringBootApplication는 @EnableAutoConfiguration, @ComponentScan, @Configuration 을 기본 포함함.
  - @SpringBootApplication이 활성화되면
    - 프로젝트에 spring-boot-starter-web이 있다 → 부트는 DispatcherServlet, 톰캣, JSON 변환 설정 등을 자동 구성
    - 프로젝트에 spring-boot-starter-data-jpa가 있다 → 부트는 DataSource, EntityManager, Hibernate 설정 등을 자동 구성

### 3. 내장AWS(Web Application Server)
#### 1. 내장 WAS
 - 스프링부트는 톰캣(Tomcat), 제티(Jetty), 언더토우(Undertow) 같은 웹 서버를 내장
 - 개발 및 배포 과정이 단순해짐
 - 외부 서버 설치 없이 JAR 파일 실행만으로 애플리케이션 실행 가능
  ```
   java -jar mySpringBootApp.jar
  ```

#### 2. 스프링과 스프링부트

| 구분       | 기존 스프링                     | 스프링부트             |
| -------- | -------------------------- |-------------------|
| 서버 필요 여부 | 외부 WAS 필요(Tomcat, JBoss 등) | 내장 WAS 포함(JAR 실행) |
| 배포 형태    | WAR 파일 배포                  | JAR 파일 실행         |
| 실행 과정    | WAS 설치 → WAR 배포 → 서버 실행    | ***단일 명령어로 실행***  |
| 환경 통일성   | 환경마다 WAS 설정 차이 존재          | 동일한 WAS 환경 제공     |


### 4. 모니터링
#### 1. 모니터링
 - 운영 환경에서 애플리케이션 상태를 확인할 수 있는 기능 제공
 - Spring Boot Actuator 라이브러리를 통해 제공
 - 헬스 체크, 메트릭, 트래픽 상태, 로그 수준 변경, 환경 변수 조회 등 가능
 - 운영 및 유지보수 편의성 증가

#### 2. 스프링과 스프링부트

| 기능     | 기존 스프링    | 스프링부트                                |
| ------ | --------- | ------------------------------------ |
| 헬스 체크  | 직접 구현해야 함 | Actuator 기본 제공 (`/actuator/health`)  |
| 메트릭 수집 | 직접 구현 필요  | Actuator 기본 제공 (`/actuator/metrics`) |
| 로그 관리  | 직접 구현 필요  | Actuator로 동적 로그 레벨 제어 가능             |
| 운영 통합  | 별도 설정 필요  | Actuator로 간단 통합 가능                   |


#### 3. Spring Boot Actuator의 주요기능

| 기능                      | 설명                                                           |
| ----------------------- | ------------------------------------------------------------ |
| **헬스 체크(Health Check)** | 애플리케이션의 정상 동작 여부 확인 (`/actuator/health`)                     |
| **메트릭(Metrics)**        | CPU 사용률, 메모리 사용량, 쓰레드 상태, HTTP 요청 통계 등 (`/actuator/metrics`) |
| **환경 정보(Environment)**  | 애플리케이션 환경 변수, 프로퍼티 정보 (`/actuator/env`)                      |
| **로그 레벨 변경**            | 실행 중 로그 레벨 변경 가능 (`/actuator/loggers`)                       |
| **애플리케이션 정보(App Info)** | 버전, 빌드 정보 등 (`/actuator/info`)                               |
| **트레이스(Trace)**         | 최근 HTTP 요청 정보(`trace`)                                       |
