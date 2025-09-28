---
title: Spring Controller
date: 2025-09-28 10:00:00 +09:00
categories: [Spring Framework, SpringMVC]
tags: [ Java, Spring, SpringMVC ]
---

## 1. Controller
### 1. Controller 설정
```
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String sayHello(Model model) {
        model.addAttribute("message", "안녕하세요, 스프링 MVC!");
        return "hello"; // 뷰 이름 (hello.jsp 또는 hello.html)
    }
}
```
## 2. 애노테이션
### 1. 클래스 레벨

| 애노테이션             | 역할                                                                   |
| ----------------- | -------------------------------------------------------------------- |
| `@Controller`     | 일반적인 웹 컨트롤러, 뷰 이름을 반환할 때 사용                                          |
| `@RestController` | JSON, XML 등 HTTP 바디 직접 반환 시 사용 (`@Controller + @ResponseBody` 기능 포함) |

### 2. http 메서드

| 애노테이션             | 역할                                |
| ----------------- | --------------------------------- |
| `@RequestMapping` | 요청 URL과 HTTP 메서드(GET, POST 등)를 매핑 |
| `@GetMapping`     | GET 요청을 처리 (편리한 단축 애노테이션)         |
| `@PostMapping`    | POST 요청을 처리                       |
| `@PutMapping`     | PUT 요청을 처리                        |
| `@DeleteMapping`  | DELETE 요청을 처리                     |
| `@PatchMapping`   | PATCH 요청을 처리                      |

### 3. 요청 데이터 => 03 MVC - 02 Reqeust&Response

| 애노테이션             | 역할                         |
| ----------------- | -------------------------- |
| `@RequestParam`   | 쿼리 파라미터 값 매핑               |
| `@PathVariable`   | URL 경로 변수 매핑               |
| `@RequestBody`    | 요청 바디(JSON 등)를 객체로 변환      |
| `@ResponseBody`   | 반환값을 HTTP 응답 바디로 직접 출력     |
| `@ModelAttribute` | 폼 데이터나 모델 객체 바인딩, 뷰에 자동 추가 |

## 3. note
### 1. 서블릿과 컨트롤러
- 기존 순수 서블릿 방식에서는 URL마다 별도의 서블릿 클래스가 필요했지만
- 스프링 MVC에서는 애노테이션을 사용해 메소드 단위로 URL 매핑이 가능하므로, 비슷한 기능을 한 클래스 안에 통합해 관리할 수 있음.

| 항목         | 순수 서블릿                     | 스프링 MVC                                |
| ---------- | -------------------------- | -------------------------------------- |
| 클래스 단위     | URL마다 별도 서블릿 클래스 필요        | 한 Controller 클래스에 여러 메소드 매핑 가능         |
| 매핑 단위      | 클래스 단위(@WebServlet URL 지정) | 메소드 단위(@RequestMapping, @GetMapping 등) |
| 코드 관리      | 기능 유사해도 클래스가 많아져 관리 복잡     | 관련 기능을 한 클래스 안에 통합 가능, 유지보수 용이         |
| 비즈니스 로직 분리 | 대부분 서블릿 안에 직접 구현           | Controller → Service 계층으로 분리 가능        |
| 확장성        | 새로운 URL마다 클래스 추가 필요        | 메소드 추가만으로 기능 확장 가능                     |
| 애노테이션 활용   | 제한적, 주로 @WebServlet        | 풍부한 매핑/설정 애노테이션 활용 가능                  |
