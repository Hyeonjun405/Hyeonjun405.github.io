---
title: Validation
date: 2025-09-28 10:00:00 +09:00
categories: [Spring Framework, Spring]
tags: [ Java, Spring Framework, Validation]
---

## 1. Validation
### 1. Spring Validation
 - 스프링은 Bean Validation(JSR-303/JSR-380) 표준을 기반으로 동작
 - 기본 구현체는 Hibernate Validator
 - DTO/VO 클래스 필드에 어노테이션으로 검증 조건을 부여하고, 컨트롤러에서 @Valid 또는 @Validated를 사용해 적용

### 2. 목적 
 - 데이터 무결성 보장: 잘못된 값이 시스템 내부로 들어오는 것을 방지.
 - 에러 예방: 비즈니스 로직 처리 중 오류 발생을 줄임.
 - 보안 강화: 악의적인 입력(SQL Injection, XSS 등) 차단.
 - 사용자 경험 개선: 잘못된 입력을 빠르게 피드백.

### 3. 주요어노테이션
#### 1. 단일항목(필드 하나만 보고 판단 가능한 제약들)

| 범주       | 어노테이션                                                                                                        | 설명                |
| -------- | ------------------------------------------------------------------------------------------------------------ | ----------------- |
| null/빈 값 | `@NotNull`, `@NotEmpty`, `@NotBlank`                                                                         | 값 존재 여부 검사        |
| 문자열      | `@Size(min, max)`, `@Pattern(regexp)`                                                                        | 길이/정규식 패턴 제한      |
| 숫자       | `@Min`, `@Max`, `@Positive`, `@PositiveOrZero`, `@Negative`, `@NegativeOrZero`, `@Digits(integer, fraction)` | 값의 범위, 부호, 자릿수 검사 |
| 날짜/시간    | `@Past`, `@PastOrPresent`, `@Future`, `@FutureOrPresent`                                                     | 시간 조건 검사          |
| 형식       | `@Email`, `@URL`                                                                                             | 특정 포맷 검사          |

#### 2. 상관 항목 검사(두개 이상의 필드를 보고 판단하는 제약)

| 방법                | 예시                                                                                 | 설명                                |
| ----------------- | ---------------------------------------------------------------------------------- | --------------------------------- |
| 클래스 레벨 어노테이션  | `@ScriptAssert(lang="javascript", script="_this.startDate.before(_this.endDate)")` | Bean 전체를 대상으로 특정 조건 검사            |
| 커스텀 제약 정의     | `@FieldMatch(first="password", second="confirmPassword")`                          | 두 필드가 같은 값인지 비교 (비밀번호 확인 등)       |
| 그룹 Validation | `@Validated(CreateGroup.class)` vs `@Validated(UpdateGroup.class)`                 | 같은 DTO지만 상황(등록/수정 등)에 따라 다른 규칙 적용 |
| 복합 조건         | 시작일 ≤ 종료일, 할인율 ≤ 100, 특정 플래그가 true일 때 다른 필드 필수 등                                   | 단일 항목으로 불가능, 반드시 커스텀 Validator 필요 |

## 2. 단일 적용
### 1. Controller
```
public class CalcController {
    @PostMapping("/sum")
    public ResponseEntity<?> sum(@Valid @RequestBody SumForm form, BindingResult bindingResult) {
    
        // Validation 실패할경우 bindingResult에 자동으로 담김
        if (bindingResult.hasErrors()) {
            return ResponseEntity.badRequest().body(bindingResult.getAllErrors());
        }

        // 비즈니스 로직: 숫자 합계
        int result = form.getNumber1() + form.getNumber2();

        return ResponseEntity.ok("합계: " + result);
    }
}
```
- @Valid
  - @Valid 붙은 파라미터 바로 다음에 BindingResult를 둬야함. 
  - 순서를 지키지 않으면 스프링이 Validation 에러를 BindingResult에 담아주지 않고, 바로 예외(MethodArgumentNotValidException)를 던져버려.
- bindingResult.hasErrors()

