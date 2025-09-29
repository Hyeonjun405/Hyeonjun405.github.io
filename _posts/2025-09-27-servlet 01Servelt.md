---
title: 01 Servlet
date: 2025-09-27 10:00:00 +09:00
categories: [Java, Servlet]
tags: [ Java, Servlet ]
---

## 1. Servlet
### 1. Servlet
 - **HTTP 요청-응답을 처리하기 위해 서버 사이드 자바 기반 서버 컴포넌트**
 - Java 기반 웹 애플리케이션에서 HTTP 요청을 받아 처리하고, 그 결과를 HTTP 응답으로 돌려주는 서버 측 컴포넌트
 - 브라우저 → 서버로 들어오는 요청을 자바 코드로 처리하게 해주는 역할

### 2. 주요개념

| 개념                    | 설명                                         |
| --------------------- |--------------------------------------------|
| HttpServletRequest    | 클라이언트 요청 정보 (파라미터, 헤더 등) 담고 있음             |
| HttpServletResponse   | 서버 → 클라이언트로 보낼 응답 (컨텐츠타입, 상태코드, body 등) 다룸 |
| web.xml / @WebServlet | 서블릿 URL 매핑 정보<br>실제 서블릿 클래스를 공개하지 않기 위해서 URL로 접근함.                       |
| Lifecycle             | init() → service() → destroy() 순으로 동작      |

### 3. Servlet 동작 흐름
- 클라이언트 요청 수신
    - 브라우저가 HTTP 요청을 보냄 (GET /hello HTTP/1.1)
    - WAS(Tomcat 등)는 지정된 포트에서 TCP 소켓으로 요청 수신
- HTTP 메시지 파싱 및 객체 생성
    - WAS가 원시 HTTP 메시지를 파싱
    - 요청 정보를 담은 HttpServletRequest, 응답을 담을 HttpServletResponse 객체 생성
    - 필요 시 세션(HttpSession)과 애플리케이션 컨텍스트(ServletContext)도 연결
- 서블릿 로딩 및 인스턴스화
    - WAS는 요청 URL에 매핑된 서블릿 클래스를 확인
    - 최초 요청이면 JVM이 클래스 로드, 메서드/필드 메모리 할당 후 서블릿 인스턴스 생성
    - init() 메서드 호출로 초기화 수행
- 요청 처리
    - WAS가 생성한 request/response 객체를 서블릿 인스턴스의 service(request, response)에 전달
    - service() 메서드에서 HTTP 메서드(GET/POST)에 맞는 doGet()/doPost() 호출
    - JVM 안에서 서블릿 코드 실행 → request에서 파라미터 읽고, response에 결과 작성
- 응답 반환
    - 서블릿 실행 후 response 객체에 담긴 내용 기반으로 WAS가 HTTP 응답 메시지 생성
    - TCP 소켓을 통해 브라우저로 응답 전송
- 정리
    - 요청이 끝나면 WAS가 request/response 객체 정리
    - 서블릿 인스턴스는 JVM 메모리에 유지 → 다음 요청 재사용 가능

### 4. 라이프사이클
 **객체나 컴포넌트가 생성되어 소멸될 때까지 거치는 단계와 상태 변화 과정**
```
 클래스 로딩 → 인스턴스 생성 → init() → service() → destroy()
```
 - 클래스 로딩 & 인스턴스 생성
   - 서버가 처음 요청을 받거나, 서블릿이 초기화될 때
   - .class 를 메모리에 로딩하고 Servlet 객체를 1개만 생성 (Singleton)
 - init(ServletConfig config)
   - 생성 직후 단 한번만 호출
   - 초기화 파라미터 세팅 등 서블릿 시작 준비 수행
   - 이 시점 이후에 요청 받기 시작함
 - service(HttpServletRequest req, HttpServletResponse resp)
   - 클라이언트 요청이 들어올 때마다 호출되는 메소드
   - HTTP 방식에 따라 내부에서 doGet(), doPost() 등 각 메소드를 호출
   - 핵심 비즈니스 로직이 실행되는 영역
 - destroy()
   - WAS가 종료되거나 서블릿이 언로드 될 때 단 한번 호출
   - 리소스 반납, 연결 종료 같은 정리(clean-up) 작업 수행

## 2. Request / Response 메소드 및 객체
### 1. 서블릿 별 http method
 ```
 protected void doGet(HttpServletRequest request, HttpServletResponse response) { ... }
 protected void doPost(HttpServletRequest request, HttpServletResponse response) { ... }
 protected void doPut(HttpServletRequest request, HttpServletResponse response) { ... }
 protected void doDelete(HttpServletRequest request, HttpServletResponse response) { ... }
 ```
