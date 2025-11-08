---
title: 02 Login 생성
date: 2025-11-02 10:00:00 +09:00
categories: [01asianaClone, asianaJava]
tags: [ asianaClone ]
---


## 1. 기능요약
 - ID/PWD 로그인
 - OTP 기능
 - 일정횟수 이상 틀릴 경우 계정 Block처리
 - 회원가입 / 비밀번호찾기

## 2. Note
 - 기본 기능 구현 작업
 - Point
   - 로그인 자체에서는 별도의 에러날 상황이나 문제가 적은 편으로 대부분 아이디 또는 패스워드 틀림으로 만들어짐.
   - 따라서 기본 컨트롤러 자체는 전체다 성공한다는 전제하에 비즈니스 로직을 잡고,
   - 실패는 서비스로직 별로 개별 Custom Exception(RuntimException)을 만들고 Exception handler을 만드는 방식을 채택. 
   - 그렇다면 에러나는 모든 것들은 각기 Exception 별로 에서 처리가 가능함.
   - 그럼 로그인은 성공 했을떄 어떻게 흘러가는지만 집중하면 되는 상황 관리됨

## 3. 구성
### 1. 컨트롤러/메소드(LoginController)

| HTTP Method | URL                    | 메소드명            | 설명                   | 반환 뷰 / 리다이렉트                              |
| ----------- | ---------------------- | --------------- | -------------------- | ----------------------------------------- |
| POST        | /login                 | login           | 일반 로그인 처리 및 2차 인증 분기 | 2차 인증 여부에 따라 `/login/otp`, `/login` 리다이렉트 |
| GET         | /login/otp             | otpLoginPage    | OTP 입력 페이지           | `login/otp` 뷰                             |
| POST        | /login/otp             | otpVerify       | 입력한 OTP 검증           | `/` 리다이렉트                                 |
| POST        | /login/otp/resend      | resendOtp       | OTP 재발송              | `/login/otp` 리다이렉트                        |
| GET         | /login/findAccount     | findAccountPage | 계정 찾기 페이지            | `login/findAccount` 뷰                     |
| POST        | /login/findAccount     | findAccount     | 계정 조회 요청 처리          | `/login` 리다이렉트                            |
| GET         | /login/registerAccount | registerAccount | 계정 생성 페이지            | `login/registerAccount` 뷰                 |
| POST        | /login/createAccount   | createAccount   | 계정 생성 처리             | 처리 후 리다이렉트 (`/login`)                  |

### 2. 실패처리(ExceptionHandler)

| 예외 클래스                  | 처리 메소드                            | 처리 내용                                                          | 리다이렉트 / 뷰                                       |
| ----------------------- | --------------------------------- | -------------------------------------------------------------- |-------------------------------------------------|
| AuthenticationException | handlerAccountStatusException     | 로그인 관련 예외 처리, 메시지를 Flash Attribute로 전달                         | `/login` 리다이렉트                                  |
| OtpException            | handlerVerifyCredentialsException | OTP 관련 예외 처리, 메시지를 Flash Attribute로 전달. 시스템 오류일 경우 로그인 페이지로 이동 | `/login/otp` 리다이렉트 (일반),<br> `login` 뷰 (시스템 오류) |

### 3. 서비스(LoginService)

| 메소드명                                         | 설명                        | 반환값 / 결과                    |
| -------------------------------------------- | ------------------------- | --------------------------- |
| findId()                                     | ID 찾기                     | 없음 (void)                   |
| findPassword(String email)                   | 비밀번호 찾기                   | 없음 (void)                   |
| registerAccount(AccountUserVO userVO)        | 계정 등록                     | 없음 (void)                   |
| checkAccountStatus(UserVO user)              | 계정 상태 확인 (휴면/잠금/정상)       | 없음 (void)                   |
| verifyCredentials(UserVO user)               | ID/PWD 확인                 | 없음 (void)                   |
| checkSecondVerification(String userNo)       | 2차 인증 필요 여부 확인 (SKIP/OTP) | String (`"SKIP"` / `"OTP"`) |
| sendOtp(String userNo)                       | OTP 발송                    | VerifyOtpVO                 |
| verifyOtp(String otp, VerifyOtpVO verifyOtp) | 입력한 OTP 검증                | 없음 (void)                   |

## 4. Java
### 1. Controller
  ```
   // 계정 상태 확인
          authenticationService.checkAccountStatus(userVo);
          log.info("계정 상태 확인 완료");
          
          // ID/PWD 확인
          authenticationService.verifyCredentials(userVo);
          log.info("계정 ID/PWD 일치 확인");
  ```

### 2. Service
  ```
  String userStatus = authenticationMapper.isLocked(user);
  
  if(userStatus == null) throw new AuthenticationException("계정 정보 없음");
  
  switch(userStatus){
      case "ACTIVE": return; // 등록+ACTIVE 계정은 통과
      case "LOCKED" : throw new AuthenticationException("계정 잠금 상태");
      case "SUSPENDED": throw new AuthenticationException("계정 정지 상태");
      default : throw new AuthenticationException("계정 정보 없음");
  }
  ```

### 3. Handler
  ```
  @ControllerAdvice(assignableTypes = {LoginController.class})
  @Slf4j
  public class LoginExceptionHandler {
  
      @ExceptionHandler(AuthenticationException.class)
      public String handlerAccountStatusException(AuthenticationException ex, RedirectAttributes redirectAttributes) {
          redirectAttributes.addFlashAttribute("error", ex.getMessage());
          return "redirect:/login"; // 로그인 페이지로 다시 이동
      }
  
      @ExceptionHandler(OtpException.class)
      public String handlerVerifyCredentialsException(OtpException ex, RedirectAttributes redirectAttributes) {
  
          if("시스템오류".equals(ex.getMessage())) return "login"; // 로그인 페이지로 다시 이동
  
          redirectAttributes.addFlashAttribute("error", ex.getMessage());
          return "redirect:/login/otp";
        }
  }
  ```
