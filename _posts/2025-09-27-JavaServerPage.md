---
title: JSP - JavaServerPage
date: 2025-09-27 10:00:00 +09:00
categories: [Java, Servlet]
tags: [ Java, Servlet ]
---


## 1. JSP
### 1. JSP(JavaServer Pages)
- HTML 코드 안에 Java 코드를 삽입해 동적인 웹 페이지를 생성할 수 있게 해주는 자바 기반 기술
- JSP는 실행되기 전에 서블릿 코드로 변환되고, 이후 컴파일되어 동작함.
- JSP 파일(.jsp)은 기본적으로 HTML 구조를 가지면서, 필요한 부분에만 Java 코드나 JSP 태그를 삽입해서 동적인 데이터를 표현함.

### 2. jsp 기본 흐름
 - 클라이언트 요청 수신
   - 브라우저가 HTTP 요청을 보냄 (GET /hello.jsp HTTP/1.1)
   - WAS(Tomcat 등)는 지정된 포트에서 TCP 소켓으로 요청 수신
 - HTTP 메시지 파싱 및 객체 생성
   - WAS가 원시 HTTP 메시지를 파싱
   - HttpServletRequest, HttpServletResponse 생성
   - 필요 시 세션(HttpSession)과 애플리케이션 컨텍스트(ServletContext) 연결
 - JSP → 서블릿 변환
   - 요청 URL이 JSP라면 WAS가 해당 JSP 파일을 서블릿 클래스(.java)로 변환
   - 변환된 서블릿 클래스가 JVM에서 로딩되고, 인스턴스 생성 후 init() 호출
 - JSP 내장 객체 생성
   - WAS가 JSP 전용 내장 객체 생성하여 서블릿 코드에 바인딩
     - 요청/응답 관련: request, response
     - 출력 관련: out (JspWriter)
     - 세션/애플리케이션: session, application
     - 페이지 관련: page, pageContext, config
     - 예외 처리: exception
 - 요청 처리
   - WAS가 request/response 객체와 JSP 내장 객체를 서블릿 인스턴스의 service(request, response)에 전달
   - JSP 변환 서블릿 코드 실행 → 내부 자바 코드 수행 → HTML 문자열 생성 → out 객체로 출력
 - 응답 반환
   - JSP 실행 결과(out에 기록된 HTML)를 WAS가 HTTP 응답 메시지로 변환
   - TCP 소켓을 통해 브라우저로 전송
 - 정리
   - 요청이 끝나면 WAS가 request/response 객체 정리
   - JSP 서블릿 인스턴스는 JVM 메모리에 유지 → 다음 요청 재사용 가능
   
### 3. JSP → 서블릿 소스 변환
- hello.jsp
```
<html>
  <body>
    <h1>Hello, <%= request.getParameter("name") %></h1>
  </body>
</html>
```
- hello_jsp.java
```
 public final class index_jsp extends HttpServlet {
    public void _jspService(HttpServletRequest request, HttpServletResponse response)
            throws IOException, ServletException {
        response.setContentType("text/html;charset=UTF-8");
        JspWriter out = response.getWriter();

        out.write("<html><body>");
        out.write("<h1>Hello, ");
        out.write(request.getParameter("name"));
        out.write("</h1>");
        out.write("</body></html>");
    }
}
```

### 4. JSP - DB
```
<%
// JSP 안에서 DB 연결 가능하기는 함. 
Connection conn = DriverManager.getConnection(url, user, password);
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT name FROM users");

    while(rs.next()){
        out.println("<p>" + rs.getString("name") + "</p>");
    }

    rs.close();
    stmt.close();
    conn.close();
%>
```
- 유지보수 문제: 화면 출력과 DB 로직이 섞이면 스파게티 코드가 됨
- 재사용성 문제: DB 처리 로직을 다른 화면에서 재사용하기 어려움
- 보안/트랜잭션 관리 문제: Connection, commit/rollback 관리가 어려움

### 6. JSP 한계

| 한계        | 설명                                |
| --------- | --------------------------------- |
| 역할 혼동     | 화면 + 로직 + DB 처리 혼합 → 스파게티 코드 발생   |
| HTML 친화성  | 스크립틀릿 때문에 HTML 구조 깨짐, 디자이너 수정 어려움 |
| 동적 UI 한계  | Ajax, SPA, WebSocket 대응 어려움       |
| 유지보수성     | 로직 혼합 → 테스트·재사용 어려움, 대규모 서비스에 비효율 |
| 최신 기술 호환성 | REST API, 프론트엔드 프레임워크와 연동 어려움     |
| 성능        | 최초 요청 시 컴파일 필요, 반복문 과다 시 서버 부하    |

## 2. jsp와 서블릿
### 1. 서블릿과 jsp

| 구분        | 서블릿(Servlet)                               | JSP(JavaServer Pages)        |
|-----------| ------------------------------------------ | ---------------------------- |
| 정의        | 자바 클래스 기반 서버 컴포넌트                          | HTML 기반 템플릿 안에 자바 코드 삽입 가능   |
|  코드 작성 방식 | 순수 자바 코드로 작성, `doGet()`, `doPost()` 메서드 사용 | HTML 중심, 필요한 부분만 자바 코드/EL 사용 |
| 역할        | 로직 처리, 요청/응답 제어 (Controller/Service 역할)    | 화면 출력, 데이터 표시 (View 역할)      |
| 실행 구조     | 개발자가 직접 작성 → 컴파일 후 실행                      | WAS가 자동으로 서블릿 변환 후 실행        |
| 장점        | 로직 제어에 유리, 구조적                             | UI 작성이 쉬움, HTML 친화적          |
| 단점        | 화면 출력 불편 (`out.println()` 남발)              | 로직 섞이면 스파게티 코드 위험            |

### 2. 생명주기

| 구분    | 서블릿(Servlet)                            | JSP                                        |
| ----- |-----------------------------------------| ------------------------------------------ |
| 로딩    | 개발자가 작성한 `.java` 컴파일 후 로딩               | JSP 파일이 요청되면 WAS가 `.java` 변환 후 `.class` 로딩 |
| 초기화   | `init()`                                | `jspInit()`                                |
| 요청 처리 | `service()` → `doGet()`, `doPost()`<br> ※ service() = 모든 요청이 들어오는 입구 | `_jspService()` (HTML+Java 코드 실행)          |
| 종료    | `destroy()`                             | `jspDestroy()`                             |
| 개발 관점 | 비즈니스 로직 중심                              | 화면(View) 중심                                |

