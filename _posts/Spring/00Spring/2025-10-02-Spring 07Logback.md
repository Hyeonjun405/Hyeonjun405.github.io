---
title: 07 Logback
date: 2025-10-02 10:00:00 +09:00
categories: [00Spring Framework, Spring]
tags: [ Java, Spring Framework, SpringBoot, Swagger ]
---

## 1. SL4J
### 1. SL4J
- **자바 로깅을 위한 ‘표준 인터페이스(API)’**
- 자바에는 이미 다양한 로깅 프레임워크가 있고, 각각은 API가 다르기 때문에 프로젝트에서 혼합해서 쓰면 충돌이나 복잡함이 생김.
- 이를 해결하기 위해 공통된 로깅 API를 만들어서 사용하는데 이것이 SL4J
- 개발자는 SLF4J API에만 의존하면 됨

### 2. 구현체

| 구현체          | 특징                                            |
| ------------ | --------------------------------------------- |
| Logback      | SLF4J의 공식 구현체, 스프링 부트 기본 로깅 엔진, 고성능·자동 재로딩 지원 |
| SLF4J-Simple | 가벼운 콘솔 로깅 구현체, 설정 간단, 주로 테스트나 소규모 프로젝트에 사용    |
| Tinylog      | 경량 로깅, SLF4J API 직접 구현, 빠른 성능, 설정 간단          |
| SimpleLogger | SLF4J 제공 기본 구현체, 설정 파일 없이 간단 사용 가능            |

### 3. 연동 대상 (Bridge 필요)

| 로깅 프레임워크                      | 특징                            | SLF4J 연동 방식             |
| ----------------------------- | ----------------------------- | ----------------------- |
| Log4j (구버전)                   | 과거 널리 쓰였지만 유지보수 중단, 보안 취약점 존재 | bridge(binder) 라이브러리 사용 |
| Log4j2                        | Log4j의 후속 버전, 고성능, 다양한 기능 제공  | log4j-slf4j-impl 모듈 사용  |
| java.util.logging (JUL)       | JDK 기본 내장 로깅, 기본 기능 제공        | jul-to-slf4j 브리지 사용     |
| JCL (Jakarta Commons Logging) | 과거 자바 라이브러리에서 많이 사용됨          | jcl-over-slf4j 모듈 사용    |

## 2. Logback
### 1. Logback
- Logback은 자바용 고성능 로깅 프레임워크로, SLF4J의 공식 구현체
- 스프링 부트에서는 기본 로깅 엔진으로 Logback을 채택
- 성능과 유연성이 뛰어나고, 설정이 단순하면서도 강력

### 2. 특징

| 특징              | 설명                                                                 |
| --------------- | ------------------------------------------------------------------ |
| 성능 최적화          | Log4j보다 빠르고 메모리 효율적                                                |
| 자동 재로딩          | 설정 파일(logback.xml, logback-spring.xml) 변경 시 애플리케이션 재시작 없이 반영       |
| 다양한 Appender 지원 | ConsoleAppender, FileAppender, RollingFileAppender, SMTPAppender 등 |
| 유연한 로그 필터링      | 로그 레벨, MDC, 조건부 필터 등을 세밀하게 제어 가능                                   |
| 패턴 기반 출력        | 날짜, 로거 이름, 로그 레벨 등 자유로운 포맷 지정                                      |
| 스프링 부트 통합       | 별도 설정 없이 기본 로깅 엔진으로 사용 가능                                          |

### 3. 스프링과 로그백

| 이유        | Logback                              | Log4j2                    |
| --------- | ------------------------------------ | ------------------------- |
| 기본 제공 여부  | 스프링 부트 기본 로깅 구현체로 별도 설정 없이 사용 가능     | 별도 의존성 추가 필요              |
| 성능        | 빠르고 메모리 효율적                          | 매우 빠르지만 초기 안정성 검증 시점이 필요  |
| 설정 편의성    | logback-spring.xml로 간단 설정, 자동 재로딩 지원 | 설정 복잡, 자동 재로딩 기본 미지원      |
| 유연성       | 다양한 Appender, 필터, MDC 지원             | 다양한 기능 제공하지만 설정이 복잡       |
| 안정성       | 오랫동안 검증된 SLF4J 공식 구현체                | 강력하지만 상대적으로 신뢰성 검증 기간이 짧음 |
| SLF4J 호환성 | 공식 구현체로 완벽 호환                        | 바인딩 모듈 필요                 |
| 스프링 통합성   | 스프링 부트에 기본 통합, 별도 설정 불필요             | 별도 설정 및 의존성 필요            |


