---
title: 10 Spring Boot Actuator
date: 2025-10-11 10:00:00 +09:00
categories: [00Spring Framework, Spring]
tags: [ Java, Spring Framework, SpringBoot, Actuator ]
---

## 1. Spring Boot Actuator
### 1. Spring Boot Actuator
 - 스프링부트 애플리케이션의 운영 상태를 모니터링하고 관리할 수 있도록 도와주는 모듈
 - 서비스가 실제로 운영 중에 어떻게 동작하고 있는지 — 헬스체크, 로그, 메트릭, 환경 정보, 스레드 상태 등 — 를 쉽게 확인할 수 있게 해주는 도구

### 2. Spring

- 스프링 프레임워크와 부트
  - 순수 스프링(Spring Framework): 웹 MVC, DI, AOP 등 “애플리케이션 개발”에 초점
  - 스프링 부트(Spring Boot): 거기에 운영, 설정 자동화, 모니터링, 내장 서버, 실행 관리 기능을 더함

- 스프링 프레임워크에서 적용 

| 구분             | 스프링 프레임워크     | 스프링 부트                                |
| -------------- | ------------- | ------------------------------------- |
| 액추에이터 기본 포함 여부 | ❌ 없음          | ✅ 기본 제공                               |
| 기능 추가 방법       | 직접 구현         | `spring-boot-starter-actuator` 의존성 추가 |
| 자동 설정          | 없음 (수동 구성 필요) | 자동 구성 (즉시 사용 가능)                      |
| 운영/모니터링 편의성    | 낮음            | 매우 높음                                 |


### 3. Actuator로 알 수 있는 것 

| 구분                | 설명                                     | 예시 엔드포인트                                                     |
| ----------------- | -------------------------------------- | ------------------------------------------------------------ |
| JVM 레벨 정보    | JVM 내부 동작 상태, 메모리, 스레드, GC, 클래스 로딩 등   | `/actuator/metrics/jvm.memory.used`, `/actuator/threaddump`  |
| 스프링 컨테이너 정보 | 등록된 Bean, DI 관계, 요청 매핑, 환경 변수          | `/actuator/beans`, `/actuator/mappings`, `/actuator/env`     |
| 애플리케이션 상태 정보 | 헬스체크, 버전, 빌드 정보, 로그 레벨 등               | `/actuator/health`, `/actuator/info`, `/actuator/loggers`    |
| 외부 연동 정보   | DB, MQ, Redis, Mail 서버 같은 외부 리소스 연결 상태 | `/actuator/health` 내부 세부 정보로 표시됨 (`db: UP`, `redis: DOWN` 등) |
| 사용자 정의 메트릭 | 개발자가 추가한 비즈니스 메트릭 (예: 주문 건수, 트래픽량)     | `/actuator/metrics/custom.order.count` (커스텀 등록 가능)           |

## 2. 활용
### 1. 실무에서 활용
- 서버 헬스체크: 로드밸런서가 /actuator/health를 주기적으로 호출해 정상 동작 여부 확인
- 성능 모니터링: Prometheus, Grafana, Datadog 같은 외부 모니터링 도구와 연동 가능
- 장애 대응: 실시간으로 로그 레벨을 바꿔 문제 상황을 즉시 디버깅 가능
- 운영 자동화: 배포 스크립트나 CI/CD 파이프라인에서 상태 확인 용도로 사용

### 2. 의존성
  ```
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```

### 3. application.properties
  ```
    # 1) 액추에이터 엔드포인트 노출 설정
    management.endpoints.web.exposure.include=health,info,metrics,env
    
    # 2) 액추에이터 엔드포인트 기본 경로 설정 (기본값은 /actuator)
    management.endpoints.web.base-path=/actuator
    
    # 3) 헬스 체크 세부 정보 표시
    management.endpoint.health.show-details=always
    
    # 4) info 엔드포인트에 표시할 정보 (빌드, 버전 등)
    info.app.name=MyApplication
    info.app.version=1.0.0
    info.author=유현준
    
    # 5) 서버 포트 (옵션)
    server.port=8080
  ```

### 4. 확인

| 구분        | 엔드포인트(URL)             | 설정 키 (application.properties)                          | 확인 내용                                            | 비고                       |
| --------- | ---------------------- | ------------------------------------------------------ | ------------------------------------------------ | ------------------------ |
| 헬스 체크     | `/actuator/health`     | `management.endpoint.health.show-details=always`       | 애플리케이션의 현재 상태 (`UP`, `DOWN`, `OUT_OF_SERVICE` 등) | 로드밸런서 헬스체크에 주로 사용        |
| 정보        | `/actuator/info`       | `info.app.name`, `info.app.version` 등                  | 애플리케이션 버전, 이름, 작성자 등                             | 사용자 정의 정보 표시 가능          |
| 메트릭       | `/actuator/metrics`    | `management.endpoints.web.exposure.include=metrics`    | JVM 메모리, GC, 요청 수, 응답 시간 등 성능 지표                 | Prometheus 등 모니터링 도구와 연동 |
| 환경 정보     | `/actuator/env`        | `management.endpoints.web.exposure.include=env`        | 현재 애플리케이션 환경 변수, 프로퍼티, 프로파일 등                    | 민감 정보 포함 가능 — 운영에서는 주의   |
| 빈 목록      | `/actuator/beans`      | `management.endpoints.web.exposure.include=beans`      | 스프링 컨테이너에 등록된 Bean 목록과 의존관계                      | 내부 구조 분석 시 유용            |
| 요청 매핑     | `/actuator/mappings`   | `management.endpoints.web.exposure.include=mappings`   | 컨트롤러 요청 URL과 핸들러 매핑 정보                           | API 라우팅 점검 시 활용          |
| 로그        | `/actuator/loggers`    | `management.endpoints.web.exposure.include=loggers`    | 현재 로그 레벨 조회 및 실시간 변경 가능                          | 운영 중 디버깅에 유용             |
| 스레드 덤프    | `/actuator/threaddump` | `management.endpoints.web.exposure.include=threaddump` | 현재 실행 중인 스레드 상태 및 스택트레이스                         | 성능 문제 추적용                |
| HTTP 트레이스 | `/actuator/httptrace`  | `management.endpoints.web.exposure.include=httptrace`  | 최근 HTTP 요청/응답 기록 (기본 100개)                       | Spring Boot 2.x에서만 기본 제공 |
| 캐시        | `/actuator/caches`     | `management.endpoints.web.exposure.include=caches`     | 캐시 상태 및 캐시 이름 목록                                 | Spring Cache 사용 시만 노출    |

## 3. note
### 1. note
 - 관리용 툴로 필요한것 체크해서 사용.
 - 
