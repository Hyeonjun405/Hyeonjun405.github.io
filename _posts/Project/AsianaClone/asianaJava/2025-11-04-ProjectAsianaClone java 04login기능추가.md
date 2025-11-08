---
title: 04 Login 기능 추가
date: 2025-11-04 10:00:00 +09:00
categories: [01asianaClone, asianaJava]
tags: [ asianaClone ]
---


## 1. 기능요약
 - ID/PWD 로그인
 - OTP 기능
 - 일정횟수 이상 틀릴 경우 계정 Block처리
 - 회원가입 / 비밀번호찾기

## 2. Note
 - 스프링 시큐리티는 일단 보류, 직접 구현
 - Intent
   - 세션에 값을 두고 값이 있을 경우에는, 메뉴선택에서 로그인 제거
   - 모든 메뉴에 적용해야해서 fragments Header에 값을 저장하고 상단에 변경처리
   - 로그인할 로그인 이름 표기하기
   - OTP화면에서 리다이렉트 설정안하면 계속 문자 재발송됨 (지금 업무중인 프로젝트에서 누락건, 여기서는 개선)

 - Point
   - Redirect하는데 파라미터가 안타는 이유!
     - error MSG를 담아서 넘기는데 타임리프에서 컨버팅할떄 사용하지 않는 상황이 발생함.
     - addAttribute는 파라미터 형태로 전달, addFlashAttribute는 세션에 임시저장했다가 리다이렉트할때 모델에 담아서 전달
   - 계정 상태 확장
     - ID/PWD 인증과 OTP에서 실패할 경우 실패 이력을 DB에 추가 저장해야함.
     - ID/PWD, OTP 기타 인증들 자체는 불가피하게 상황에 따라서 추가해야하는 부분,
     - 실패 이력 또는 계정 상태 관리는 email만 있으면 관리되는 작업이므로,
     - 실패 또는 상태 변경에 관한 이력관리가 필요한 경우 `UserStatusService`에 추가하여 활용.

## 3. Java
### 1. Redirect하는데 파라미터가 안타는 이유!
 - 리다이렉트의 구분
  ```
   redirectAttributes.addAttribute(String name, Object value)
   redirectAttributes.addAttribute(Object value)
   redirectAttributes.addAllAttributes(Map<String, ?> attributes)

   redirectAttributes.addFlashAttribute(String name, Object value)
   redirectAttributes.addFlashAttributes(Map<String, ?> attributes)
  ```
 - 리다이렉트에 파라미터를 태우는 방법은 2종류로, addAttribute와 addFLashAttribute가 있음
 - 기존에는 계속 addAttribute로 값을 넘긴뒤에 필요한 작업하는 용도로 사용했기 때문에 문제가 없었음.
 - 지금 상황에서는 발생하는 오류에 대한 String값을 웹에서 컨버팅하는 용도로 사용할떄 필요.
 - 그럴 경우에는 addFlashAttribute을 사용해야함.
 - addAttribute는 쿼리파라미터로 넘겨서, 컨트롤러에서 DTO나 파라미터로 받아서 사용이 가능하고 컨버팅에 사용하려면 모델에 담아야함.
 - addFlashAttribute는 바로 모델에 담아서 컨버팅할 수 있게함.
 - 구분

   | 구분                | URL에 노출됨 | 다음 컨트롤러 파라미터에서 받기 | 화면에서 1회 표시      | 저장 위치   |
   | ----------------- | -------- | ----------------- | --------------- | ------- ㅊ|
   | addAttribute      | O        | O                 | 직접 Model에 넣어야 함 | URL 쿼리  |
   | addFlashAttribute | X        | 모델에 자동 포함됨        | O               | 세션(1회용) |


### 2. 계정 상태 확장
 - LoginController에 넣을까 고려는 했으나, 그럼 상황 또는 로직이 추가되면 계속 Controller를 검증 해야하므로 Service자체에서 진행.
 - Class
  ```
    @Component
    public class UserStatusService {
    
        @Autowired
        private UserStatusMapper mapper;
    
        public void increaseFailCount(UserVO user){
    
            int fail = mapper.updateFailureCountByUserId(user);
            if(fail >= 5) {
                mapper.lockAccount(user);
            }
        }
    } 
  ```
 