| 코드 요소                                 | 역할             | 상세 설명                                |
| ------------------------------------- | -------------- | ------------------------------------ |
| `bindingResult.hasErrors()`           | 에러 여부 확인       | `@Valid` 검사 후 오류가 있으면 true 반환        |
| `bindingResult.getAllErrors()`        | 에러 목록 반환       | `ObjectError` / `FieldError` 리스트를 반환 |
| `ResponseEntity.badRequest()`         | 400 응답 생성      | 클라이언트 요청이 잘못됐음을 의미                   |
| `.body(bindingResult.getAllErrors())` | 응답 바디에 에러정보 담기 | 에러 객체 리스트가 JSON으로 직렬화되어 내려감          |

### 2. DTO
```
public class SumForm { 
    @NotNull(message = "첫 번째 숫자는 필수 입력값입니다.")
    @Positive(message = "첫 번째 숫자는 양수여야 합니다.")
    private Integer number1;

    @NotNull(message = "두 번째 숫자는 필수 입력값입니다.")
    @Positive(message = "두 번째 숫자는 양수여야 합니다.")
    private Integer number2;

    // Getter & Setter
    public Integer getNumber1() {
        return number1;
    }

    public void setNumber1(Integer number1) {
        this.number1 = number1;
    }

    public Integer getNumber2() {
        return number2;
    }

    public void setNumber2(Integer number2) {
        this.number2 = number2;
    }
```
- @NotNull(message = "…")처럼 message 속성에 적어둔 문자열은 Validation 실패 시 반환되는 기본 에러 메시지

## 3. 다중 적용
### 1. 컨트롤러
```
public class CalcController {

    @Autowired
    private final CalcValidator calcValidator;

    /* 주입으로 생략
    public CalcController(CalcValidator calcValidator) {
        this.calcValidator = calcValidator;
    }
    */

    // SumForm에 대해 커스텀 Validator 등록
    @InitBinder("calcForm")
    public void initBinder(WebDataBinder binder) {
        binder.addValidators(calcValidator);
    }

    @PostMapping("/sum")
    public ResponseEntity<?> sum(@ModelAttribute("calcForm") @Valid SumForm form,
                                 BindingResult bindingResult) {

        if (bindingResult.hasErrors()) {
            return ResponseEntity.badRequest().body(bindingResult.getAllErrors());
        }

        int result = form.getNumber1() + form.getNumber2();
        return ResponseEntity.ok("합계: " + result);
    }
}
```
 - @InitBinder로 등록한 Validator(calcValidator)는 해당 컨트롤러 안에서만 적용

### 2. Validator(별도 검증기)
```
@Component
public class CalcValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return SumForm.class.equals(clazz); // SumForm만 검증
    }

    @Override
    public void validate(Object target, Errors errors) {
        SumForm form = (SumForm) target;

        if (form.getNumber1() == null) return; // 단일 항목은 @NotNull 담당
        if (form.getNumber2() == null) return;

        int sum = form.getNumber1() + form.getNumber2();
        if (sum > 100) {
            errors.reject("sum.exceed", "두 숫자의 합이 100을 초과할 수 없습니다.");
        }
    }
}
```
- 특정 클래스타입만 검증하는지 확인 
  ```
  public boolean supports(Class<?> clazz) {
    return SumForm.class.equals(clazz);
  }
  ```
  - clazz → WebDataBinder가 전달하는 검증 대상 객체의 클래스 정보
  - supports()는 이 Validator가 이 클래스 타입을 처리할 수 있는지 여부를 boolean으로 반환
  - 작업하려고 하는 타입이 SumForm이고, 지금 여기에 접근한 타입이 해당 SumForm과 동일한지 확인함
- ```SumForm form = (SumForm) target;``` : 초기에 타겟을 형변환 필요함.
- 문제가 없는 경우 통과 /아닌 경우 별도 표기필요
   ```
    // 문제가 없으면 void
  
    // reject(String errorCode, Object[] errorArgs, String defaultMessage)
  
    // 클래스 레벨(Global) 에러, 메시지 없이
    errors.reject(null, null, null);
  
    // 클래스 레벨(Global) 에러만 등록, 메시지 없이
    errors.reject("sum.exceed", null, null);

    // 필드 레벨 에러, 메시지 없이
    errors.rejectValue("number1", "positive", null, null);
   ```
  
