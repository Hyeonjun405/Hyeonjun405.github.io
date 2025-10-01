---
title: 06 Swagger
date: 2025-10-01 10:00:00 +09:00
categories: [00Spring Framework, Spring]
tags: [ Java, Spring Framework, SpringBoot, Swagger ]
---

## 1. Swagger
### 1. Swagger?
 - Swagger는 REST API를 설계·문서화·테스트할 수 있도록 도와주는 오픈소스 프레임워크
 - 단순한 문서 작성 도구가 아니라, API의 전체 라이프사이클을 지원하는 툴 모음

### 2. Swagger 주요 역할

| 역할                  | 설명                                              | 기대 효과                       |
| ------------------- | ----------------------------------------------- | --------------------------- |
| 문서화 (Documentation) | 작성된 REST API를 자동으로 읽어 사람이 이해하기 쉬운 문서 형태(UI)로 제공 | API 문서 작성 비용 절감, 최신 상태 유지   |
| 테스트 (Testing)       | Swagger UI에서 직접 API 요청·응답을 확인 가능                | 별도 툴(Postman 등) 없이 빠른 검증 가능 |
| 협업 (Collaboration)  | 프론트엔드·백엔드 개발자가 동일한 API 명세를 공유                   | 변경 사항 실시간 반영, 커뮤니케이션 오류 감소  |
| 코드 생성 (Codegen)     | OpenAPI 명세를 기반으로 클라이언트 SDK, 서버 스텁 코드 자동 생성      | 개발 초기 생산성 향상, 반복 작업 최소화     |

### 3. Swagger 구성

| 구성 요소                           | 설명                                                          |
| ------------------------------- | ----------------------------------------------------------- |
| OpenAPI Specification (OAS) | Swagger의 기반이 되는 API 명세서 표준. JSON 또는 YAML 형식으로 API 설계 내용을 기술 |
| Swagger UI                  | OAS 문서를 시각화하는 웹 인터페이스. API 문서 보기와 테스트 가능                    |
| Swagger Editor              | OAS 문서를 작성·수정할 수 있는 웹 기반 편집기                                |
| Swagger Codegen             | OAS 문서를 기반으로 서버 스텁 코드나 클라이언트 SDK를 자동 생성하는 도구                |
| Swagger Hub                 | Swagger 도구를 온라인으로 통합 관리하는 클라우드 플랫폼                          |
| Swagger CLI                 | 명세 생성, 검증, 코드 생성 등 Swagger 관련 명령어 실행 도구                     |


## 2. OAS(OpenAPI Specification)
### 1. OAS?
 - REST API를 표준화해서 설계·문서화하는 규격(명세서)
 - Swagger Specification이라는 이름이 **OpenAPI Specification (OAS)**으로 변경

### 2. OAS 특징 
 - REST API 구조를 표준화된 형식(JSON 또는 YAML)으로 표현
 - 개발자, 테스터, 프론트엔드·백엔드 팀이 동일한 API 설계를 공유
 - API 변경 시 명세서만 갱신하면 문서·테스트 환경이 자동 업데이트

### 3. 구성

| 구성 요소            | 설명                   | 예시                                      |
| ---------------- | -------------------- | --------------------------------------- |
| openapi      | OAS 버전 정보            | `"3.0.3"`                               |
| info         | API에 대한 기본 정보        | 제목(title), 설명(description), 버전(version) |
| servers      | API 서버 정보            | 기본 URL, 환경별 URL 등                       |
| paths        | API 경로(Path)와 메서드 정의 | `/users`, `/orders` 등                   |
| components   | 재사용 가능한 객체 정의        | 요청·응답 스키마(schemas), 보안Schemes 등         |
| security     | API 보안 요구사항 정의       | JWT, OAuth2, API Key 등                  |
| tags         | API 엔드포인트 그룹화        | 사용자 API, 주문 API 등                       |
| externalDocs | 외부 문서 링크             | API 사용 가이드 링크                           |


