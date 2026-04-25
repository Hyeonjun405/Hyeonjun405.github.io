---
title: (Sparta_WIL_03) 스프링 AI
date: 2026-03-28 10:00:00 +09:00
categories: [Sparta, SpartaWIL03]
tags: [ Java, Spring Framework, SpringBoot ]
---

## 1. Log
### 1. New Keyword
 - QuerlDSL -> 생각보다 쉬운데 생각보다 어렵다
 - JPA - Request/Response -> 새로운 개념은 아니지만 묶는것을 어려워한다는 것을 알게됨!
 
## 2. 문제의 기록
 - JPA / Request / Response
   - JPA의 문제가 아니라 Request와 Response를 최소한으로 쓰는 방법에 대한 문제
   - 뭔가 잘 되는 것 같기는 한데 그럼에도 불구하고 자꾸 오류가 있음
   - 계속 엉키거나 비슷한데 수가 많아지는 상황
   - 연습만이 살길이다!

## 3. 1주차 과제 관련
### 1. 작업내용
 - 어드민 API - Category
   - 카테고리 등록 : POST /api/admin/categories
   - 카테고리 수정 : PUT /api/admin/categories/{categoryId}
   - 카테고리 삭제 : DELETE /api/admin/categories/{categoryId}
 
 - 어드민 API - Product
   - 상품 등록 : POST /api/admin/products
   - 상품 수정 : PUT /api/admin/products/{productId}
   - 상품 옵션 수정 : PUT /api/admin/products/{productId}/options

 - 사용자 API (User-Facing)
   - ProductController 및 ProductService 클래스 생성
   - 기능 미구현

### 2. 고민 & 질문
#### 1. CategoryId를 조회하는 반복 작업
 - AdminProductService 와 CategoryService에서 공통적으로 CategoryId를 확인하는 부분이 생겼습니다.
 - 어차피 로직이 동일해서 공통 컴포넌트를 만들면 만들 수는 있을 것 같은데요,
 - 근데 클래스 자체가 의존하는 게 너무 늘어나는 느낌이라, 각 클래스 내에서 private로 분리해서 사용하는 방법을 채택했습니다.
 - 이거를 그냥 지금처럼 서비스단에 두고 처리하는 게 낫은 방법일까요?

## 4. 3주차 피드백
요구사항의 핵심 파이프라인을 끝까지 연결해 둔 점이 좋습니다. 특히 Spring MVC, QueryDSL, Spring AI를 한 흐름으로 묶으려는 시도가 분명했고, 단순 CRUD를 넘어서 자연어 검색 API를 완성하려는 학습 방향이 잘 보였습니다. 이제는 응답 규격과 예외 흐름만 다듬으면 훨씬 완성도가 올라가겠습니다.

📢 주요 피드백
- [도메인 설계와 연관관계] ✔️: Category-Product-ProductOption 관계를 JPA로 자연스럽게 풀었고, 자기참조 카테고리까지 포함해 학습 범위 안의 모델링을 충실히 수행했습니다. 연관관계를 엔티티 메서드로 연결하려는 습관도 좋은 출발입니다.
- [Spring MVC 요청 처리 흐름] ✔️: Controller → Service → Repository 역할이 분리되어 있어 요청을 어디서 받고, 어디서 비즈니스 로직을 처리하며, 어디서 조회하는지 읽기 쉽습니다. 과제 요구사항을 코드 구조로 옮기는 기본기가 잘 보입니다.
- [QueryDSL 동적 쿼리] ✔️: 가격·카테고리·상품명 조건을 null일 때 제외하는 방식으로 구현해 요구사항의 “조건이 있으면 추가, 없으면 생략” 의도를 잘 살렸습니다. 최신 등록순 정렬도 조회 요구사항과 맞게 반영했습니다.
- [LLM 기반 조건 추론] ✔️: 자연어 query를 ProductSearchCondition으로 변환한 뒤 DB 조회로 넘기는 흐름이 살아 있어 이번 과제의 핵심 주제를 제대로 구현했습니다. 카테고리 목록을 Tool로 제공한 점도 Spring AI 학습 범위와 잘 연결됩니다.
- [추천 결과 가공] ✔️: 조회 결과를 다시 추천 문구 생성에 활용한 점은 Step 3의 의도를 잘 반영했고, 단순 조회 API를 넘어 추천형 API로 확장하려는 시도가 보입니다. 사용자가 체감하는 응답을 만들려는 방향이 좋습니다.
- [응답 규격과 예외 처리] ❌: /api/products/search는 예시와 달리 ApiResponse.ok를 사용해 result/message 구조가 맞지 않고, 검색 결과가 없거나 상품 검색과 무관한 문장일 때 안내 문구 대신 예외가 발생합니다. 요구사항 문서와 실제 응답 형태가 어긋난 부분은 꼭 맞춰주는 편이 좋습니다.
- [조회 안정성] ❌: getProduct, getProducts, searchWithLLM 같은 조회 메서드에 readOnly 트랜잭션 경계가 없고, 상세 조회에서 LAZY 연관 객체를 DTO로 바꾸는 구조라 환경에 따라 추가 쿼리나 의존성이 생길 수 있습니다. 지금은 동작하더라도 학습 단계에서 이유를 알고 정리해 두는 것이 중요합니다.