- WebServlet으로 요청온 http method에 따라서 get/post/put/delete 등으로 각기 분류됨

### 2. HttpServletRequest 객체
 - HttpServletRequest 는 서블릿 간, JSP와 같은 서버 사이에서 데이터를 전달하는 용도
 - request 속성은 같은 요청 내에서만 유효
 - 메서드 종류
 
| 메서드                               | 반환 / 설명                 | 용도                       |
| --------------------------------- | ----------------------- | ------------------------ |
| `getParameter(String name)`       | String                  | GET/POST 파라미터 조회         |
| `getParameterValues(String name)` | String\[]               | 체크박스 등 다중값 조회            |
| `getParameterMap()`               | Map\<String, String\[]> | 전체 파라미터 Map 조회           |
| `getHeader(String name)`          | String                  | 요청 헤더 조회                 |
| `getContentType()`                | String                  | 요청 Content-Type 조회       |
| `getMethod()`                     | String                  | HTTP 메서드(GET, POST 등) 확인 |
| `getRequestURI()`                 | String                  | 요청 URI 조회                |
| `getContextPath()`                | String                  | 서블릿 컨텍스트 경로 조회           |
| `getSession()`                    | HttpSession             | 세션 객체 얻기                 |
| `getCookies()`                    | Cookie\[]               | 요청 쿠키 조회                 |
| `getReader()`                     | BufferedReader          | 요청 Body 문자 읽기 (JSON 등)   |
| `getInputStream()`                | ServletInputStream      | 요청 Body 바이트 읽기           |

### 3. HttpServletRequest 객체

| 메서드                                    | 설명                  | 용도                        |
| -------------------------------------- | ------------------- | ------------------------- |
| `setContentType(String type)`          | Content-Type 지정     | 클라이언트가 데이터를 올바르게 해석하도록    |
| `setCharacterEncoding(String enc)`     | 문자 인코딩 지정           | UTF-8 등                   |
| `setHeader(String name, String value)` | 응답 헤더 설정            | Cache-Control, Location 등 |
| `setStatus(int sc)`                    | 상태 코드 설정            | 200, 404, 500 등           |
| `sendError(int sc, String msg)`        | 에러 응답 전송            | 404 Not Found 등           |
| `sendRedirect(String location)`        | 클라이언트 리다이렉트         | 다른 URL로 이동                |
| `getWriter()`                          | PrintWriter         | 문자 기반 응답 출력               |
| `getOutputStream()`                    | ServletOutputStream | 바이너리 응답 출력 (파일, 이미지 등)    |
| `addCookie(Cookie cookie)`             | 쿠키 추가               | Set-Cookie 헤더 전송          |


## 3.Servlet에서 응답(Response) 처리 방식

| 방식          | 메서드                                                  | 동작                            | 특징               |
| ----------- | ---------------------------------------------------- | ----------------------------- | ---------------- |
| 직접 출력       | `response.getWriter()`, `response.getOutputStream()` | 응답 바디에 직접 작성                  | API, 파일 다운로드     |
| 내부 위임       | `RequestDispatcher.forward()`                        | 같은 서버 내 다른 JSP/Servlet이 응답 생성 | MVC 패턴           |
| 클라이언트 리다이렉트 | `response.sendRedirect()`                            | 브라우저가 새로운 요청 전송               | URL 변경, 요청 새로 시작 |


### 1. 직접 응답 작성 (스트림 이용)
#### 1. response 객체 이용
 - response.getWriter() → PrintWriter로 HTML 같은 텍스트를 직접 작성
 - response.getOutputStream() → 이미지/파일 같은 바이너리 데이터를 직접 작성
 - 특징
    - 응답 바디를 개발자가 직접 생성
    - 간단한 응답(“Hello World”)은 편하지만, 화면이 커지면 유지보수 어려움
    - 주로 API 응답(JSON/XML)이나 파일 다운로드 같은 경우 사용
#### 2. 예시 
```
@WebServlet("/hello") // URL 패턴 설정
public class FirstServlet extends HttpServlet { // HttpServlet 확장

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException { // http method

        resp.setContentType("text/html"); // 응답 타입 설정
        resp.setContentType("text/html; charset=UTF-8"); // 응답 타입 설정 + UTF8로 설정

        PrintWriter out = resp.getWriter(); // 리턴 객체 설정 
        out.print("<html><body><h1>"); // HTTP 형식으로 작성
        out.print("Hello Servlet");  // HTTP 형식으로 작성
        out.print("</h1></body></html>");  // HTTP 형식으로 작성
    }
```