## 3. Swagger 활용
### 1. 의존성
#### 1. 스프링(스프링부트X)
- 순수 Spring에서는 자동 설정이 없기 때문에 설정 클래스와 함께 라이브러리를 추가해야함.

 ```
 <dependency>
     <groupId>io.springfox</groupId>
     <artifactId>springfox-swagger2</artifactId>
     <version>2.9.2</version>
 </dependency>
 <dependency>
     <groupId>io.springfox</groupId>
     <artifactId>springfox-swagger-ui</artifactId>
     <version>2.9.2</version>
 </dependency>
 ```

#### 2. 스프링부트
- springdoc는 스프링부트 전용 의존성
- org.springdoc 사용 권장

 ```
 <dependency>
     <groupId>org.springdoc</groupId>
     <artifactId>springdoc-openapi-ui</artifactId>
     <version>2.1.0</version>
 </dependency>
 ```

### 2. 스웨거 클래스
- 스프링부트의 경우 의존성 추가하는 경우에 자동으로 생성
- 스프링의 경우에는 별도 클래스 생성필요함.

  ```
  @Configuration
  @EnableSwagger2
  public class SwaggerConfig {
      @Bean
      public Docket api() {
          return new Docket(DocumentationType.SWAGGER_2)
                  .select()
                  .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
                  .paths(PathSelectors.any())
                  .build();
      }
  }
  ```

## 4. JWT / OAuth2
### 1. 스웨거 - JWT / OAuth2 
 - 테스트 과정에서 API가 인증(Authentication)이나 권한 부여(Authorization)를 필요로 한다면
 - Swagger UI에서 JWT나 OAuth2 같은 인증 정보를 입력하고 테스트할 수 있도록 지원함.

### 2. JWT (JSON Web Token)
#### 1. JWT
 - JWT는 사용자 인증 정보를 안전하게 전달하기 위한 토큰 기반 인증 방식
 - 사용자 로그인 성공 시 서버에서 JWT를 생성하여 클라이언트에 전달
 - 클라이언트는 이후 요청 시 JWT를 HTTP 헤더(Authorization: Bearer 토큰)에 포함
 - 서버는 JWT를 검증해 사용자의 신원을 확인

#### 2. 구성
 - Header: 토큰 타입, 해시 알고리즘 정보
 - Payload: 인증 정보(클레임, 사용자 ID 등)
 - Signature: 토큰 변조 방지 서명

#### 3. Note
 - 서버가 HTTP 요청을 받으면 header에 Authorization이라는 header Key에 JWT(헤더+페이로드+시그니처) 문자열을 value로 담아서 Response
 - 클라이언트는 다시 요청할때 Header에 받았던 Authorization을 담아서 주면 서버에서 인증함.
  ```
  GET /api/data HTTP/1.1
  Host: example.com
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiIxMjM0NTYiLCJuYW1lIjoiSm9obiBEb2UifQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
  ```

### 3. OAuth2
#### 1. OAuth2
 - 다른 서비스(리소스)에 대한 권한 위임을 위한 인증·인가 프로토콜
 - 주로 소셜 로그인(구글, 카카오, 페이스북 로그인)에 사용
 - 클라이언트 → 인증 서버 → 리소스 서버 순으로 인증 절차 진행
 - 발급된 액세스 토큰(Access Token)을 사용해 API 접근

#### 2. 역할 구분

| 서버                           | 역할                                     | 예시                                                                                    |
| ---------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------- |
| 인증 서버 (Authorization Server) | 사용자 인증 + 권한 부여 + Access Token 발급       | accounts.google.com (구글 로그인 서버)                                                       |
| 리소스 서버 (Resource Server)     | 실제 API/데이터 제공, Access Token 검증 후 요청 처리 | [www.googleapis.com](http://www.googleapis.com) (Google Drive API 서버, Gmail API 서버 등) |

#### 3. 흐름
 - 클라이언트 앱 → 인증 서버에 권한 요청
 - 인증 서버 → 사용자 인증 페이지 제공
 - 사용자가 로그인 + 권한 승인
 - 인증 서버 → 클라이언트에 Authorization Code 발급
 - 클라이언트 → 인증 서버에 Authorization Code로 Access Token 요청
 - 인증 서버 → Access Token 발급
 - 클라이언트 → Resource Server에 Access Token 전달
 - Resource Server → Access Token 검증 후 요청 처리
