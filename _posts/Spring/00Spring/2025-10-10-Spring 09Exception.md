---
title: 09 Spring Exceptionㅋ
date: 2025-10-10 10:00:00 +09:00
categories: [00Spring Framework, Spring]
tags: [ Java, Spring Framework, SpringBoot, Exception ]
---

## 1. Exception
### 1. Exception
 - 프로그램이 실행되는 도중, 정상적인 흐름을 방해하는 오류 상황을 예외(Exception)
 - 예외는 단순한 문법 오류(Syntax Error)가 아니라, 실행 중(Runtime) 발생하는 문제 
 - 코드가 예상치 못한 상태에 빠질 때 시스템이 이를 감지하고 처리 흐름을 바꾸기 위한 메커니즘

### 2. 일반적인 처리 흐름
 - 예외 발생 (Throw)
   - 코드 실행 중 오류 조건이 발생하면 예외 객체를 생성하고 던짐.
   - 예: Java의 throw new IOException()
 - 전달 (Propagate)
   - 예외가 발생한 메서드 내에서 처리되지 않으면 상위 호출 스택으로 전달
   - 이 과정을 "Exception Propagation"이라고 하며, 처리 가능한 코드 블록을 만날 때까지 계속 올라감.
 - 처리 (Catch)
   - 예외를 처리할 수 있는 블록(try-catch)에 도달하면, 그곳에서 예외를 잡아 적절한 조치를 취함.
   - 예를 들어, 로그를 남기거나, 사용자에게 오류 메시지를 보여주거나, 대체 로직을 실행
  
### 3. 다른 언어와의 비교

| 구분                   | C                    | Java                       | Python                 | JavaScript                 |
| -------------------- | -------------------- | -------------------------- | ---------------------- | -------------------------- |
| 기본 키워드               | 없음 (반환값 / errno 사용)  | try / catch / finally      | try / except / finally | try / catch / finally      |
| 예외 계층 구조             | 없음 (구조적 예외 처리 없음)    | Exception / Error 계층 구분 명확 | 모든 예외가 Exception 상속    | ECMAScript Error 기반        |
| Checked Exception 존재 | 없음                   | 있음                         | 없음                     | 없음                         |
| 예외 선언 필요             | 없음                   | `throws` 키워드로 명시           | 필요 없음                  | 필요 없음                      |
| 런타임 처리 방식            | 함수 반환값 또는 errno 값 확인 | JVM이 스택을 타고 전파             | 인터프리터가 예외 전파           | JS 엔진이 비동기/동기 분리 처리        |
| 비동기 예외 처리            | 없음 (직접 구현 필요)        | 제한적 (주로 동기 코드 중심)          | asyncio의 try/except    | Promise.catch 또는 try/catch |

## 2. Java Exception 처리

### 1. 계층 구조
  ```
  java.lang.Throwable
  ├── java.lang.Error
  └── java.lang.Exception
      ├── java.lang.RuntimeException
      └── Checked Exceptions (그 외 Exception)
  ```
### 2. Throwable / Error / Exception

| 구분            | 설명                                  | 주요 예시                                                                        | 특징                                                                      |
| ------------- | ----------------------------------- | ---------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Throwable | 자바 예외 처리 계층의 최상위 클래스. 예외와 오류 모두 상속. | —                                                                            | 모든 예외와 오류는 Throwable을 상속해야 함. 예외 처리의 최상위 개념.                            |
| Error     | JVM 내부의 심각한 문제나 시스템 오류를 나타냄.        | `OutOfMemoryError`, `StackOverflowError`, `VirtualMachineError`              | 시스템 수준의 치명적 오류. 보통 애플리케이션에서 직접 처리하지 않음. JVM 동작 중단 가능성 있음.               |
| Exception | 애플리케이션에서 처리 가능한 예외의 최상위 클래스.        | `IOException`, `SQLException`, `NullPointerException`, `ArithmeticException` | 프로그램 실행 중 발생하는 처리 가능한 오류. Checked Exception과 Unchecked Exception으로 구분됨. |

### 3. checkedException & uncheckedException

| 구분                      | 설명                                               | 처리 의무                           | 주요 예시                                                                           | 특징                                                        |
| ----------------------- | ------------------------------------------------ | ------------------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Checked Exception   | 컴파일 시점에 반드시 처리해야 하는 예외. 메서드 선언에 `throws`로 명시 필요. | 반드시 try-catch 또는 throws 처리해야 함. | `IOException`, `SQLException`, `ClassNotFoundException`                         | 주로 외부 I/O, 데이터베이스, 네트워크 등 외부 환경 요인에서 발생. 컴파일러가 처리 여부를 강제. |
| Unchecked Exception | 컴파일 시점에 처리 여부를 강제하지 않는 예외. 런타임 시 발생.             | 처리 선택 가능.                       | `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException` | 주로 프로그래머의 논리적 오류나 잘못된 코드에서 발생. RuntimeException을 상속.      |


