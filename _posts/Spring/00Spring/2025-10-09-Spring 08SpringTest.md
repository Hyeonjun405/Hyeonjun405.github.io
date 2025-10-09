---
title: 08 SpringTest
date: 2025-10-09 10:00:00 +09:00
categories: [00Spring Framework, Spring]
tags: [ Java, Spring Framework, SpringBoot, SpringTest ]
---

## 1. SpringTest
### 1. Junit - SpringTest - SpringBootTest
  
  | 구분                           | 역할                                                                   | 관계                          |
  | ---------------------------- | -------------------------------------------------------------------- |-----------------------------|
  | JUnit                    | 자바에서 단위 테스트를 작성하고 실행하기 위한 테스트 프레임워크                              | 스프링과 무관하게 단독으로 사용 가능        |
  | Spring Test (스프링 테스트 모듈) | 스프링 컨테이너, 빈, 트랜잭션 등을 테스트 환경에서 제어할 수 있게 도와주는 확장 도구                | 내부적으로 **JUnit 위에서 동작**      |
  | Spring Boot Test         | `@SpringBootTest`, `@MockBean`, `@DataJpaTest` 등 통합 테스트 지원 기능 제공 | Spring Test + Boot 환경 통합 버전 |

### 2. JUnit 구성요소

  | 구성 요소              | 설명                          | 주요 역할                                                      | 특징                                    |
  | ------------------ | --------------------------- | ---------------------------------------------------------- | ------------------------------------- |
  | JUnit Platform | 테스트 실행의 기반 엔진               | 테스트를 실행하고 관리하는 환경 제공 (IDE, Gradle, Maven 등과 연동)        | 테스트 엔진을 탐색하고 실행 결과를 리포팅함              |
  | JUnit Jupiter  | JUnit 5의 핵심 API 모듈          | 우리가 사용하는 `@Test`, `@BeforeEach`, Assertions 등 실제 기능 제공 | JUnit 5 스타일 테스트 작성 시 기본적으로 사용하는 부분    |
  | JUnit Vintage  | 이전 버전(JUnit 3, 4) 테스트 호환 모듈 | 기존 프로젝트의 레거시 테스트 코드 실행 지원                              | JUnit 5 환경에서도 JUnit 4 테스트를 함께 돌릴 수 있음 |

## 2. JUnit Jupiter가 제공하는 애노테이션

### 1. 테스트 생명주기 관련 애노테이션

| 어노테이션         | 설명                                  |
| ------------- | ----------------------------------- |
| `@BeforeAll`  | 테스트 클래스 시작 전에 한 번 실행 (ex. DB 연결 설정) |
| `@BeforeEach` | 각 테스트 실행 전마다 실행 (ex. 테스트 초기화)       |
| `@Test`       | 실제 테스트 코드                           |
| `@AfterEach`  | 각 테스트 실행 후마다 실행 (ex. 리소스 정리)        |
| `@AfterAll`   | 모든 테스트 종료 후 한 번만 실행                 |

### 2. 단언(Assertion) API

| 메서드                                                | 설명               |
| -------------------------------------------------- | ---------------- |
| `assertEquals(expected, actual)`                   | 예상값과 실제값이 같은지    |
| `assertTrue(condition)` / `assertFalse(condition)` | 조건이 참/거짓인지       |
| `assertThrows(Exception.class, () -> { ... })`     | 특정 예외 발생 여부 확인   |
| `assertAll()`                                      | 여러 검증을 한꺼번에 실행   |
| `assertNotNull(obj)`                               | 객체가 null이 아닌지 확인 |

### 3. 테스트 조건 제어

| 어노테이션                             | 설명                    |
| --------------------------------- | --------------------- |
| `@Disabled`                       | 특정 테스트 비활성화           |
| `@EnabledOnOs(OS.WINDOWS)`        | 특정 운영체제에서만 실행         |
| `@EnabledIfEnvironmentVariable()` | 환경 변수 조건에 따라 실행 여부 결정 |

### 4. 테스트 반복 파라미터

