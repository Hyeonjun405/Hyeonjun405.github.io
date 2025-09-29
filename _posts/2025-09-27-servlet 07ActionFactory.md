---
title: 07 ActionFactory
date: 2025-09-27 10:00:00 +09:00
categories: [Java, Servlet]
tags: [ Java, Servlet ]
---

## 1. ActionFactory
### 1. ActionFactory
 - 팩토리 패턴 + 액션(Action) 객체 구조”를 결합한 웹 애플리케이션 설계 패턴

#### 1. 액션(Action) 객체
 - 요청 하나하나를 처리하는 작은 책임 단위 클래스
 - 예: LoginAction, LogoutAction, RegisterAction
 - 각 Action은 execute(request, response) 같은 공통 메서드를 가지고 처리

#### 2. 팩토리(Factory)
 - 요청 URL이나 명령(Command)에 따라 적절한 액션 객체를 생성
 - 서블릿은 팩토리에서 액션을 받아 실행만 함
 - 이렇게 하면 서블릿이 객체 생성 책임을 가지지 않음

#### 3. 서블릿(Servlet)
 - 클라이언트 요청 수신
 - 팩토리에 액션 요청 전달 → 액션 실행
 - 응답 처리

## 2. 구분
### 1. 액션 객체 분류
 - 요청의 성격에 따라 액션(Action) 객체를 분리
 - 요청 처리 로직과 화면 로직이 섞이지 않고, 역할별로 책임이 명확해짐.
 - 예시
    - 로그인 관련 액션: LoginAction, LogoutAction, RegisterAction
    - 화면 이동용 액션: HomeAction, ProfileAction, DashboardAction

### 2. 팩토리 분류
 - 실제로 팩토리도 기능 단위로 나눔.
 - 서블릿에서는 단순히 “팩토리에게 요청 객체 줘, 액션 객체 받아서 실행”만 하면 됨.
 - 서블릿은 어떤 액션이 실행되는지 알 필요가 없고, 새로운 액션이 생겨도 서블릿 코드를 수정할 필요가 거의 없음.
 - 예시
    - LoginActionFactory → 로그인 관련 액션 객체만 생성
    - PageActionFactory → 화면 이동/뷰 관련 액션 객체 생성

## 3. 예시 소스
### 1. Action 인터페이스
```
public interface Action {
    void execute(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

### 2. 액션 구현
```
public class LoginAction implements Action {
    @Override
    public void execute(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String id = request.getParameter("id");
        String pw = request.getParameter("pw");
        // 로그인 로직 처리
        response.getWriter().println("Login Success: " + id);
    }
}

public class LogoutAction implements Action {
    @Override
    public void execute(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // 로그아웃 처리
        response.getWriter().println("Logout Success");
    }
}
```

### 3. 액션팩토리
```
public class ActionFactory {
    public static Action getAction(String command) {
        if ("login".equals(command)) {
            return new LoginAction();
        } else if ("logout".equals(command)) {
            return new LogoutAction();
        }
        return null;
    }
}
```

### 4. 서블릿
```
@WebServlet("/controller")
public class FrontControllerServlet extends HttpServlet {
    
    @Override // 서비스를 오버라이드하면 get/post무관하게 여기로 다들어옴
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
        String command = request.getParameter("command"); // ?command=login
        Action action = ActionFactory.getAction(command); // command값 체크

        if (action != null) {
            action.execute(request, response);
        } else {
            response.getWriter().println("Unknown Command");
        }
    }
}
```

## 4. 액션팩토리 장점
### 1. 서블릿의 단순함과 확장성 문제 해결
 - 서블릿은 기본적으로 **요청(request) → 처리 → 응답(response)**의 흐름만 제공함.
 - 예를 들어 /login.do, /logout.do 같은 요청이 오면, 서블릿 하나에서 조건문(if-else 또는 switch)으로 분기하는 방식이 일반적
 - 요청이 많아지고 기능이 늘어나면, 서블릿 하나가 거대한 if-else 덩어리가 되어 유지보수와 확장성이 급격히 떨어짐.

### 2. 객체 생성 책임 분리
 - 각 요청마다 서로 다른 액션(Action) 객체를 만들어야 할 때, 매번 new LoginAction(), new LogoutAction() 같은 코드를 쓰면, 서블릿이 직접 객체 생성 책임까지 갖게됨.
 - 팩토리 패턴을 적용하면 서블릿은 요청만 전달하고, 실제 액션 객체 생성은 팩토리에서 처리하게 됨.
 - 객체 생성과 사용 책임을 분리해서 단일 책임 원칙(SRP)을 지킬 수 있음.

### 3. 유연한 확장과 유지보수
 - 새로운 기능(예: RegisterAction)을 추가할 때, 팩토리만 수정하면 되고, 서블릿 코드는 그대로 유지함.
 - 만약 조건문 기반이라면, 매번 서블릿 코드를 열어서 분기를 추가해야함.
 - 또한 팩토리를 활용하면 동적 로딩, 예를 들어 클래스 이름을 문자열로 받아 Class.forName()으로 생성하는 방식도 가능해서, 배포 시점에 액션 추가/수정이 더 유연함.


## 5. 서블릿을 사용한다면?
### 1. 서블릿 수가 급격히 늘어남
 - 요청마다 서블릿을 만들면, 로그인, 로그아웃, 회원가입, 프로필, 게시글 등 요청이 늘어날수록 서블릿 클래스 수가 폭발적으로 증가함.
 - 프로젝트가 커지면 패키지 구조 관리가 복잡해짐.
 - 웹.xml 또는 어노테이션으로 서블릿 매핑도 늘어나고, 관리 포인트가 많아짐.

### 2. 공통 처리 로직 중복
 - 로그인, 로그아웃, 회원가입 모두 요청/응답 처리, 예외 처리, 세션 관리 등 공통 로직이 필요함.
 - 서블릿을 개별로 만들면 공통 로직이 각 서블릿에 중복될 가능성이 높음
 - 팩토리나 공통 액션 구조를 쓰면 공통 처리 코드 재사용이 쉬워짐

### 3. 동적 확장성 제한
 - 팩토리를 쓰면 클래스 이름이나 요청 이름 기반으로 액션 객체를 동적으로 생성 가능 (Class.forName() 활용)
 - 서블릿을 개별로 만들면, 새로운 요청 추가 시 항상 클래스 생성 + 매핑 등록이 필요 → 배포 시점 수정이 필요하고, 동적 확장에는 취약합니다.
 - 테스트와 Mock 어려움 ( ※ Mock 객체 실제 객체를 대신해서 동작을 흉내 내는(fake) 객체 )
 - 서블릿 단위로만 나누면, 액션 단위 테스트가 어렵고, Mock 객체를 주입하기 힘듬. 
 - 팩토리 + 액션 구조면, 서블릿과 액션을 분리해서 액션만 단위 테스트 가능

## 5.note
### 1. MVC 기본 흐름 (클라이언트 → 컨트롤러 → 액션 → 서비스/DAO)
  - 클라이언트가 form이나 요청을 보냄
  - 컨트롤러(FrontController)가 요청 파라미터를 꺼냄
  - 컨트롤러에서 DTO/VO 객체를 생성하고, 파라미터 값들을 채움
  - 그 DTO/VO를 Action(또는 Service)에 넘겨줌
  - Action은 DTO/VO를 이용해 비즈니스 로직 실행 (DAO 호출, DB 작업 등)
  - 결과를 또 다른 DTO/VO에 담아서 request/session에 실어 JSP(View)에 전달