### 4. 로그백의 구성(Logback 라이브러리 안에서 제공되는 클래스와 기능)
#### 1. 전체 구성

| 구성 요소            | 역할           | 설명                                                                |
| ---------------- | ------------ | ----------------------------------------------------------------- |
| Logger           | 로그 생성 주체     | 클래스 단위로 생성, 로그 레벨 지정 가능 (`LoggerFactory.getLogger()`)             |
| Appender         | 로그 출력 대상     | 콘솔, 파일, 이메일, DB 등 다양한 출력 방식 제공                                    |
| Encoder / Layout | 로그 메시지 형식 지정 | 로그 출력 패턴, 날짜, 로그 레벨, 로거 이름 등을 지정 가능                               |
| Layout           | 로그 출력 포맷 템플릿 | `%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n` 형태로 포맷 정의 |

 - 동작흐름
  ```
  Logger → Appender → Encoder/Layout → 최종 출력
  ```

#### 2. Appender
- 구조<br>
  ![내 그림](assets/img/Spring/appenderClassDiagram.jpg "이미지")
  
  | 클래스/인터페이스                      | 역할                                       | 주요 메소드                                                                                               |
  | ------------------------------ | ---------------------------------------- | ---------------------------------------------------------------------------------------------------- |
  | Appender (interface)       | 로그 출력을 정의하는 최상위 인터페이스                    | `doAppend(E event)` : 로그 이벤트를 처리                                                                     |
  | UnsynchronizedAppenderBase | Appender의 기본 기능 구현, 동기화 없음               | `doAppend(E event)` : 실제 로그 처리, `append(E event)` : 로그 처리 로직                                         |
  | OutputStreamAppender       | 로그를 OutputStream으로 출력하는 Appender의 기본 클래스 | `setOutputStream(OutputStream)` : 출력 스트림 설정, `setEncoder(Encoder<E>)` : Encoder 설정                   |
  | ConsoleAppender            | 콘솔(표준 출력)으로 로그 출력                        | `setTarget(String)` : 출력 대상(Console, System.err 등) 설정                                                |
  | FileAppender               | 파일에 로그를 기록                               | `setFile(String)` : 로그 파일 경로 지정, `setAppend(boolean)` : 기존 파일에 추가 여부                                 |
  | RollingFileAppender        | 로그 파일 기록 + 롤링(회전) 기능                     | `setRollingPolicy(RollingPolicy)` : 파일 롤링 정책, `setTriggeringPolicy(TriggeringPolicy<E>)` : 롤링 트리거 조건 |
  | Filter                     | 로그 필터링 처리                                | `decide(E event)` : 로그를 처리할지 결정하는 메소드                                                                |
  | Encoder (interface)        | 로그 메시지의 포맷을 정의                           | `init(OutputStream)`, `doEncode(E event)`, `close()`                                                 |

- 스프링부트의 경우에는 별도설정을 안할 경우 자동 설정 됨.
- 스프링 프레임워크에서는 사용자가 설정 파일에서 지정한 Appender만 활성화
   - Logback은 logback.xml(또는 logback-test.xml)을 읽어서 어떤 Appender를 사용할지 결정
   - 예: <appender name="FILE" class="ch.qos.logback.core.FileAppender">
   - 설정에 명시된 Appender만 생성·등록

#### 3. Layout
- 기본 로그
  ```
  <pattern>[%d{yyyy-MM-dd HH:mm:ss}] %-5level [%logger{36}] [%thread] - %msg%n</pattern>
  ```
  