| 어노테이션                                     | 설명                  |
| ----------------------------------------- | ------------------- |
| `@RepeatedTest(5)`                        | 동일한 테스트를 여러 번 반복    |
| `@ParameterizedTest`                      | 여러 입력값으로 같은 테스트를 반복 |
| `@ValueSource(strings = {"a", "b", "c"})` | 파라미터화 테스트용 값 지정     |

### 5. 태깅 및 그룹화

| 어노테이션                                 | 설명                   |
| ------------------------------------- | -------------------- |
| `@Tag("fast")`, `@Tag("integration")` | 테스트를 분류할 때 사용        |
| `@DisplayName("사용자 등록 기능 테스트")`       | 테스트 설명을 사람이 읽기 좋게 표시 |


## 3. 스프링 프레임워크와 스프링부트 테스트
### 1. 스프링 프레임워크, 스프링부트

  | 구분            | Spring Framework Test                                         | Spring Boot Test                                                      |
  | ------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------- |
  | 제공 주체     | Spring Framework                                                  | Spring Boot                                                               |
  | 주요 패키지    | `org.springframework.test.`                                      | `org.springframework.boot.test.`                                         |
  | 테스트 목적    | 스프링 컨테이너, 빈 주입, 트랜잭션 등 핵심 기능 검증                               | 스프링부트 애플리케이션 전체 또는 특정 계층 테스트 간소화                                      |
  | 환경        | 순수 스프링 환경 (부트 의존 X)                                               | 스프링부트 프로젝트 환경 (Auto Configuration 활용)                                     |
  | 대표 어노테이션  | `@ContextConfiguration`, `@WebAppConfiguration`, `@Transactional` | `@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest`, `@MockBean`, `@SpyBean` |
  | 주요 도구/클래스 | `MockMvc`, `TestContext`, `ApplicationContext`                    | `SpringBootContextLoader`, `SpringBootTestContextBootstrapper`            |
  | 설정 방식     | 설정 파일(XML 또는 Java Config)을 직접 지정                                  | 스프링부트 설정을 자동으로 로드 (간단함)                                                   |
  | 테스트 범위    | 특정 Bean, 트랜잭션, 컨텍스트 레벨                                            | 애플리케이션 전체 또는 특정 계층 단위 (MVC, JPA 등)                                        |
  | 특징        | • 테스트 인프라 제공 (핵심 기능 중심)<br>• 구성 복잡하지만 유연함                         | • 자동 설정 기반, 작성 간편<br>• 실제 서비스 환경에 가까운 테스트 가능                              |

### 2. Spring FrameWork Test
  
  | 주요 어노테이션                                | 설명                                      |
  | --------------------------------------- | --------------------------------------- |
  | `@ContextConfiguration`                 | 테스트에서 사용할 스프링 설정 파일(XML/Java Config) 지정 |
  | `@WebAppConfiguration`                  | 웹 애플리케이션 환경 테스트 설정                      |
  | `@Transactional`                        | 테스트 실행 후 트랜잭션 롤백 처리                     |
  | `@RunWith(SpringRunner.class)` (JUnit4) | 스프링 테스트 컨텍스트와 통합 실행                     |
  | `@TestPropertySource`                   | 테스트 환경에서 사용할 프로퍼티 지정                    |


### 3. 스프링부트 애노테이션

  | 분류                        | 목적                      | 주요 어노테이션                                                                                                                                  |
  | ------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
  | 전체 애플리케이션 통합 테스트      | 애플리케이션 전체를 띄워 통합적으로 테스트 | `@SpringBootTest`                                                                                                                         |
  | 계층별 테스트 (Test Slices) | 특정 계층만 독립적으로 테스트        | `@WebMvcTest`, `@DataJpaTest`, `@JdbcTest`, `@DataJdbcTest`, `@RestClientTest`, `@JsonTest`, `@WebFluxTest`                               |
  | 테스트 환경 설정 및 구성     | 테스트 실행 환경 조정 및 설정 추가    | `@TestConfiguration`, `@TestPropertySource`, `@DynamicPropertySource`, `@ActiveProfiles`, `@DirtiesContext`, `@AutoConfigureTestDatabase` |
  | Mock / Spy 관련 테스트 지원 | 의존성 모킹 및 스파잉            | `@MockBean`, `@SpyBean`, `@MockitoBean`, `@MockitoSpyBean`                                                                                |


