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
