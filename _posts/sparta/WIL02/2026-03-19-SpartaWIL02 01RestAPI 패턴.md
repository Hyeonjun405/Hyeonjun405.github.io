---
title: 01 RestAPI 패턴
date: 2026-03-19 10:00:00 +09:00
categories: [Sparta, SpartaWIL02]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - RestAPI를 사용할떄 응답에 대한 공통 처리 부분.
 - 특정한 패턴으로 값을 리턴할때
   - ResponseEntity를 사용해서 했었는데
   - 이 방법은 공통의 규칙을 만들어줘야하는 단점이 있었음.
   - 근데 그냥 아에 제네릭타입으로 ApiResponse를 만들면 공통처리가 가능함.
 - @RestControllerAdivce
   - 공통 예외처리 조금 신박!
   - 이런 패턴을 활용하면 View도 방법이 있을 것 같은데

## 2. Controller + APIResponse + Excepion 조합
### 1. Note 
 - API 리스폰을 두고 성공/실패 관려해서 어떻게 넘길지 결정함.
 - 성공의 경우에는
   - 기본타입으로 Data 필드에 태워서 보내면 되고
   - Custom Exception의 경우에는 필드에 에러코드와 상태를 담고,
 - @RestControllerAdvice 잡힌 핸들러에서
   - @ExceptionHandler(DomainException.class)특정 예외들을
   - API리스폰에다가 담아서 전송함.

### 2. @RestControllerAdvice
 - 스프링에서 전역 예외 처리 + 공통 응답 처리를 담당하는 어노테이션
 - Spring MVC 예외 처리 메커니즘
 - AOP는 아님!
 
   | 구분 | AOP    | @RestControllerAdvice |
   | -- | ------ | --------------------- |
   | 기반 | 프록시    | DispatcherServlet     |
   | 시점 | 실행 전/후 | 예외 발생 후               |
   | 대상 | 전 계층   | 컨트롤러                  |
   | 목적 | 공통 로직  | 예외 처리                 |


## 3. 전체 소스
### 1. RestController
 ```
  @GetMapping
  public ApiResponse<List<ProductResponse>> getAllProducts() {
    //리턴 타입을 Api Response 형태로
    return ApiResponse.ok(productService.getAllProducts());
  }
 ```

### 2. API Response 형태
 ```
  @Getter
  @Builder
  @NoArgsConstructor
  @AllArgsConstructor
  @JsonInclude(JsonInclude.Include.NON_NULL) // null 필드는 제외함.- > 만약 error이 null이면 error은 제외하고 리턴됨.
  @FieldDefaults(level = AccessLevel.PRIVATE)
  public class ApiResponse<T> {
  
    Error error; //에러필드
    T data; // T타입 데이터 필드
  
    //파라미터가 <없는> ok메소드
    public static <T> ApiResponse<T> ok() { 
      return ApiResponse.<T>builder().build();
    }
  
    //파라미터가 <있는> ok메소드
    public static <T> ApiResponse<T> ok(T message) {
      // 객체타입의 빌더를 사용해서 값을 리턴하게됨.
      return ApiResponse.<T>builder()
          // 데이터필드에 메시지를 담으면, 리턴할때 data타입에 담겨서 가게됨.    
          .data(message)
          .build();
    }
    // fail 메소드
    public static <T> ResponseEntity<ApiResponse<T>> fail(HttpStatus httpStatus, String errorCode,
        String errorMessage) {
      return ResponseEntity.status(httpStatus)
          .body(ApiResponse.<T>builder()
              .error(Error.of(errorCode, errorMessage))
              .build());
    }
  
    @JsonInclude(JsonInclude.Include.NON_NULL)
    public record Error(String errorCode, String errorMessage) {
  
      public static Error of(String errorCode, String errorMessage) {
        return new Error(errorCode, errorMessage);
      }
    }
  }
 ```