## 4. AssertTJ
### 1. AssertJ
 - Java용 Fluent Assertion 라이브러리
 - 테스트에서 결과를 검증하는 강력하고 직관적인 API 제공
 - 자연어처럼 읽히는 문법(Fluent API) 지원
 - spring-boot-starter-test에 기본 포함돼 있어서 스프링부트에서 바로 사용 가능

## 2. 검증 메소드
### 1. 분류

  | 분류                   | 검증 메소드                        | 설명                         |
  | -------------------- | ----------------------------- | -------------------------- |
  | 기본 비교            | isEqualTo(expected)           | 실제 값이 기대 값과 같은지 검증         |
  |                      | isNotEqualTo(expected)        | 실제 값이 기대 값과 다름을 검증         |
  |                      | isSameAs(expected)            | 실제 값이 동일 객체인지 검증           |
  |                      | isNotSameAs(expected)         | 실제 값이 동일 객체가 아님을 검증        |
  | 널/불리언 검증         | isNull()                      | 값이 null인지 검증               |
  |                      | isNotNull()                   | 값이 null이 아님을 검증            |
  |                      | isTrue()                      | 값이 true인지 검증               |
  |                      | isFalse()                     | 값이 false인지 검증              |
  | 숫자 검증            | isGreaterThan(value)          | 실제 값이 주어진 값보다 큰지 검증        |
  |                      | isLessThan(value)             | 실제 값이 주어진 값보다 작은지 검증       |
  |                      | isBetween(start, end)         | 값이 범위 안에 있는지 검증            |
  |                      | isPositive(), isNegative()    | 값이 양수/음수인지 검증              |
  | 문자열 검증           | startsWith(prefix)            | 문자열이 특정 접두사로 시작하는지 검증      |
  |                      | endsWith(suffix)              | 문자열이 특정 접미사로 끝나는지 검증       |
  |                      | contains(substring)           | 문자열에 특정 내용이 포함되는지 검증       |
  |                      | matches(regex)                | 정규식과 일치하는지 검증              |
  | 컬렉션 검증           | hasSize(size)                 | 컬렉션의 크기가 특정 값인지 검증         |
  |                      | contains(element)             | 컬렉션이 특정 요소를 포함하는지 검증       |
  |                      | containsExactly(elements)     | 요소 순서와 내용까지 일치하는지 검증       |
  |                      | isEmpty(), isNotEmpty()       | 컬렉션이 비어 있는지/비어 있지 않은지 검증   |
  | 예외 검증            | isInstanceOf(Exception.class) | 예외 타입 검증                   |
  |                      | hasMessageContaining(text)    | 예외 메시지에 특정 문자열 포함 여부 검증    |
  | 옵셔널(Optional) 검증 | isPresent()                   | Optional 값이 존재하는지 검증       |
  |                      | isEmpty()                     | Optional 값이 비어있는지 검증       |
  |                      | contains(value)               | Optional 값이 특정 값을 포함하는지 검증 |

### 2. 테스트
#### 1. 체크
  ```
  import static org.assertj.core.api.Assertions.assertThat;
  
  @Test
  void exampleTest() {
      int result = 2 + 2;
  
      assertThat(result).isEqualTo(4);  // true → 테스트 통과
      assertThat(result > 5).isTrue(); // false → AssertionError 발생, 테스트 실패
  }
  ```

#### 2. 체이닝
  ```
    import static org.assertj.core.api.Assertions.assertThat;

    @Test
    void chainingExample() {
        String name = "홍길동";
    
        assertThat(name)
            .isNotNull() // .isNotNull() → null 여부 확인
            .startsWith("홍") // .startsWith("홍") → 문자열 시작 여부 확인
            .contains("길") // .contains("길") → 특정 문자열 포함 여부 확인
            .endsWith("동"); // .endsWith("동") → 문자열 끝 여부 확인
    }
  ```

### 3. 결과값
#### 1. 결과값
 - AssertJ는 기본적으로 실패 시 기대값 vs 실제값 비교 메시지를 자동 생성
 - 메시지를 따로 지정하지 않으면 AssertJ가 제공하는 기본 메시지가 출력됨

