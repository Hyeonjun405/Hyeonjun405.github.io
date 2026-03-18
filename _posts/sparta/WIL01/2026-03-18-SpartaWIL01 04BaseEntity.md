---
title: 04 JPA BaseEntity
date: 2026-03-18 10:00:00 +09:00
categories: [Sparta, SpartaWIL01]
tags: [ Java, Spring Framework, JPA ]
---

## 1. Note
### 1. BaseEntity
 - 특별한 설정은 없지만 반복적으로 사용해야하는 필드들은
 - BaseEntity를 만들어서 Extends하여 공통된 필드를 추가하는 패턴을 사용함.
 - 각기 엔티티에따라서 옵션을 주거나 제거할수도 있음.

## 2. BaseEntity
### 1. BaseEntity
 - 공통 필드 + 공통 동작을 한 곳에 모아서 재사용하기 위한 추상 클래스
 - JPA에서는 생성/수정 시간, 작성자 같은 것들을 중복 없이 관리하기 좋음
 - 선언을 하게 되면 테이블 또는 엔티티가 생성되지 않고, 
   - extends를 사용해서 기존엔티티에 붙여서 사용함
   - 다중상속으로 여러개 상속가능함.

### 2. 이점
#### 1. 중복 제거 (가장 직접적인 효과)
 - 모든 엔티티에 반복되는 필드 제거
   - id
   - createdAt
   - updatedAt
 - 코드량 감소 + 실수 방지

#### 2. 변경 비용 최소화
 - BaseEntity만 수정하면 전체 반영
 - 개별 엔티티 수정 불필요하여 유지보수 비용 크게 감소

#### 3. Entity에 대한 강제성
 - 모든 테이블은 생성/수정 시간 가져야 한다라고하면
  - BaseEntity로 강제 가능
  - 누락되는 엔티티 없음

#### 4. 엔티티의 명확성
 - 공통 부분을 별도로 분리하여 엔티티 자체는 도메인에 가까워짐


## 3. BaseEntity 소스
### 1. BaseEntity
 ```
  package com.example.project.global.entity;

  import jakarta.persistence.*;
  import org.springframework.data.annotation.CreatedDate;
  import org.springframework.data.annotation.LastModifiedDate;
  import java.time.LocalDateTime;
  
  @MappedSuperclass // 테이블로 생성되지 않음, 필수로 사용해야함.
  @EntityListeners(AuditingEntityListener.class) // 생명주기를 관리하는 케이스, 커스텀은 가능함.
  public abstract class BaseTimeEntity {
  
      @CreatedDate
      @Column(updatable = false)
      private LocalDateTime createdAt;
  
      @LastModifiedDate
      private LocalDateTime updatedAt;
  
      public LocalDateTime getCreatedAt() {
          return createdAt;
      }
  
      public LocalDateTime getUpdatedAt() {
          return updatedAt;
      }
  }
 ```

### 2. extends할 entity
 ```
  @Entity
  public class User extends BaseEntity { // extends로 확장하여 추가
  
      @Column(nullable = false)
      private String name;
  
      protected User() {} // JPA 기본 생성자
  
      public User(String name) {
          this.name = name;
      }
  
      public String getName() {
          return name;
      }
  }
 ```
### 3. 메인어플리케이션
 ```
  @SpringBootApplication
  @EnableJpaAuditing // 추가필요
  public class Application {
  }
 ```

## 4. 기타 포인트
### 1. 생명주기 이벤트 예시소스
 ```
  @MappedSuperclass
  public abstract class BaseTimeEntity {
  
      private LocalDateTime createdAt;
      private LocalDateTime updatedAt;
  
      @PrePersist
      protected void onCreate() {
          this.createdAt = LocalDateTime.now();
          this.updatedAt = this.createdAt;
      }
  
      @PreUpdate
      protected void onUpdate() {
          this.updatedAt = LocalDateTime.now();
      }
  }
  
  // 특정 상황에 save()하면,
  // PrePersist가 달려있는 onCreate() 실행
  // createdAt, updatedAt 세팅하여 업데이트됨.
  // 해당 값들을 넣지 않으면 직접 셋팅이 필요해짐.
 ```

### 2.종류
  
  | 애노테이션          | 실행 시점     |
  | -------------- | --------- |
  | `@PrePersist`  | INSERT 전에 |
  | `@PostPersist` | INSERT 후에 |
  | `@PreUpdate`   | UPDATE 전에 |
  | `@PostUpdate`  | UPDATE 후에 |
  | `@PreRemove`   | DELETE 전에 |
  | `@PostRemove`  | DELETE 후에 |
  