### 3. GlobalException
#### 1. Service에서 Exception
 ```
 public ProductResponse getProductById(Long id) {
    Product product = productRepository.findById(id)
        .orElseThrow(() -> new DomainException(DomainExceptionCode.NOT_FOUND_PRODUCT));
         //에러코드를 리턴함.
 ```
#### 2. Domain용 RuntimeException 
 ```
  // httpStauts 와 Code를 가지고 있는 DomainException 필드
  @Getter
  @FieldDefaults(level = AccessLevel.PRIVATE, makeFinal = true)
  public class DomainException extends RuntimeException {
  
    HttpStatus httpStatus;
    String code;
  
    public DomainException(DomainExceptionCode exceptionCode) {
      super(exceptionCode.getMessage());
      this.httpStatus = exceptionCode.getStatus();
      this.code = exceptionCode.name();
    }
  }
 ```

#### 3. Domain Exception Code
 ```
  @Getter
  @AllArgsConstructor //생성자 생성
  @FieldDefaults(level = AccessLevel.PRIVATE)
  public enum DomainExceptionCode {
    // 코드별 필드값들
    INVALID_TOKEN(HttpStatus.UNAUTHORIZED, "잘못된 토큰입니다."),
    EXPIRED_TOKEN(HttpStatus.UNAUTHORIZED, "만료된 토큰입니다."),
    MISSING_TOKEN(HttpStatus.UNAUTHORIZED, "토큰이 누락되었습니다."),
    UNAUTHORIZED_ACCESS(HttpStatus.UNAUTHORIZED, "인증되지 않은 접근입니다."),
    JSON_PROCESSING_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "Json 데이터 처리 중 에러가 발생하였습니다."),
    NOT_FOUND_PRODUCT(HttpStatus.NOT_FOUND, "상품 정보를 찾을 수 없습니다."),
    ;
  
    final HttpStatus status;
    final String message;
  } 
 ```

#### 4. GlobalExceptionHandler
 ```
  @Slf4j
  @Hidden
  @RestControllerAdvice // RestController에서 예외가 발생할 경우 여기서 잡힘 
  public class GlobalExceptionHandler {
  
    private static final String VALIDATION_ERROR = "VALIDATION_ERROR";
    private static final String SERVER_ERROR = "SERVER_ERROR";
  
    //예외를 잡을 Exception 구분
    @ExceptionHandler(DomainException.class)
    public ResponseEntity<ApiResponse<Void>> handleDomainException(DomainException ex) {
      log.warn("[DomainException] : code={}, message={}", ex.getCode(), ex.getMessage());
      //ApiResponse 객체의 fail 메소드
      return ApiResponse.fail(ex.getHttpStatus(), ex.getCode(), ex.getMessage());
    }
  
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<Void>> handleMethodArgumentNotValidException(
        MethodArgumentNotValidException ex) {
      String errorMessage = extractErrorMessages(ex);
      log.warn("[ValidationException] : {}", errorMessage);
      return ApiResponse.fail(HttpStatus.BAD_REQUEST, VALIDATION_ERROR, errorMessage);
    }
  
    @ExceptionHandler(BindException.class)
    public ResponseEntity<ApiResponse<Void>> handleBindException(BindException ex) {
      String errorMessage = extractErrorMessages(ex);
      log.warn("[BindException] : {}", errorMessage);
      return ApiResponse.fail(HttpStatus.BAD_REQUEST, VALIDATION_ERROR, errorMessage);
    }
  
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<Void>> handleException(Exception ex) {
      log.error("[Exception] : ", ex);
      String message = ex.getMessage() != null ? ex.getMessage() : "서버 오류가 발생하였습니다.";
      return ApiResponse.fail(HttpStatus.INTERNAL_SERVER_ERROR, SERVER_ERROR, message);
    }
  
    private String extractErrorMessages(BindException ex) {
      return ex.getBindingResult()
          .getAllErrors()
          .stream()
          .map(DefaultMessageSourceResolvable::getDefaultMessage)
          .collect(Collectors.joining(", "));
    }
  }
 ```