## 3. Spring Exception 처리

### 1. Spring Exception 처리
#### 1. Spring Exception

| 구조                   | 전역 예외 처리 지원   | @ExceptionHandler 사용         | 전용 예외 처리 인터페이스           | 특징                                          |
| -------------------- | ------------- | ---------------------------- | ------------------------ | ------------------------------------------- |
| **Spring MVC**       | ✅             | ✅                            | HandlerExceptionResolver | DispatcherServlet 기반. 가장 체계적이고 다양한 처리 방식 제공 |
| **Spring WebFlux**   | ✅             | ✅                            | WebExceptionHandler      | 비동기(Reactive) 방식 예외 처리. 비동기 전역 처리 지원        |
| **Spring WebSocket** | ✅             | ✅ (@MessageExceptionHandler) | 없음                       | 메시지 단위 예외 처리. 지속 연결 기반                      |
| **서버리스**             | ❌ (환경에 따라 다름) | 일부 가능                        | 없음                       | 함수 단위 처리. 플랫폼 의존적                           |
| **Spring Batch**     | ❌             | ❌                            | SkipPolicy, RetryPolicy  | Step/Job 단위 예외 처리. HTTP 구조 없음               |


#### 2. DispatcherServlet과 Exception 흐름
  ```
  Client → DispatcherServlet → Controller → Exception 발생  
                                ↓
                    HandlerExceptionResolver → Exception 처리
  
  ```
  - 클라이언트 요청 → DispatcherServlet으로 전달
  - DispatcherServlet → 요청을 처리할 적합한 컨트롤러 검색 (HandlerMapping)
  - 컨트롤러 실행 → 예외 발생 시 해당 예외를 캡처
  - HandlerExceptionResolver가 예외 처리 담당
  - 예외 처리 결과 → 적절한 뷰(또는 ResponseBody) 반환


#### 3. HandlerExceptionResolver 동작 순서

| 순서 | HandlerExceptionResolver 종류         | 설명                                               |
| -- | ----------------------------------- | ------------------------------------------------ |
| 1  | `ExceptionHandlerExceptionResolver` | `@ExceptionHandler` 애노테이션이 있는 메서드 처리             |
| 2  | `ResponseStatusExceptionResolver`   | `@ResponseStatus`가 지정된 예외 처리                     |
| 3  | `DefaultHandlerExceptionResolver`   | 스프링 기본 예외 처리 (HttpMessageNotReadableException 등) |

#### 4. 주요 애노테이션 (@ExceptionHandler, @ControllerAdvice, @RestControllerAdvice)
- @ControllerAdvice
  - 전역(Global) 예외 처리를 위해 사용.
  - 모든 컨트롤러에서 발생한 예외를 공통적으로 처리 가능.
  - @ExceptionHandler와 결합해 사용.
  - 예시
  ```
  @ControllerAdvice
  public class GlobalExceptionHandler {
  
      @ExceptionHandler(value = Exception.class)
      public ResponseEntity<String> handleException(Exception ex) {
          return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                               .body("전역 예외 발생: " + ex.getMessage());
      }
  }
  ```

- @RestControllerAdvice
  - @ControllerAdvice + @ResponseBody 기능 통합.
  - REST API 예외 처리를 전역에서 적용할 때 사용.
  - JSON 형태로 예외 응답을 제공.
  - 예시
  ```
  @RestControllerAdvice
  public class ApiExceptionHandler {
  
      @ExceptionHandler(value = Exception.class)
      public Map<String, String> handleApiException(Exception ex) {
          Map<String, String> error = new HashMap<>();
          error.put("error", ex.getMessage());
          return error;
      }
  }
  ```

- @ExceptionHandler
  - 특정 컨트롤러 또는 클래스에서 발생하는 예외를 처리할 수 있다.
  - 메서드에 선언하며, 처리할 예외 타입을 지정한다.
  - 예시
  ```
  @ExceptionHandler(value = { NullPointerException.class })
  public ResponseEntity<String> handleNullPointer(Exception ex) {
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
    .body("NullPointerException 발생: " + ex.getMessage());
  }
  ```