👨‍🏫 개선 포인트
- [응답 규격과 예외 처리] 개선: 현재 searchProductWithLLM은 정상 응답에서도 data 필드로 나가고, 결과가 비면 NOT_FOUND_PRODUCT 예외로 끝납니다. 이번 과제는 “무관한 질문이면 빈 목록 + 안내 문구”가 중요한 요구사항이라, 검색 결과가 없을 때 recommended.message를 채워 정상 응답을 반환하도록 분기하는 것이 더 맞습니다. 동시에 컨트롤러도 okWithMessage 같은 형태로 맞춰 응답 스키마를 요구사항과 일치시키면 API 신뢰도가 크게 올라가고, 프론트 입장에서도 처리하기 쉬워집니다.
- [조회 트랜잭션과 로딩 전략] 개선: ProductDetailResponse 변환 시 productOption 같은 LAZY 연관 객체를 읽는데, 서비스 메서드에 조회용 트랜잭션 경계가 보이지 않아 환경에 따라 예외나 숨은 추가 쿼리가 생길 수 있습니다. 지금 단계에서는 @Transactional(readOnly = true)를 조회 메서드에 붙이고, 상세 조회처럼 연관 데이터를 꼭 쓰는 곳은 fetch join이나 전용 조회 쿼리로 가져오면 JPA 동작을 더 안정적으로 이해할 수 있습니다. 이렇게 해야 “왜 어떤 API는 빠르고 어떤 API는 느린가”를 코드로 설명할 수 있게 됩니다.
- [LLM 결과 사용 방식] 개선: 현재는 LLM이 반환한 categoryName을 바로 조회 조건으로 사용하고, 추천 상품도 첫 번째 결과를 그대로 대표 상품으로 선택합니다. 학습 과제 기준에서는 충분히 출발점이 되지만, 조금만 더 나아가면 LLM 출력이 비어 있거나 애매할 때를 대비한 기본값 처리와 추천 기준 검증을 서비스에서 한 번 더 해주는 편이 좋습니다. 이렇게 하면 프롬프트 성능이 흔들려도 API 결과가 덜 흔들리고, 서비스 로직의 책임도 더 분명해집니다.

🛎️ 추가 팁
- QueryDSL에서는 지금처럼 null이면 조건을 빼는 방식을 계속 연습해보면 좋습니다. BooleanExpression 메서드 분리와 BooleanBuilder를 언제 쓰는지 비교해 보면 동적 쿼리 감각이 더 빨리 잡히고, 조건이 늘어나도 코드가 덜 복잡해집니다.
- JPA 연관관계는 “엔티티가 편하게 연결되어 있다”와 “조회 쿼리가 효율적이다”가 항상 같지 않습니다. @EntityGraph, fetch join, FetchType.LAZY, OSIV 키워드를 같이 공부하면 왜 조회 API와 저장 API를 다르게 설계하는지 감이 옵니다.
- Spring AI 쪽은 “프롬프트 작성”보다 “구조화된 출력 검증”을 같이 보는 습관이 중요합니다. Structured Output, Tool Calling, fallback message 같은 키워드로 찾아보면 지금 구현을 훨씬 안정적으로 발전시킬 수 있고, LLM을 믿되 그대로 의존하지 않는 감각도 생깁니다.
