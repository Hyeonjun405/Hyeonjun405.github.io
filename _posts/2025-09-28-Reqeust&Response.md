---
title: Spring Request&Response
date: 2025-09-28 10:00:00 +09:00
categories: [Spring Framework, SpringMVC]
tags: [ JAVA, Spring Framework, SpringMVC ]
---

## 1. 전체흐름
### 1. Reqeust (클라이언트 -> 서버)
 - DispatcherServlet: 프론트 컨트롤러, 모든 요청 진입점.
 - HandlerMapping: URL, 메서드, 애노테이션 등을 기반으로 어떤 컨트롤러 메서드를 호출할지 결정.
 - HandlerAdapter: 선택된 컨트롤러를 실제 실행할 수 있도록 어댑터 역할.
 - Controller(@Controller / @RestController):
   - 요청 파라미터를 받음 (HttpServletRequest, @RequestParam, @RequestBody, @ModelAttribute, DTO 바인딩).
   - 내부 로직 처리(Service 호출 등).
 - Model / @ModelAttribute: 뷰 렌더링이나 응답 데이터 전달을 위해 필요한 데이터 준비.

### 2. Response 흐름 (서버 -> 클라이언트)
 - Controller 리턴값:
    - String → View 이름
    - 객체 → @ResponseBody 로 JSON/XML 변환
    - ResponseEntity → 응답 상태/헤더/바디 직접 제어
 - ViewResolver: View 이름을 실제 뷰 객체(JSP, Thymeleaf 등)로 변환.
 - HttpMessageConverter:
    - @ResponseBody 나 ResponseEntity 사용 시,
    - 객체 → JSON, XML, String 등으로 변환.
 - DispatcherServlet: 최종 응답을 클라이언트로 전달.

## 2. Request에서 사용하는 객체 / 애노테이션
### 1. 파라미터 바인딩 계열
 - @RequestParam : 단일 파라미터 매핑 (쿼리스트링, form field).
   ```
   // /search?keyword=spring&page=2 -> keyword="spring", page=2
   // page가 없으면 기본값 1
   // @RequestParam Map<String,String>으로 요청 파라미터 전체를 Map으로 받기.
   @GetMapping("/search")
   public String search(@RequestParam String keyword,
   @RequestParam(defaultValue = "1") int page) {
       return "result";
   }
   ```
 - @PathVariable : URL 경로 변수 매핑.
   ```
   // /users/100 -> id = 100
   @GetMapping("/user/{id}")
   public String getUser(@PathVariable Long id) { ... }
   ```
 - @RequestBody : HTTP body(JSON, XML 등)를 객체로 변환. (HttpMessageConverter 동작).
   ```
   // 요청 JSON: {"name":"kim","age":20}
   // userDto.getName() = "kim", userDto.getAge() = 20
   @PostMapping("/user")
   public String save(@RequestBody UserDto dto) { ... }
   ```
 - @ModelAttribute : 요청 파라미터를 객체(DTO/VO)로 바인딩 + Model에 자동 등록.
   ```
   // 요청 파라미터: name=kim&age=20
   // → User 객체 생성: user.setName("kim"), user.setAge(20)
   // 동시에 model에 "user" 라는 이름으로 등록됨 
   @PostMapping("/user")
   public String save(@ModelAttribute User user) { ... }
   ```
   
### 2. 요청 자체에대한 정보 접근

| 구분                 | 설명                             | 예시                                                                                             |
| ------------------ | ------------------------------ | ---------------------------------------------------------------------------------------------- |
| HttpServletRequest | 서블릿 API 직접 접근 (파라미터, 헤더, 세션 등) | 요청: `/req?name=kim` → request.getParameter("name") = "kim"                                     |
| HttpSession        | 세션 데이터 접근 및 관리                 | 로그인 성공 시 `session.setAttribute("user","kim")` → 이후 요청에서 `session.getAttribute("user") = "kim"` |
| @RequestHeader     | 특정 HTTP 헤더 값 매핑                | 요청 헤더: `User-Agent: Chrome` → 파라미터 ua = "Chrome"                                               |
| @CookieValue       | 쿠키 값 매핑                        | 요청 쿠키: `token=abc123` → 파라미터 token = "abc123"                                                  |