### 2. Exception 처리 구현 방식
#### 1. 개별 컨트롤러에서 처리 (@ExceptionHandler)
  ```
  @RestController
  public class MyController {
  
      @GetMapping("/test")
      public String test() {
          throw new CustomException("테스트 예외");
      }
  
      @ExceptionHandler(CustomException.class)
      public ResponseEntity<String> handleCustomException(CustomException ex) {
          return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                               .body("컨트롤러 전용 처리: " + ex.getMessage());
      }
  }
  ```
   - 여기서 Exception은 별도로 선언이 필요함.
   - 해당 컨트롤러 전용 예외 처리.
   - 전역 처리기보다 우선 순위가 높음.
   - 코드 관리가 간단하지만, 컨트롤러마다 중복 코드가 생길 수 있음.

#### 2. 전역 처리용 핸들러 (@ControllerAdvice)
  ```
  @RestControllerAdvice
  public class GlobalExceptionHandler {
  
      @ExceptionHandler(CustomException.class)
      public ResponseEntity<String> handleCustomException(CustomException ex) {
          return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                               .body("전역 예외 처리: " + ex.getMessage());
      }
  
      @ExceptionHandler(Exception.class)
      public ResponseEntity<String> handleGeneralException(Exception ex) {
          return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                               .body("알 수 없는 예외 발생: " + ex.getMessage());
      }
  }
  ```
   - 전역적으로 예외를 처리 → 중복 코드 최소화.
   - 유지보수성이 높음.
   - 예외별 응답 표준화 가능.
   - REST API에서는 @RestControllerAdvice를 주로 사용.

#### 3. ResponseEntityExceptionHandler 상속 방식
  ```
  @RestControllerAdvice
  public class CustomGlobalExceptionHandler extends ResponseEntityExceptionHandler {
  
      @Override
      protected ResponseEntity<Object> handleMethodArgumentNotValid(
              MethodArgumentNotValidException ex,
              HttpHeaders headers,
              HttpStatus status,
              WebRequest request) {
  
          Map<String, String> errors = new HashMap<>();
          ex.getBindingResult().getFieldErrors()
            .forEach(error -> errors.put(error.getField(), error.getDefaultMessage()));
  
          return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
      }
  
      @ExceptionHandler(CustomException.class)
      public ResponseEntity<String> handleCustomException(CustomException ex) {
          return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                               .body("ResponseEntityExceptionHandler 처리: " + ex.getMessage());
      }
  }
  ```
   - 스프링 기본 예외 처리 기능 활용 가능.
   - 세밀한 커스터마이징이 가능.
   - 검증 실패, 메시지 변환 오류 등 다양한 예외 처리에 적합.
   - REST API 표준 응답 구조 설계에 유리.

### 3.  커스텀 Exception 설계
#### 1. Custom Exception 클래스 (에러 클래스)
  ```
  public class CustomException extends RuntimeException {
      private final String errorCode;
  
      public CustomException(String errorCode, String message) {
          super(message);
          this.errorCode = errorCode;
      }
  
      public String getErrorCode() {
          return errorCode;
      }
  }
  ```

#### 2. ErrorResponse DTO (에러를 넘겨주는 DTO)
  ```
  public class ErrorResponse {
      private String errorCode;
      private String message;
      private LocalDateTime timestamp;
  
      public ErrorResponse(String errorCode, String message) {
          this.errorCode = errorCode;
          this.message = message;
          this.timestamp = LocalDateTime.now(); // 필요한 정보 생성
      }
  
      // Getter, Setter 생략 
      // lombok처리
  }
  ```

#### 3. 예외 공통 처리 로직 (에러 발생시 Json리턴)
- 에러 핸들러
  ```
  @RestControllerAdvice
  public class GlobalExceptionHandler {
  
      private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
  
      @ExceptionHandler(CustomException.class)
      public ResponseEntity<ErrorResponse> handleCustomException(CustomException ex) {
          logger.error("CustomException 발생: {}", ex.getMessage(), ex);
  
          ErrorResponse errorResponse = new ErrorResponse(ex.getErrorCode(), ex.getMessage());
          return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
      }
  
      @ExceptionHandler(Exception.class)
      public ResponseEntity<ErrorResponse> handleGeneralException(Exception ex) {
          logger.error("알 수 없는 예외 발생: {}", ex.getMessage(), ex);
  
          ErrorResponse errorResponse = new ErrorResponse("INTERNAL_ERROR", "서버 내부 오류가 발생했습니다.");
          return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
      }
  }
  ```
- 실제 리턴값
  ```
  {
      "errorCode": "CUSTOM_001",
      "message": "테스트 예외 발생",
      "timestamp": "2025-10-10T18:15:00"
  }
  ```