### 2. 서버 내부 이용
#### 1. RequestDispatcher 객체 
 - RequestDispatcher.forward(request, response)
 - 다른 JSP/Servlet이 대신 응답을 만들어줌
 - 특징
    - 비즈니스 로직(Servlet)과 뷰(JSP)를 분리 가능
    - MVC 패턴에서 흔히 사용됨
    - forward된 대상이 최종적으로 response에 쓰기 때문에, 원래 Servlet이 직접 출력할 필요 없음
#### 2. 예시
```
@Override
protected void doPost(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException {

        String id = request.getParameter("id");
        String pw = request.getParameter("pw");

        
        if ("admin".equals(id) && "1234".equals(pw)) {
            // 로그인 성공 → main.jsp로 서버 내부 위임    
            
            //**서버 안에서 "xxx.html"이라는 리소스를 실행할 수 있는 전달자(RequestDispatcher 객체)**
            RequestDispatcher dispatcher = request.getRequestDispatcher("main.jsp");
            
            //서블릿에서 쓰던 request, response 객체 그대로 다음 리소스(JSP, HTML, 또 다른 서블릿 등)한테 전달됨.
            dispatcher.forward(request, response);
        } else {
            // 로그인 실패 → login.jsp로 다시 위임
            RequestDispatcher dispatcher = request.getRequestDispatcher("login.jsp");
            dispatcher.forward(request, response);
        }
    }
```

### 3. 클라이언트 리다이렉트
#### 1. redirect
 - response.sendRedirect("url")
 - 브라우저에 “다른 URL로 다시 요청하라”는 HTTP 302 응답을 보냄
 - 특징
   - 브라우저 주소창이 바뀜
   - 새로운 요청이기 때문에 request에 담긴 데이터는 초기화됨

#### 2. 예시
```
@WebServlet("/redirectExample")
public class RedirectServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        // 로그인 성공 시 메인 페이지로 리다이렉트
        resp.sendRedirect("main.jsp");
    }
}
```

#### 3. Forward - Redirect 

| 상황                | Forward | Redirect |
| ----------------- | ------- | -------- |
| 브라우저 URL 바꾸고 싶을 때 | X       | O        |
| POST-재전송 방지(PRG)  | X       | O        |
| 외부 서버 이동          | X       | O        |
| 서버 내부 요청 위임       | O       | X        |


### 3. Content-Type
**HTTP 메시지에서 전송되는 데이터의 형식(MIME 타입)을 나타내는 헤더(Header)**
#### 1. 요청구분
 - 요청 방향에 따라서 Content-Type의 역할과 Servlet에서 가지는 의미가 달라짐. 
  
   | 요청 방향           | 사용 목적                        | Servlet에서 의미                                             |
   | ------------ | ---------------------------- | -------------------------------------------------------- |
   | 요청(Request)  | 서버가 어떤 형식으로 데이터를 받았는지 알려줌    | `request.getContentType()` 으로 확인 → 어떤 방식으로 읽을지 결정        |
   | 응답(Response) | 클라이언트에게 어떤 형식으로 데이터를 보낼지 알려줌 | `response.setContentType()` 으로 지정 → 브라우저나 클라이언트가 올바르게 해석 |

#### 2. Content-Type 종류
  
  | Content-Type                      | 의미           | Servlet에서 처리 방법                                                        |
  | --------------------------------- | ------------ | ---------------------------------------------------------------------- |
  | application/x-www-form-urlencoded | 일반 HTML 폼 전송 | `request.getParameter()` 로 자동 파라미터 추출 가능                               |
  | multipart/form-data               | 파일 업로드 폼     | `Servlet 3.0` 이상: `@MultipartConfig` + `request.getPart()` 사용          |
  | application/json                  | JSON 데이터     | `request.getReader()` / `request.getInputStream()` 로 직접 읽어서 JSON 파싱 필요 |
  | text/plain                        | 일반 텍스트       | `request.getReader()` 로 읽음                                             |

#### 3. Request에서 파라미터 확인
  - QueryString과 Pom
  ```
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        // 쿼리스트링 파라미터 꺼내기
        String key1 = request.getParameter("key1"); // key1 = value1 
        String key2 = request.getParameter("key2"); // key2 = valeu2
    }
 ```

 - Json Type
 ```
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

      // 1. Request Body 획득
      StringBuilder sb = new StringBuilder();
      BufferedReader reader = request.getReader(); //버퍼로 한줄씩 읽음.
      String line;
      while((line = reader.readLine()) != null) {
          sb.append(line); 
      }

      String jsonString = sb.toString(); // JSON 문자열 획득 

      // 2. JSON 파싱 (예: org.json 사용)
      JSONObject json = new JSONObject(jsonString);
      String name = json.getString("name");
      int age = json.getInt("age");

    }
 ```
