---
title: 05 API Gateway
date: 2026-02-19 10:00:00 +09:00
categories: [infra, MSAInflearn]
tags: [ infra, MSAInflearn ]
---

## 1. API Gateway
### 1. API Gateway
 - 네트워크 상에서 다른 네트워크로 들어가는 입구 역할을 하는 지점
 - 클라이언트(웹, 모바일 등)가 백엔드 API 서버에 접근하기 전에 거쳐가는 입구 역할을 하는 서버
 - ![내 그림](assets/img/infra/msa_inflearn/05/ApiGateway.png "이미지")

### 2. API Gateway를 사용하는 이유
 - 라우팅 (Routing)
   - 클라이언트 요청을 적절한 서비스로 전달하기 위해 사용
   - MSA구조에서 여러가지 서비스를 이용할 경우, 클라이언트에서 필요한 서비스에 접근하기 위해서는
   - 서비스의 수만큼 주소를 알아야하는 상황이 생김,
   - 이때 API Gateway를 사용하면 API Gateway 주소 하나로만 통신해서 알아서 분배하도록 설정함.
   
 - 공통 로직 처리
   - 인증, 로깅과 같이 여러 서비스에서 공통으로 사용하는 횡단 관심사 로직을 한 곳에서 처리
   - 특정 인증 방식이나 패턴을 사용중 변경해야할때,
     - 서비스마다 설정 변경하면 유지보수가 어렵지만
     - 게이트웨이에서 일괄처리후에 진행을 하면 간단해짐.

### 3. API Gateway를 구현할 때 많이 사용하는 툴

| 항목             | Spring Cloud Gateway | Kong        | Traefik           | Envoy (+ Istio)  | NGINX       |
| -------------- | -------------------- | ----------- | ----------------- | ---------------- | ----------- |
| 주 사용 계층        | 애플리케이션 레벨            | 플랫폼 Gateway | Cloud-native Edge | Service Mesh 진입점 | 인프라 프록시     |
| 주요 목적          | Spring MSA 라우팅       | API 관리 & 정책 | 컨테이너 자동 라우팅       | 서비스 간 트래픽 제어     | 고성능 라우팅     |
| 라우팅 방식         | 코드 & 설정              | 플러그인 정책     | 서비스 자동 탐지         | L7 정책 라우팅        | 설정 기반       |
| 인증/인가          | JWT/OAuth 필터         | 플러그인 제공     | 미들웨어              | mTLS, 정책 기반      | 기본 TLS      |
| Rate Limiting  | 필터/Redis             | 내장 기능       | 미들웨어              | 정책 기반            | 모듈 설정       |
| 서비스 디스커버리      | Eureka/Consul 연동     | DB/레지스트리    | Docker/K8s 자동     | Mesh 내부          | 별도 구성       |
| 관측성            | Spring Actuator      | 로그/메트릭      | Prometheus 연동     | 강력한 텔레메트리        | 로그 중심       |
| Kubernetes 친화성 | 보통                   | 매우 높음       | 매우 높음             | 매우 높음            | Ingress로 활용 |
| 운영 복잡도         | 낮음                   | 중간          | 낮음                | 높음               | 낮음          |
| 확장성            | 코드 확장                | 플러그인        | 동적 구성             | Mesh 확장          | 모듈 확장       |
| 적합 환경          | Spring 중심 MSA        | 엔터프라이즈 API  | 컨테이너 플랫폼          | 대규모 MSA          | 범용 인프라      |


## 2. Spring Gateway
### 1. Gateway / Reactive GateWay
 - ![내 그림](assets/img/infra/msa_inflearn/05/SpringGateWay 의존성.png "이미지")
 - Servlet 기반 Gateway → 동기 / 스레드 블로킹
 - Reactive Gateway → 비동기 / 논블로킹 (현재 표준)

### 2. Servlet 방식
 - Servlet Container(Tomcat 등)가 요청을 스레드로 처리( 요청 수 = 스레드 수 )
 - MVC에서 특정 URL을 다른 서버로 전달하던 방식과 같은 계열
 - 흐름
   - 요청 수신
   - 스레드 할당
   - 내부 서비스 호출 (대기)
   - 응답 반환
 - I/O 대기 동안 스레드가 묶임 (Blocking)

### 3. Reactive 방식(권장)
 - 결과가 준비될 때까지 기다리지 않고, 이벤트가 발생하면 그때 처리하는 방식
 - 흐름
    ```
    요청 → 이벤트 루프 등록 → I/O 요청 → 스레드 반환
                         → 결과 준비 이벤트 발생 → 응답 처리      
    ``` 
    - 스레드가 대기하지 않고 적은 스레드로 많은 요청 처리 가능해짐
 - 이벤트 기반으로 처리한다는 건
    - 파이프라인 자체에 요청했던 정보나 작업 데이터가 들어있어서
    - 스레드가 바뀌어도 그 작업이 뭔지 알 수 있고,
    - 어떻게 마무리해야하는지 파악이 됨.
    - 그래서 굳이 대기하지 않아도 처리가 가능해지는 상황이 옴.

