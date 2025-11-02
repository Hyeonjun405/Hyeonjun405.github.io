---
title: 01 Login 1~2
date: 2025-11-01 10:00:00 +09:00
categories: [01asianaClone, asianaLogin]
tags: [ asianaClone ]
---


## 1. 기능요약
 - ID/PWD 로그인
 - OTP 기능
 - 일정횟수 이상 틀릴 경우 계정 Block처리
 - 회원가입 / 비밀번호찾기

## 2. Note
 - Intent
   - 서비스를 가장 작은 단위로 쪼개서 구성 1개의 Service에 포함시킴. 
   - 컨트롤러에서는 전부 성공한다는 가정하에 로직을 구성하고, 패스워드 오류, OTP오기입 등등 케이스는 오류는 전부 Exception 처리해서 excetpionHandler에서 처리하도록 진행함.
   - 로그인 자체는 한번 구성해두면 변경할일이 거의없기 때문에 고정으로 사용하기 위한 방법으로 판단하고 진행.

 - Problem
   - OTP와 검증등을 넣기 시작하면서 Controller와 service가 엄청 뚱뚱해지는 상황이 발생함.
   - 컨트롤러에서 기능이 굉장히 많아지면서 뭔가 서비스를 수정하면 복합적으로 건드려야하는 상황이 발생.
     - 서비스의 각기 메소드는 영향없게 잘처리는 했으나, 원래 있어야했던 영향을 컨트롤러가 부담하게됨.
     - 지금보다 컨트롤러에 들어가던 것들을 더빼야함. => 기능의 결정을 컨트롤러가해서 간단히 서비스 수정해도 더 복잡해짐.
     - 컨틀롤러가 책임이 많아진다 / 단일책임 원칙 위배 => 이해완료      
   - 모든 실패와 오류들을 Excetpion으로 처리 했더니, 실제 Exception과 default가 묶어서 보내주는거랑 구분이 안됨.
     - 컨트롤러가 뚱뚱해지는 것을 막기 위해서 Exception으로 분기를 다처리했는데 더 복잡해짐.
     - **이 패턴 다음부터 절대 사용X, 사람들이 안하는데는 이유가 있다.**  

 - Solution
   - 1차로 service를 쪼개서 AccountService, AuthenticationService, OtpService 3개로 분리.
   - 기본 로직은 유지하고 퍼사드 패턴 활용해서 만들기 ★ 
   - 컨트롤러는 단순하게 URL 매핑과 세션저장 분기정도로만 활용필요.
  
 - Memo
   - AP가 돌아가고 무엇인가 요청과 응답이 있다면 무조건 누군가는 책임을 지게 되어있음.
     - 그렇다면 이 책임을 한곳에 몰지 않고, 레벨링을 잘해서 특정 클래스와 레벨단에서 처리하고 대응하도록 조절필요.     
     - OOP는 최대한 클래스에 힘을 빼고 비슷한것을 묶는 패턴으로 작업 필요.
     - 조금더 확장성있게 클래스를 쪼개는게 필요. 컨트롤러와 서비스로 1개씩으로만 작업할 것이 아님!
   - `로그인 기본 설계 패턴 파악하고, 이것을 바탕으로 다른 기능들 구현 필요할듯. 굳이 급하게 완성할 필요가 없을 듯.`
   - `일단 익숙하지 않은 패키지 구조와 설계부터 천천히 파악!.`
   
 - Question
   - 그럼 어디부터 어디까지 클래스를 단계와 레벨링을 해야하는걸까?  
  
## 3. 사용한 방식
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
| POST        | /login/createAccount   | createAccount   | 계정 생성 처리             | 처리 후 리다이렉트 (보통 `/login`)                  |

### 2. 실패처리(ExceptionHandler)

| 예외 클래스                  | 처리 메소드                            | 처리 내용                                                          | 리다이렉트 / 뷰                                   |
| ----------------------- | --------------------------------- | -------------------------------------------------------------- | ------------------------------------------- |
| AuthenticationException | handlerAccountStatusException     | 로그인 관련 예외 처리, 메시지를 Flash Attribute로 전달                         | `/login` 리다이렉트                              |
| OtpException            | handlerVerifyCredentialsException | OTP 관련 예외 처리, 메시지를 Flash Attribute로 전달. 시스템 오류일 경우 로그인 페이지로 이동 | `/login/otp` 리다이렉트 (일반), `login` 뷰 (시스템 오류) |

### 3. 서비스(LoginService)
#### 1. 분리전

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

#### 2. 분리후

| 클래스 | 메소드명                                         | 설명                        | 반환값 / 결과                    |
|-----| -------------------------------------------- | ------------------------- | --------------------------- |
|AuthenticationService| checkAccountStatus(UserVO user)              | 계정 상태 확인 (휴면/잠금/정상)       | 없음 (void)                   |
|AuthenticationService| verifyCredentials(UserVO user)               | ID/PWD 확인                 | 없음 (void)                   |
|AuthenticationService| checkSecondVerification(String userNo)       | 2차 인증 필요 여부 확인 (SKIP/OTP) | String (`"SKIP"` / `"OTP"`) |
|OtpService| sendOtp(String userNo)                       | OTP 발송                    | VerifyOtpVO                 |
|OtpService| verifyOtp(String otp, VerifyOtpVO verifyOtp) | 입력한 OTP 검증                | 없음 (void)                   |
|AccountService| findId()                                     | ID 찾기                     | 없음 (void)                   |
|AccountService| findPassword(String email)                   | 비밀번호 찾기                   | 없음 (void)                   |
|AccountService| registerAccount(AccountUserVO userVO)        | 계정 등록                     | 없음 (void)                   |