## 3. Response 반환
### 1. Response 반환

| 반환 타입                                  | 역할 / 의미             | 특징 / 사용 예시                                                       |
| -------------------------------------- | ------------------- | ---------------------------------------------------------------- |
| String                                 | 뷰 이름(view name)     | ViewResolver가 뷰를 찾아 렌더링. 예: `"home"` → `/WEB-INF/views/home.jsp` |
| void                                   | 뷰 이름 생략             | URL 기준 기본 뷰 사용, HttpServletResponse로 직접 처리 가능                    |
| ModelAndView                           | 뷰 이름 + 모델 데이터       | `new ModelAndView("viewName").addObject("key", value)`           |
| Model / ModelMap / Map\<String,Object> | 뷰에 전달할 데이터만 담음      | 뷰 이름은 별도로 String 리턴, Model과 유사                                   |
| @ResponseBody / 객체                     | HTTP 응답 바디 직접 반환    | 객체 → JSON/XML/String 변환, REST API 주로 사용                          |
| ResponseEntity<T>                      | 상태 코드, 헤더, 바디 직접 제어 | REST API에서 상태 코드와 헤더까지 설정 가능                                     |
| String + "redirect:/url"               | 클라이언트 리다이렉트         | Redirect 시 사용, RedirectAttributes와 함께 사용 가능                      |

### 2. 컨트롤러 리턴 타입에 따른 처리
 - 컨트롤러가 반환하면 DispatcherServlet이 반환 타입을 보고 처리가 가능한 곳으로 보냄

   | 반환 타입                            | 처리 위치                                         |
   | -------------------------------- | --------------------------------------------- |
   | String, ModelAndView, Model, Map | ViewResolver → View 렌더링                       |
   | @ResponseBody / 객체               | HttpMessageConverter → HTTP 바디 변환(JSON/XML 등) |
   | ResponseEntity                   | HttpMessageConverter → 상태코드/헤더/바디 직접 제어       |
   | String + "redirect:/url"         | RedirectView → 302 Redirect 전송                |

#### 1. ViewResolver
 - 역할 : 컨트롤러가 반환한 뷰 이름(String) → 실제 View 객체(JSP, Thymeleaf 등) 로 변환
 - 동작 예시
   - 컨트롤러: return "home";
   - ViewReoslver: "home" -> /WEB-INF/views/home.jsp
 - 종류
   - InternalResourceViewResolver (JSP)
   - ThymeleafViewResolver (Thymeleaf)
 - 파일별 접근경로

   | View 타입   | 추천 경로                           | 특징                                            |
   | --------- | ------------------------------- | --------------------------------------------- |
   | JSP       | `/WEB-INF/views/`               | 직접 URL 접근 불가, InternalResourceViewResolver 사용 |
   | Thymeleaf | `src/main/resources/templates/` | Spring Boot 기본 경로, classpath 기준               |
   | HTML (정적) | `src/main/resources/static/`    | ViewResolver가 아닌 정적 리소스로 직접 제공                |

  - `NOTE`
     - 스프링부트에서 JSP를 사용한다는 것은 JSP를 서블릿 클래스로 변환해서, 내장 톰캣(Web Container)에서 실행하는 형태이고
     - Thymeleaf는 스프링이 데이터를 HTML 템플릿에 바인딩하여 바로 클라이언트로 반환하는 형태

#### 2. HttpMessageConverter
 - 역할: 컨트롤러가 반환한 객체 -> HTTP 응답 바디 로 변환 (JSON, XML, String 등)
 - 동작 예시
   - 컨트롤러: @ResponseBody User user
   - HttpMessageConverter: User 객체 → JSON → HTTP 응답 전송
 - 종류
   - MappingJackson2HttpMessageConverter (JSON)
   - Jaxb2RootElementHttpMessageConverter (XML)
