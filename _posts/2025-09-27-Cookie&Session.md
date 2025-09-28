---
title: Cookie&Session
date: 2025-09-27 10:00:00 +09:00
categories: [Java, Servlet]
tags: [ Java, Servlet ]
---

## 1. 쿠키(Cookie)
### 1. 쿠키
 - 클라이언트(브라우저)에 작은 데이터를 저장하고, 이를 통해 서버가 사용자를 식별하거나 상태를 일부 유지할 수 있게 하는 수단
 - 용도 
   - 로그인 상태 유지 (단, 중요한 정보는 세션 사용)
   - 사용자 환경 설정 저장 (테마, 언어)
   - 장바구니 정보 일부 유지
 - 구조 
 
  | 항목                | 설명                 |
   | ----------------- | ------------------ |
   | Name              | 쿠키 이름 (Key)        |
   | Value             | 쿠키 값 (Value)       |
   | Domain            | 쿠키가 전송되는 도메인       |
   | Path              | 쿠키가 유효한 URL 경로     |
   | Max-Age / Expires | 쿠키 만료 시간 (초 또는 날짜) |
   | Secure            | HTTPS 전송만 허용 여부    |
   | HttpOnly          | JS에서 접근 불가, 서버 전용  |

### 2. 쿠키 메소드
| 구분     | 메소드                                 | 설명                                    |
| ------ | ----------------------------------- | ------------------------------------- |
| 생성     | `Cookie(String name, String value)` | 쿠키 이름과 값으로 생성                         |
| 이름/값   | `getName()`                         | 쿠키 이름 반환                              |
| 이름/값   | `getValue()`                        | 쿠키 값 반환                               |
| 이름/값   | `setValue(String value)`            | 쿠키 값 변경                               |
| 수명/만료  | `setMaxAge(int seconds)`            | 쿠키 유지 시간 설정 (0: 삭제, -1: 세션 종료 시까지)    |
| 수명/만료  | `getMaxAge()`                       | 설정된 쿠키 수명 가져오기                        |
| 경로/도메인 | `setPath(String uri)`               | 쿠키가 유효한 URL 경로 지정                     |
| 경로/도메인 | `getPath()`                         | 쿠키 경로 반환                              |
| 경로/도메인 | `setDomain(String domain)`          | 쿠키가 유효한 도메인 지정                        |
| 경로/도메인 | `getDomain()`                       | 쿠키 도메인 반환                             |
| 보안     | `setSecure(boolean flag)`           | HTTPS 전송만 허용 여부 설정                    |
| 보안     | `getSecure()`                       | 보안 설정 확인                              |
| 보안     | `setHttpOnly(boolean flag)`         | JS 접근 제한, 서버 전용 쿠키 설정                 |
| 보안     | `isHttpOnly()`                      | HttpOnly 여부 확인                        |
| 버전     | `setVersion(int v)`                 | 쿠키 버전 설정 (0 = Netscape, 1 = RFC 2109) |
| 버전     | `getVersion()`                      | 쿠키 버전 반환                              |

### 3. 쿠키 흐름
 1. 서버에서 쿠키 생성 → response 헤더에 Set-Cookie 추가
 2. 브라우저 저장
 3. 브라우저가 다음 요청 시 Cookie 헤더로 서버 전송
 4. 서블릿에서 request.getCookies()로 읽어 로직 처리
 5. 필요 시 삭제 → Max-Age=0 또는 만료 날짜 설정

### 4. 쿠키 사용
#### 1. 생성
    // 쿠키 생성
    Cookie cookie = new Cookie("userId", "alice");
    
    // 쿠키 옵션 설정
    cookie.setMaxAge(60*60); // 1시간
    cookie.setPath("/");      // 전체 경로에서 유효
    cookie.setHttpOnly(true); // JS 접근 제한
    
    // 응답에 쿠키 추가
    response.addCookie(cookie);
    
#### 2. 조회
    Cookie[] cookies = request.getCookies();
    for(Cookie c : cookies){
        if("userId".equals(c.getName())){
            String userId = c.getValue();
            System.out.println("쿠키 값: " + userId);
        }
    }

#### 3. 쿠키 제거
    Cookie[] cookies = request.getCookies();

    for (Cookie c : cookies) {
        if ("userId".equals(c.getName())) { // 삭제할 쿠키 이름
            c.setValue("");               // 값 초기화
            c.setMaxAge(0);               // 즉시 만료
            c.setPath("/");               // 기존 쿠키와 동일한 Path
            response.addCookie(c);        // 응답에 추가 -> 브라우저가 삭제
        }
    }
 
