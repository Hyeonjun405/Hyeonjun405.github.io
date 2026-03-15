---
title: (Sparta_WIL_01) 1주차 익숙하지 않은 JPA
date: 2026-03-15 10:00:00 +09:00
categories: [Sparta, SpartaWIL01]
tags: [ Java, Spring Framework, JPA ]
---

## 1. Log
### 1. 공부와 생각해야 할 새로운 Key
 - Flyway => DDL 관련 로그를 남겨서 공통된 테이블을 바라보는 방법
 - Docker => 굳이 DB를 설치하지말고 Docker를 띄워서 진행하면 편함.
 - JPA => 엔티티 / 외래키 / N+1 패턴

## 2. 문제의 기록
### 1. 설계의 고민
 - 프로젝트에 투입되고 신규 시스템을 보거나 개발을 진행해야할 때, 밑바닥부터 테이블의 연관관계를 고민하거나 고려하지 않았음. 
 - 보통 선임이 있거나 어느정도 구축되어 있는 상태에서 진행을 하기 때문에 엔티티를 구성하는 경험이 부족함.
 - 그렇다면
   - 지금 운영하는 시스템도 누군가 의도를 가지고 만들었을텐데 의도가 뭘까?
   - 추가로 이 설계에 관한 패러다임을 가질 방법찾기

### 2. 깃을 다루는 방법
 - 깃을 다루는 것 자체가 익숙하지 않음.
 - 평소에 하던 것만 진행하기 때문에 pull, push 정도.. 
 - 이건 내용이 많은건 아닌것 같고 간단히 명령어나 사용방법, 구조 등등은 공부해서 조금 더 찾아봐야할듯함.

## 3. 1주차 과제 관련
### 1. 작업내용
 - 프로젝트 생성
   - Build Tool : Gradle (Java 21)
   - Database : PostgreSQL (localhost:5433/sparta)
   - Flyway migration location : classpath:db/migration
   - Swagger configuration
   - BaseEntity 생성
 - Product, Entity 생성
   - 상품 상태 관련하여 ProductStatus Enum 생성
 - Category Entity 생성
 - Production Entity 생성
   - 옵션 종류 관련하여 ProductOptionType Enum 생성

### 2. 고민이 되었던 부분과 어떻게 대응하셨는지 남겨주세요
 - Product Option에 option_type, option_value두고 이것에 대한 재고 stock를 둔 상황입니다.
 - 이때 상품이 상품 옵션을 1개만 선택할 수 있다면 재고가 문제가 없지만,
 - 상품이 두가지 이상의 옵션(빨강+L사이즈)을 동시에 가져야하는 상황이라면,
 - 한가지 옵셥으로 stock을 정의 내릴 수 없는 상황이 발생하였습니다.
 - 이때 두가지 방법이 있을 것 같습니다.
   - 사용가능한 옵션의 수를 컬럼으로 정해서, 사용자에게 최대 옵션의 수를 제공하는 방법
   - Product id를 FK로 둔 옵션들을 모아둔 엔티티를 만드는 방법
 - 그럼 큰 시스템이라면 회사내부 인프라 환경이나, 자바 등등 무엇을 고려해서 어떤 근거로 판단을 해야할까요..?