- 레벨
  - 레벨종류 

  | 레벨    | 설명                                        | 사용 사례                 |
  | ----- | ----------------------------------------- | --------------------- |
  | TRACE | 가장 상세한 로그. 디버깅 목적, 거의 모든 실행 정보를 기록        | 메소드 진입/종료, 변수 상태 추적 등 |
  | DEBUG | 개발 단계에서 문제 파악용. TRACE보다 덜 상세하지만 충분한 정보 제공 | 로직 흐름 확인, 변수 값 점검     |
  | INFO  | 일반적인 정보 메시지. 운영 환경에서 기본 로그 수준             | 서비스 시작/종료, 주요 이벤트 발생  |
  | WARN  | 경고 메시지. 오류는 아니지만 주의가 필요한 상황               | 성능 저하, 권장하지 않는 사용     |
  | ERROR | 치명적인 오류 메시지. 시스템 동작에 영향이 있는 경우            | 예외 발생, 실패 처리          |

 - 우선순위 
  ```
  TRACE < DEBUG < INFO < WARN < ERROR < OFF
  ```
  - 로그 레벨을 설정해두면 그 하위 레벨은 나오지 않음.
  - OFF/ ALL은 설정 전용



- 패턴
 
  | 패턴                   | 설명                                      | 예시 출력                                             |
  | -------------------- | --------------------------------------- | ------------------------------------------------- |
  | `%d{패턴}`             | 로그 발생 시간                                | `%d{yyyy-MM-dd HH:mm:ss}` → `2025-10-02 14:30:15` |
  | `%level` / `%p`      | 로그 레벨                                   | `INFO`, `DEBUG`, `ERROR`                          |
  | `%logger`            | 로거 이름(클래스명 포함)                          | `com.example.service.UserService`                 |
  | `%logger{length}`    | 로거 이름 축약                                | `%logger{36}` → 최대 36자 출력                         |
  | `%msg` / `%m`        | 로그 메시지                                  | `"사용자 생성 완료"`                                     |
  | `%thread`            | 로그를 발생시킨 쓰레드 이름                         | `"main"`                                          |
  | `%X{key}`            | MDC(Mapped Diagnostic Context)의 특정 값 출력 | `%X{userId}` → `"12345"`                          |
  | `%n`                 | 줄바꿈                                     | OS별 줄바꿈 적용                                        |
  | `%file`              | 로그 발생 소스 파일 이름                          | `"UserService.java"`                              |
  | `%line`              | 로그 발생 코드 라인 번호                          | `"42"`                                            |
  | `%method`            | 로그 발생 메소드 이름                            | `"createUser"`                                    |
  | `%class`             | 로그 발생 클래스 이름                            | `"com.example.service.UserService"`               |
  | `%mdc`               | MDC 전체 출력                               | `{userId=12345, traceId=abc123}`                  |
  | `%ex` / `%throwable` | 예외 스택 트레이스 출력                           | 전체 예외 로그                                          |
  | `%relative`          | 애플리케이션 시작 이후 경과 시간(ms)                  | `"1523"`                                          |
  | `%property{key}`     | 시스템 속성 값                                | `%property{user.home}` → `"/Users/username"`      |

## 3.Logback 적용
### 1. 의조선
#### 1. 스프링부트
 - 별도 설치 필요 없음 → spring-boot-starter 안에 이미 logback-classic 포함
 - 기본적으로 로그는 INFO 레벨로 콘솔 출력됨

#### 2. 스프링프레임워크
  ```
  <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.4.11</version> <!-- 최신 버전 확인 -->
  </dependency>
  ```

### 2. 로거 생성
  ```
  private static final Logger logger = LoggerFactory.getLogger(Example.class);
  
      public static void main(String[] args) {
          logger.trace("TRACE 로그");
          logger.debug("DEBUG 로그");
          logger.info("INFO 로그");
          logger.warn("WARN 로그");
          logger.error("ERROR 로그");
      }
  }
  ```
 - 클래스 단위로 로그의 수준이나 상황을 다르게 표기하기 위해서 주입을 사용하지 않고 개별 객체로 사용 권장.
 - 로그에서 어떤 클래스인지 나와서, 타겟 클래스를 찾기 쉬워짐. 

### 3. 로그 설정
#### 1. 스프링부트
  ```
  logging.level.root=INFO
  logging.level.com.example=DEBUG
  logging.file.name=logs/app.log
  ```

#### 2. Logxml
  ```
  <configuration>
    <!-- 콘솔 출력 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss}] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 파일 출력 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss}] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 루트 로거 -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
  </configuration>
  ```

