---
title: 04 자바종류
date: 2025-09-24 10:00:00 +09:00
categories: [note, JavaNote]
tags: [Java]
---

## 1. 자바 버전
### 1. 자바 종류

| 용어                                          | 의미             | 특징                                                     |
| ------------------------------------------- | -------------- | ------------------------------------------------------ |
| JDK (Java Development Kit)                  | 자바 개발 도구       | 컴파일러(javac), JVM 실행기(java), 표준 라이브러리 포함                |
| Java SE (Standard Edition)                  | 자바 표준 플랫폼      | JDK 위에 표준 라이브러리 제공, 콘솔/데스크톱/기본 앱 개발 가능                 |
| Java EE (Enterprise Edition, 현재 Jakarta EE) | 엔터프라이즈용 확장 플랫폼 | Java SE 기반 + 서버/웹/기업용 기능 추가 (Servlet, JSP, JPA, JMS 등) |
| JRE (Java Runtime Environment)              | 실행 환경          | JDK 없이 자바 프로그램 실행만 가능, 컴파일러는 없음                        |

### 2. Java - JAVA SE - JAVA EE

| 용어                             | 언제 쓰는가                         | 특징/사용 예시                                                                                             |
| ------------------------------ | ------------------------------ | ---------------------------------------------------------------------------------------------------- |
| JDK (Java Development Kit) | 자바 애플리케이션을 개발할 때           | 컴파일러(`javac`), JVM 실행기(`java`), 표준 라이브러리 포함. 콘솔 앱, 데스크톱 앱, 웹앱 개발 시 필수                                |
| Java SE (Standard Edition) | 자바 표준 API/라이브러리를 이용해 개발할 때 | 컬렉션, 스레드, I/O, 네트워킹 등 기본 기능 제공. 단독 프로그램, 유틸리티, 간단한 서버/클라이언트 프로그램 개발에 사용                              |
| Java EE / Jakarta EE       | 웹/서버/엔터프라이즈 애플리케이션 개발 시    | Servlet, JSP, JPA, JMS, REST API 등 서버 사이드, 대규모 시스템용 기능 포함. WAS(Tomcat, WildFly, GlassFish 등) 환경에서 사용 |

### 3. Servlet 버전별 Java EE

| Servlet 버전 | 출시 시기                | 주요 특징/변화                                                                                   |
| ---------- | -------------------- | ------------------------------------------------------------------------------------------ |
| 2.5        | 2005 (Java EE 5)     | JSP 2.1 지원, EL(Expression Language) 2.1, 기본적인 웹 애플리케이션 처리                                  |
| 3.0        | 2009 (Java EE 6)     | Multipart 파일 업로드 표준 API, 애노테이션 기반 설정(`@WebServlet`, `@MultipartConfig`), 비동기 서블릿 지원 시작 |
| 3.1        | 2013 (Java EE 7)     | Non-blocking I/O(NIO) 지원, HTTP/2 준비, 확장된 비동기 처리                                        |
| 4.0        | 2017 (Java EE 8)     | HTTP/2 공식 지원, JSON 처리 API 표준 포함                                                        |
| 5.0        | 2020 (Jakarta EE 9)  | 패키지 이름 `javax.servlet` → `jakarta.servlet` 변경, 기존 코드 마이그레이션 필요                             |
| 6.0        | 2022 (Jakarta EE 10) | 최신 Jakarta EE 표준 적용, 새로운 API 업데이트, Servlet 자체 기능 개선                                        |

## 2. note
### 1. 자바 - Spring
#### 1. 기존
- 웹 애플리케이션을 만들 때 필요한 기본 요소
  - HTTP 요청/응답 처리 → Servlet API
  - 파일 업로드 → Multipart 처리
  - URL 매핑, 파라미터 파싱, 세션 관리 등
- 전통적으로 Servlet/JSP만 쓰면 개발자가 직접 이 모든 것을 처리해야했음.
  - 예: cos.jar로 MultipartRequest 직접 처리
  - URL 패턴에 맞춰 HttpServlet 구현
  - 세션, 쿠키, 요청/응답 파싱 직접 구현

#### 2. 스프링

| 기능           | Servlet 방식                       | Spring 방식                              |
| ------------ | -------------------------------- | -------------------------------------- |
| URL 매핑       | `web.xml` 또는 `@WebServlet` 직접 설정 | `@Controller`, `@RequestMapping` 사용    |
| Multipart 파일 | cos.jar / request parsing 직접     | `MultipartFile` + 자동 MultipartResolver |
| 세션/쿠키        | 직접 HttpSession 사용                | Spring MVC가 wrapper 제공                 |
| JSON 요청/응답   | 수동 변환 필요                         | `@RestController` + Jackson 자동 변환      |
 - Spring Framework, Spring Boot는 Servlet 기반 위에서 많은 기능을 추상화함.
 - 개발자가 Servlet API를 직접 다루지 않아도, Spring이 내부에서 Servlet/Jakarta EE API를 사용해 처리
 - 파일 업로드, 요청 파라미터, URL 라우팅, JSON 변환 등 대부분 자동화
