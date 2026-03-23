---
title: (Sparta_WIL_02) 스프링 기본기
date: 2026-03-23 10:00:00 +09:00
categories: [Sparta, SpartaWIL02]
tags: [ Java, Spring Framework, SpringBoot ]
---

## 1. Log
### 1. New Keyword
 - RestAPI 패턴 
 - Builder / MapStruct - JPA사용하려면 익숙해져야하는 부분
 - 영속성 컨텍스트 - 어려움

## 2. 문제의 기록
 - 엔티티를 생성하는 방법
   - 엔티티를 생성할때는 생성자가 있어야 하는데,
   - PK나 FK가 걸려있으면 기본 생성자로 작업하는데 까다로워짐
   - 따라서 별도로 Request에서 요청오는 인자들만 별도로 생성자만들고
   - @Builder 해주면 조금 빠르게 작업은 가능함.
   - 그런데 서비스단에서 FK, PK를 넣어줘야함.
 - 엔티티 편의 메소드
   - 엔티티 자체를 수정할때 Setter를 쓰게 되면
   - 자동으로 작업해주는 기능들에서 Setter를 이용해서 뭔가 할 수가 있어서 조심해야함.
   - 별도로 편의메소드로 setter처럼 만들어주는게 유리.
   - 그리고 필요로 하는 추가 메소드도 편의 메소드!
 - 마이바티스는 SQL로 한큐에, JPA는 서비스단에서 거의 모든 작업을 !

## 3. 1주차 과제 관련
### 1. 작업내용
#### 1. 어드민 API - Category
 - 카테고리 등록 : POST /api/admin/categories
 - 카테고리 수정 : PUT /api/admin/categories/{categoryId}
 - 카테고리 삭제 : DELETE /api/admin/categories/{categoryId}

#### 2. 어드민 API - Product
 - 상품 등록 : POST /api/admin/products
 - 상품 수정 : PUT /api/admin/products/{productId}
 - 상품 옵션 수정 : PUT /api/admin/products/{productId}/options

#### 3.  사용자 API (User-Facing)
 - ProductController 및 ProductService 클래스 생성
 - 기능 미구현

### 2. 고민 & 질문
#### 1. CategoryId를 조회하는 반복 작업
 - AdminProductService 와 CategoryService에서 공통적으로 CategoryId를 확인하는 부분이 생겼습니다.
 - 어차피 로직이 동일해서 공통 컴포넌트를 만들면 만들 수는 있을 것 같은데요,
 - 근데 클래스 자체가 의존하는 게 너무 늘어나는 느낌이라, 각 클래스 내에서 private로 분리해서 사용하는 방법을 채택했습니다.
 - 이거를 그냥 지금처럼 서비스단에 두고 처리하는 게 낫은 방법일까요?

#### 2. Entity
 - 엔티티를 생성하고 편의성 메소드만들고 요런 부분들이 익숙하지 않은 상태라, 혹시 놓치거나 불필요한 부분이 있을까요?



