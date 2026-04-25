---
title: (Sparta_WIL_05) 캐싱전략
date: 2026-04-12 10:00:00 +09:00
categories: [Sparta, SpartaWIL05]
tags: [ Java, Spring Framework ]
---


## 1. Log
### 1. Keyword
- JPA / Optional
  - JPA에서 엔티티 설정하는 부분 복습하고
  - Optional로 어떻게 사용하는지 파악하고 
  - 영속성에 대해서 공부하고
  - QueryDSL에 대해서 접근하면 될 듯
- 계속 기반을 건너뛰고 용도만 접근하려고 하니 어려움.  
  
## 2. 문제의 기록
- 엔드포인트에 대한 고민
  - URL은 어떤 기준으로 잡는게 좋을까 ?
  - 리스폰에서 애매한데 뭐는 뭘로 잡고 뭐는 뭘로하고?  
- Optional과 JPA 활용에 대한 고민
  - 서비스단에서 JPA로 조회하는 부분은 조회만 할 것.
  - JPA로 조회하고 튀어나온 Optional에서 조회된걸로 뭔가를 하는건 애매한듯
  - 딱딱 역할에 맞게만 정의할 것
- JPA에서 외래키가 잡혀있을때,
  - 가운데 엔티티에서 위아래 영속성을 끊어버리면
  - 그 엔티티는 붕떠서 사라짐!

## 3. 5주차 피드백
전체적으로 인증, 세션, Redis 연동, 장바구니까지 요구사항의 큰 줄기를 잘 구현하셨습니다. 특히, Spring Security 세션 흐름과 장바구니 도메인 분리를 시도한 점이 좋았습니다.

📢 주요 피드백
- [회원 가입/로그인/로그아웃/내 정보 조회 API 기본 구현] ✔️: 회원 가입, 로그인, 로그아웃, 상태 조회 엔드포인트가 모두 존재하고 흐름도 자연스럽게 연결되어 있습니다.
- [email unique 제약 조건 적용 및 예외 처리] ✔️: User 엔티티와 Flyway SQL 모두에 unique 제약이 반영되어 있고, 서비스에서 중복 이메일을 선확인하여 예외 처리까지 챙겼습니다.
- [로그인 시 사용자 정보 저장 및 인증 유지] ✔️: AuthenticationManager로 인증 후 SecurityContext를 세션 저장소에 보관하는 방식이라 인증 유지 목적은 충족했습니다.
- [비밀번호 암호화 적용] ✔️: BCryptPasswordEncoder를 사용해 저장 전 비밀번호를 암호화한 점이 좋습니다.
- [특정 URL 인증 없이 통과] ✔️: 화이트리스트 경로를 SecurityConfig에서 permitAll로 분리해 둔 점이 요구사항과 잘 맞습니다.
- [Spring Session Data Redis 설정] ✔️: @EnableRedisHttpSession과 Redis 설정을 사용해 세션 저장소를 Redis로 확장하려는 방향이 잘 보입니다.
- [장바구니 목록/추가/수정/삭제 API] ✔️: 장바구니 CRUD API가 모두 준비되어 있고 서비스 책임도 비교적 잘 분리되어 있습니다.
- [세션 인증을 통해 userId 추출] ✔️: @AuthenticationPrincipal을 통해 로그인 사용자 식별값을 가져오는 흐름이 일관적입니다.
- [이미 담긴 상품은 수량 증가] ✔️: 기존 CartItem이 있으면 addQuantity로 누적시키는 로직이 요구사항에 정확히 맞습니다.
- [장바구니 도메인/테이블 설계] ✔️: Cart와 CartItem을 분리하고 DB에 unique(cart_id, product_id)를 둔 점은 안정적인 설계입니다.
- [상품 정보 조인 조회] ✔️: CartItem 조회 시 product를 join fetch하고 응답에 productName을 포함해 도전 과제도 일부 잘 수행했습니다.

👨‍🏫 개선 포인트
- [로그인 세션 표현] 개선: 이번 과제의 힌트는 HttpSession에 USER_ID 같은 값을 직접 저장하는 방식에 더 가까웠는데, 현재 구현은 Spring Security의 SecurityContext 저장 방식으로 풀었습니다. 이 방법도 실무적으로 매우 자주 쓰이지만, 과제 의도상 “세션에 무엇이 저장되는지”를 더 명시적으로 보여주면 학습 효과가 커집니다. 예를 들어 session.setAttribute로 userId를 직접 넣어보거나, 현재 방식이라면 세션 안에 SecurityContext가 저장된다는 설명을 코드나 주석으로 남기면 이해가 더 또렷해집니다.
- [Redis 연동 검증] 개선: Redis 설정과 EnableRedisHttpSession은 잘 들어갔지만, “정말 Redis에 세션이 저장되는지”를 확인하는 마지막 한 걸음이 빠져 있습니다. 학습 과제에서는 설정 자체보다 검증 경험이 더 중요합니다. 로그인 후 Redis 키를 조회하거나 RedisTemplate으로 spring:session 관련 키를 출력해 보면 세션 클러스터링 개념이 훨씬 선명하게 잡힙니다.
- [장바구니 API 설계] 개선: 장바구니 기능 자체는 잘 동작하겠지만, GET 요청에서 RequestBody를 받는 구조는 HTTP 사용 관점에서 다소 어색합니다. 지금 과제에서는 큰 감점 포인트는 아니지만, 조회 조건은 보통 쿼리 파라미터로 받는 편이 더 자연스럽고 테스트하기도 쉽습니다. 또 도전 과제에서 가격까지 요구했다면 응답 DTO에 상품 가격도 함께 담아주면 요구사항 충족도가 더 높아졌을 것입니다.

🛎️ 추가 팁
- 장바구니 조회에서 현재는 상품명을 함께 가져오도록 잘 구성하셨습니다. 이후 조회 항목이 늘어나면 fetch join, @EntityGraph 같은 키워드를 함께 공부해 두면 N+1 문제를 예방하는 감각을 익히는 데 도움이 됩니다.
- Cart와 CartItem처럼 연관관계가 있는 도메인은 “어느 쪽이 주인인지”, “삭제는 어디까지 전파할지”를 같이 보는 습관이 중요합니다. 지금처럼 orphanRemoval과 cascade를 사용한 이유를 스스로 설명해보면 JPA 이해가 빠르게 깊어집니다.
- Spring Security 기반 세션 인증을 사용하셨기 때문에, 다음 단계에서는 SecurityContextPersistenceFilter와 HttpSessionSecurityContextRepository가 어떤 흐름으로 동작하는지 한 번 따라가 보시면 좋습니다. 이번 과제의 로그인 유지 원리가 훨씬 명확해질 것입니다.
