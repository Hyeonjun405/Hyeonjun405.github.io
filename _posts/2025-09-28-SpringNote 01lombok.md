---
title: 01 Spring lombok
date: 2025-09-28 10:00:00 +09:00
categories: [Spring Framework, SpringNote]
tags: [ Java, Spring Framework, Spring Note ]
---

### 1. lombok 
 - Lombok은 자바용 오픈소스 라이브러리로, 반복되는 보일러플레이트 코드(boilerplate code)를 컴파일 시점에 자동으로 생성해주는 도구
 - note
   - 보일러플레이트 코드 = 개발자가 매번 비슷하게 작성해야 하는 코드들 (getter/setter, toString, equals, hashCode, 생성자 등)
   - Lombok이 어노테이션 프로세서(Annotation Processor)로 동작하면서, 이런 코드를 컴파일 타임에 자동으로 추가해 줍니다.
   - 개발자가 직접 안 써도, 컴파일된 클래스(.class) 안에는 자동으로 해당 메서드가 포함돼 있습니다.

### 2. lombok 작동방식
 - 개발자가 클래스에 Lombok 애노테이션을 붙임 (@Getter, @Setter, @Data 등).
 - 자바 컴파일러(Javac)가 코드를 컴파일할 때, Lombok의 Annotation Processor가 실행됨.
 - 어노테이션에 맞는 메서드(getter/setter 등)를 자동 생성해서 클래스에 삽입.
 - 결과적으로 바이트코드(.class 파일)에는 완성된 메서드가 존재.
 - 런타임이 아니라 컴파일 타임에 코드가 추가되는 것입니다.(☆)

### 2. 애노테이션
#### 1. 필드

| 애노테이션     | 설명                   | 비고                 |
| --------- | -------------------- | ------------------ |
| `@Getter` | 모든 필드의 getter 메서드 생성 | 클래스 단위/필드 단위 적용 가능 |
| `@Setter` | 모든 필드의 setter 메서드 생성 | 클래스 단위/필드 단위 적용 가능 |

#### 2. 생성자

| 애노테이션                      | 설명                                        | 비고                  |
| -------------------------- | ----------------------------------------- | ------------------- |
| `@NoArgsConstructor`       | 파라미터 없는 기본 생성자 생성                         | JPA Entity 등에 자주 사용 |
| `@AllArgsConstructor`      | 모든 필드를 매개변수로 받는 생성자 생성                    | DTO 등에서 편리          |
| `@RequiredArgsConstructor` | `final` 또는 `@NonNull` 필드만 매개변수로 받는 생성자 생성 | 의존성 주입 시 자주 사용      |

#### 3. 객체 메서드

| 애노테이션                | 설명                                 | 비고                                |
| -------------------- | ---------------------------------- | --------------------------------- |
| `@ToString`          | `toString()` 메서드 자동 생성             | 특정 필드 제외 가능 (`@ToString.Exclude`) |
| `@EqualsAndHashCode` | `equals()`와 `hashCode()` 메서드 자동 생성 | 컬렉션 비교, VO 객체 등에 유용               |

#### 4. 클래스 단위

| 애노테이션    | 설명                                                                                     | 비고         |
| -------- | -------------------------------------------------------------------------------------- | ---------- |
| `@Data`  | `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@RequiredArgsConstructor` 종합 | 가장 많이 쓰임   |
| `@Value` | 불변 객체 생성 (`private final` 필드 + getter만 생성)                                             | DTO/VO에 유용 |

#### 5. 빌더

| 애노테이션      | 설명              | 비고                       |
| ---------- | --------------- | ------------------------ |
| `@Builder` | 빌더 패턴 메서드 자동 생성 | 생성자, 클래스, 메서드 레벨에서 사용 가능 |