## 2. 세션(Session)
### 1. 세션
 - HTTP는 Stateless(상태 비저장) 프로토콜 → 요청 간 상태 유지 불가
 - 세션(Session)은 서버가 관리하는 사용자별 상태 저장 공간
 - 클라이언트마다 고유 ID(Session ID)를 발급하고, 서버에서 이 ID를 통해 사용자 상태를 추적
   - Session ID는 쿠키를 통해서 확인하고 매칭
   - 개발자는 Cookie부터 확인하는 것이 아니라 getSession()을 통해서 자동으로 매칭함. 
 - 용도
   - 로그인 상태 유지
   - 장바구니 데이터 관리
   - 사용자별 임시 데이터 저장
 - 구조

  | 구성 요소             | 설명                                                       |
   | ----------------- | -------------------------------------------------------- |
   | Session ID        | 클라이언트를 식별하는 고유 문자열 (예: `JSESSIONID`)                     |
   | 속성(Attribute) Map | Key-Value 형태로 세션 데이터 저장 (`setAttribute`, `getAttribute`) |
   | 생성 시간             | 세션 객체가 생성된 시각                                            |
   | 마지막 접근 시간         | 마지막 요청 시각, 비활성 시간 판단에 사용                                 |
   | 최대 비활성 시간         | `setMaxInactiveInterval()`로 설정 가능, 초 단위                  |
   | 상태                | 활성(active) / 만료(invalidated) 여부                          |

### 2. 세션 메소드
| 구분       | 메소드                                       | 설명                      |
| -------- | ----------------------------------------- | ----------------------- |
| 세션 생성/조회 | `getId()`                                 | 세션 ID 반환                |
| 세션 생성/조회 | `getCreationTime()`                       | 세션이 생성된 시간 반환 (ms)      |
| 세션 생성/조회 | `getLastAccessedTime()`                   | 마지막 접근 시간 반환 (ms)       |
| 세션 생성/조회 | `getMaxInactiveInterval()`                | 최대 비활성 시간(초) 반환         |
| 세션 생성/조회 | `setMaxInactiveInterval(int seconds)`     | 최대 비활성 시간 설정 (초)        |
| 속성 관리    | `setAttribute(String name, Object value)` | 세션 속성 저장                |
| 속성 관리    | `getAttribute(String name)`               | 세션 속성 조회                |
| 속성 관리    | `removeAttribute(String name)`            | 특정 속성 제거                |
| 속성 관리    | `getAttributeNames()`                     | 모든 속성 이름 Enumeration 반환 |
| 세션 무효화   | `invalidate()`                            | 세션 무효화, 서버에서 제거         |
| 상태 확인    | `isNew()`                                 | 새로 생성된 세션 여부 반환         |

### 3. 세션 흐름
 - 세션 생성
   - 클라이언트가 최초 요청 → request.getSession() 호출
   - 서버 메모리에 HttpSession 객체 생성
   - Session ID 발급 → 클라이언트 쿠키(JSESSIONID)로 전달
 - 데이터 저장
   - setAttribute(key, value) → 내부 Attribute Map에 저장
   - 여러 요청에서 값 공유 가능
 - 데이터 조회
   - getAttribute(key) → Map에서 값 조회
 - 세션 만료
   - 서버가 마지막 접근 시간 기준 비활성 시간 초과 시 세션 무효화
   - invalidate() 호출 시 즉시 세션 제거

### 4. 세션 사용
#### 1. 세션생성
   ```
   // 기존 세션 가져오기, 없으면 null
   HttpSession session = request.getSession(false);
   
   // 없으면 새 세션 생성
   HttpSession session2 = request.getSession(); // getSession(true)와 동일
   ```
#### 2. 세션사용
   ```
   //세션 저장
   session.setAttribute("userName", "Alice");
   session.setAttribute("cartItems", itemList);

   //세션 호출
   String userName = (String) session.getAttribute("userName");
   List<Item> cartItems = (List<Item>) session.getAttribute("cartItems");
   ```

#### 3. 세션 제거
   ```
   // 특정 속성 제거
   session.removeAttribute("cartItems");
   
   // 세션 전체 무효화 (로그아웃 시 주로 사용)
   session.invalidate();
   ```