### 4. 특징 비교

| 항목             | Servlet 방식(GateWay) | Reactive 방식(Reactive Gateway) |
| -------------- |---------------------|--------------------------------|
| 처리 방식          | 스레드 1:1             | 이벤트 루프                         |
| I/O            | Blocking            | Non-Blocking                   |
| 서버             | Tomcat              | Netty                          |
| 동시 처리          | 제한적                 | 매우 높음                          |
| MSA 적합성        | 보통                  | 매우 높음                          |
| Spring Gateway | Zuul                | Spring Cloud Gateway           |



## 3. Reactive GateWay
### 1. application.properties(yaml)
  ```
  server:
    port: 8000
  
  spring:
    cloud:
      gateway:
        server:
          webflux:
            routes:
              # api/users로 시작하는 모든 경로의 요청은 user-service(localhost:8080)으로 전달
              - id: user-service
                uri: http://localhost:8080  # 2) 여기로 보낸다.
                predicates:
                  - Path=/api/users/** # 1) 이 경로로 들어오는 요청들은 ↑
              
              - id: board-service
              uri: http://localhost:8081
              predicates:
                - Path=/api/boards/**    
                  
  ```

### 2. Service에서의 url
 - 내부와 외부 컨트롤러를 구분해서 사용하는 것이 현명함. (내부연결용 `/api/*`, 외부 연결용`/internel/*` 등)
 - 그다음 외부 클라이언트에서는 gateway접근만 허용을 하고,
 - 내부적(서비스간)에는 서로 소통할 수 있게 설정하면
 - 게이트웨이를 통과해야만 외부에서 접근할 수 있는 구조로 완성됨.

## 4. JWT
### 1. JWT
 - 서버가 사용자 정보를 서명된 토큰 형태로 클라이언트에 전달하고,
 - 클라이언트는 요청마다 이 토큰을 보내 인증을 증명하는 방식

### 2. JWT 구조
 - HEADER.PAYLOAD.SIGNATURE
 - Header
   ```
   {
    "alg": "HS256",
    "typ": "JWT"
   }
   ```
 - PAYLOAD
   ```
   {
    "sub": "user123",
    "role": "ADMIN",
    "exp": 1710000000
   }
   ```
 - Signature
   ```
    HMACSHA256(
    base64Url(header) + "." + base64Url(payload),
    secretKey
    )
   ```
   
### 3. 인증 흐름
 - 로그인 : 사용자 → ID/PW 전송
 - 서버 인증 성공 : 서버 → JWT 생성 후 전달
 - 이후 요청 : 클라이언트 → 헤더에 토큰 포함

### 4. JWT & 세션기반

| 항목      | 세션 | JWT   |
| ------- | -- | ----- |
| 서버 저장   | 필요 | 불필요   |
| 확장성     | 낮음 | 높음    |
| MSA 적합성 | 낮음 | 매우 높음 |
| 모바일/API | 불리 | 유리    |
| 즉시 로그아웃 | 쉬움 | 어려움   |


## 5. JWT&Spring MSA
### 1. Memo
 - Gateway에서 
   - 내부적으로 쓰는 것과 안쓰는 것을 구분하고
   - JWT 검증이 필요한지 안한지 확인하고 (단순 조회용/유저별 권한이 있어서 작성용)
   - JWT 생성을 위한 요청이면 생성으로 보내고
   - 그외의 요청은 JWT 검증을 진행해야하고
   - JWT(ID값으로 만든 토큰)을 복호화해서 각 서비스에 전달을 진행함.
 - 유저서비스에서 
   - 유저의 정보를 검증한뒤에 userID로 토큰을 생성하고(JWT방식은 암복호화는 제공된 기준이 많음)
   - 오로지 생성은 유저서비스에서만
 - 각 서비스에서
   - gateway에서는 있는 패턴인지 없는패턴인지 검증했다면
   - 여기서는 실제 권한이 있는건지 userID값으로 2차 인증을 진행 
   - JWT가 들어가지 않고 기존처럼 내부적으로 검증만함.

### 2. JWT 토큰생성 - UserService
#### 1. memo 
 - jwt 토큰 생성의 가장 기본 유형,
 - 유저서비스에서는 인증후에 JWT 토큰값을 만들고, gateWay에 돌려줘야함.
 - 토큰의 생성은 여기서만 진행.
 - 다른 서비스들에서는 토큰을 복호화해서 userID를 쓰게됨

#### 2. Controller
 ```
  @PostMapping("login")
  public ResponseEntity<LoginResponseDto> login(
     @RequestBody LoginRequestDto loginRequestDto
     ) {
     LoginResponseDto loginResponseDto = userService.login(loginRequestDto);
     return ResponseEntity.ok(loginResponseDto); //
  }
 ```

#### 3. 의존성/properties
 - 의존성 추가 ` implementation 'io.jsonwebtoken:jjwt-api:0.13.0' `
 - yaml에 암호화 key값
   ```
   jwt:
     secret: jscode-secret-1234-1234-1234-1234 # 임의의값
   ```