#### 2. 일반 실패
 - Case
    ```
    assertThat(2 + 2).isEqualTo(5);
    ```
 - 실패로그
    ```
     Expecting:
     <4>
     to be equal to:
     <5>
     but was not.
   ```

### 3. 커스텀 실패
 - Case
   ```
    assertThat(2 + 2)
     .withFailMessage("계산 결과가 예상과 다릅니다. 실제값: %s", 2 + 2)
     .isEqualTo(5);
   ```
 - 실패로그
   ```
    계산 결과가 예상과 다릅니다. 실제값: 4
    Expecting:
    <4>
    to be equal to:
    <5>
    but was not.
   ```

## 5. 실제 테스트 예시
### 1. AssertJ & Junit
  
  | 구분            | JUnit                                                        | AssertJ                                               |
  | ------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
  | 역할        | 테스트 실행 프레임워크                                                 | Assertion 라이브러리                                       |
  | 주요 기능     | 테스트 메서드 실행, 라이프사이클 관리 (`@Test`, `@BeforeEach`, `@AfterEach`) | 테스트 결과 검증 (`assertThat`, `isEqualTo`, `isNotNull` 등)  |
  | 문법 스타일    | assertEquals(expected, actual)                               | assertThat(actual).isEqualTo(expected)                |
  | 가독성       | 다소 직관적이지 않음                                                  | 자연어에 가까운 가독성                                          |
  | 확장성       | 제한적                                                          | 풍부하고 직관적인 검증 API 제공                                   |
  | 지원 데이터 타입 | 단순 비교                                                        | 문자열, 컬렉션, 객체, 예외 등 다양한 타입 검증                          |
  | 객체 검증     | 제한적                                                          | 필드 단위 검증, 복잡 객체 비교 가능                                 |
  | 실무 트렌드    | 테스트 실행용으로 사용                                                 | 검증 로직은 대부분 AssertJ로 작성                                |
  | 대표 메서드    | assertEquals, assertTrue, assertFalse, assertNull            | assertThat, isEqualTo, isNotNull, contains, hasSize 등 |
  | 의존성       | JUnit 라이브러리                                                  | AssertJ 라이브러리 (`spring-boot-starter-test`에 포함)        |
  
### 2. 클래스 위치
- 테스트 코드 위치: src/test/java/
- 테스트 패키지: 보통 실제 코드 패키지 구조를 그대로 복제
   예: 실제 코드 com.example.demo.service → 테스트 코드 com.example.demo.service
- 이유: 이렇게 하면 테스트가 해당 코드와 논리적으로 연결되고, IDE나 빌드 도구(Gradle/Maven)가 자동으로 테스트를 찾음
- Package
  ```
  src/
   ├── main/
   │    ├── java/
   │    │    └── com/example/demo/   ← 실제 코드
   │    └── resources/
   └── test/
        ├── java/
        │    └── com/example/demo/   ← 테스트 코드
        └── resources/
  ```

### 3. 서비스 로직 테스트
#### 1. 서비스클래스
  ```
  package com.example.demo.service;
  
  import org.springframework.stereotype.Service;
  
  @Service
  public class CalculatorService {
      public int add(int a, int b) {
          return a + b;
      }
  
      public int subtract(int a, int b) {
          return a - b;
      }
  }
  ```

#### 2. 테스트 클래스
  ```
  package com.example.demo;
  
  import com.example.demo.service.CalculatorService;
  import org.junit.jupiter.api.Test;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  
  import static org.assertj.core.api.Assertions.assertThat;
  
  @SpringBootTest
  class CalculatorServiceTest {
  
      @Autowired
      private CalculatorService calculatorService;
  
      @Test
      void testAdditionAndSubtraction() {
          int sum = calculatorService.add(5, 3);
          int difference = calculatorService.subtract(5, 3);
  
          assertThat(sum)
              .isEqualTo(8)
              .isPositive()
              .isGreaterThan(7);
  
          assertThat(difference)
              .isEqualTo(2)
              .isPositive()
              .isLessThan(5);
      }
  }
  ```

#### 3. 결과값
- 성공
  ```
  BUILD SUCCESS
   Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
  ```

- 실패
  ```
  org.opentest4j.AssertionFailedError:
   Expecting:
   <9>
   to be equal to:
   <8>
   but was not.
  ```