#### 4. service
 ```
  public LoginResponseDto login(LoginRequestDto loginRequestDto) {
    // ID/PWD 검사로직
    
    // JWT를 만들 때 사용하는 Key 생성 (공식 문서 방식)
    SecretKey secretKey = Keys.hmacShaKeyFor(
		   jwtSecret.getBytes(StandardCharsets.UTF_8)
		);

    // JWT 토큰 만들기
    String token = Jwts.builder()
        .subject(user.getUserId().toString()) // 유저 id로 토큰을 만듬
        .signWith(secretKey)
        .compact();
    
    return new LoginResponseDto( token ); //바디값으로 넘김
 ```

#### 5. DTO
 - loginResponseDto : tocken(String)

### 3. JWT tocken 인증 - Gateway
#### 1. memo
 - 게이트웨이에서 JWT 인증을 진행하고, 변환해서 내부로 보내는 역할을 진행해야함.
 - 해당 소스에서는 간단하게 검증하는 흐름파악용으로 상세하게 검증하는 것은 확인 필요
 - 추가로
   - Gateway는 JWT 토큰만 검증할 뿐, payload가 맞는 패턴이면 그대로 downstream 전달 가능
   - “실제 존재하는 사용자 ID인가?”까지는 알 수 없음
   - 공격자가 우연히 또는 악의적으로 payload를 조작하면
   - 존재하지 않는 ID가 넘어올 수 있음
   - 서비스에서 DB 조회 없이 처리하면 권한 탈취/데이터 오류 위험이 생김
   - Service가 최종 검증을 담당해야 안전함.
   - 유저 정보에 대한 최소한의 안전장치가 필요함.
 - 단계별 역할

   | 레이어     | 무엇을 막는가        | 목적                |
   | ------- | -------------- | ----------------- |
   | Gateway | 토큰 유효성, 서명, 만료 | 중앙 인증, 변조 방지      |
   | 서비스     | 세부 권한, 도메인 검증  | 비즈니스 보안 강화, 2중 방어 |


#### 2. 의존성/properties
 - 의존성 추가 ` implementation 'io.jsonwebtoken:jjwt-api:0.13.0' `
 - yaml에 암호화 key값
  ```
  jwt:
    secret: jscode-secret-1234-1234-1234-1234 # 기존에 사용한 임의값 동일하게 사용
  ```

#### 3. Filter Class 생성
  ```
  @Component
  public class JwtAuthenticationFilter extends AbstractGatewayFilterFactory {
  
    @Value("${jwt.secret}") // 프로퍼티스값
    private String jwtSecret;
  
    @Override
    public GatewayFilter apply(Object config) {
      return (exchange, chain) -> {
      
        // Reqeuest Header에서 토큰 가져오기
        String token = exchange.getRequest() // Reqeust에서 값가져오기
            .getHeaders()
            .getFirst("Authorization");
        System.out.println("토큰 : " + token);
        
        // 토큰이 없을 경우 401 Unauthorized로 응답
        if (token == null) {
          exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
          return exchange.getResponse().setComplete();
        }
  
        // SecretKey로 토큰 검증 및 Payload(userId 담겨있음) 가져오기
        // 공식문서의 해석 패턴.
        SecretKey secretKey = Keys.hmacShaKeyFor(jwtSecret.getBytes(StandardCharsets.UTF_8));
        String subject = Jwts.parser()
            .verifyWith(secretKey)
            .build()
            .parseSignedClaims(token)
            .getPayload()
            .getSubject();            
        System.out.println("userId : " + subject); //subject에 들어있는게 값
  
        // Payload를 X-User-Id 헤더에 담아서 Request 전달
        // (= 다른 마이크로서비스에 요청 전달할 때 userId 정보를 담아서 보냄)
        return chain.filter(
            exchange.mutate()
                .request(
                    exchange.getRequest()
                        .mutate()
                        .header("X-User-Id", subject) // 헤더에 복호화된 X-User-Id값을 보냄
                        .build()
                )
                .build()
        );
      }; // return (exchange, chain) end
    }
  }
  ```

#### 4. application.yaml
 ```
  # yaml에서 라우터 설정할때는 위에서부터 검증을 하고 아래로 넘어감.
  # 인증필요X가 위에 있으면 맨아래까지 안가고 인증X에서 끝나버림. 
 
  # 인증이 필요한 서비스 (글작성 / POST로 주고 받을때)
  # 글을 작성할떄 누가 작성하는지 확인하기 위해 filters 클래스를 이용해서 헤더에 값을 추가함.
  - id: board-service-jwt
    uri: http://localhost:8081
    predicates:
      - Path=/api/boards
      - Method=POST
    filters: # 필터를 추가함.
      - JwtAuthenticationFilter
      
  # 인증이 필요 X 서비스 (글 보기 / POST외에)
  - id: board-service
    uri: http://localhost:8081
    predicates:
      - Path=/api/boards/**
 ```
