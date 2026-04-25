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
#### 1. 옵션 조합과 재고 관리
 - 질문
   - Product Option에 option_type, option_value두고 이것에 대한 재고 stock를 둔 상황입니다.
   - 이때 상품이 상품 옵션을 1개만 선택할 수 있다면 재고가 문제가 없지만,
   - 상품이 두가지 이상의 옵션(빨강+L사이즈)을 동시에 가져야하는 상황이라면,
   - 한가지 옵셥으로 stock을 정의 내릴 수 없는 상황이 발생하였습니다.
   - 이때 두가지 방법이 있을 것 같습니다.
     - 사용가능한 옵션의 수를 컬럼으로 정해서, 사용자에게 최대 옵션의 수를 제공하는 방법
     - Product id를 FK로 둔 옵션들을 모아둔 엔티티를 만드는 방법
   - 그럼 큰 시스템이라면 회사내부 인프라 환경이나, 자바 등등 무엇을 고려해서 어떤 근거로 판단을 해야할까요..?
 - 피드백
   - 현재 `option_type`(COLOR) + `option_value`(빨강) + `stock` 구조면, "빨강"과 "L사이즈"의 재고가 따로 존재하게 되어 "빨강+L사이즈" 조합의 재고를 관리할 곳이 애매해집니다.
   - 가장 단순하고 흔한 방식은, 옵션 속성을 쪼개지 않고 **판매 가능한 조합 자체를 하나의 row로** 표현하는 겁니다.
   - "빨강/L", "빨강/XL", "파랑/L" 이런 식으로요. 이러면 재고 차감 단위가 한 row에 딱 대응되어서 관리가 깔끔합니다.
   - 옵션 그룹이나 Variant 테이블 분리는 조합이 동적으로 대량 생성되어야 하는 규모에서 고려하면 되고, 지금 단계에서는 이 방식이 적합합니다.
   - 기능 추가보다 **테이블과 엔티티가 1:1로 정확히 대응되는 것**이 먼저입니다.
   - DDL 컬럼과 엔티티 필드를 나란히 놓고 비교하는 습관을 들이면 런타임 에러를 많이 줄일 수 있습니다.

## 4. 1주차 피드백
프로젝트 고생하셨습니다. 도메인별 패키지 분리, 공통 엔티티, Flyway 스키마 관리 흐름이 잘 잡혀 있고, `cascade` + `orphanRemoval` 설정도 방향이 맞습니다. 아직 미완성이라고 하셨는데 기본 뼈대는 잘 나와 있어서, 엔티티와 DDL 사이에 안 맞는 부분만 맞춰 주시면 완성도가 많이 올라갈 것 같습니다.

체크리스트 기반 평가
- [products 테이블 설계]
  기본 컬럼은 잘 들어가 있습니다. 다만 엔티티의 `status` 필드가 DDL에 빠져 있어서, `ddl-auto: none` 환경에서는 저장 시 오류가 날 수 있습니다. `price`도 DDL은 `INTEGER`이고 엔티티는 `BigDecimal`이라 타입을 맞춰 주시면 좋겠습니다.
- [Product 엔티티 캡슐화]
  setter를 안 열고 `@Getter`만 쓴 건 좋습니다. 여기에 비즈니스 메서드까지 추가하면 더 좋을 것 같고, `@NoArgsConstructor`도 `access = AccessLevel.PROTECTED`로 제한해 주시면 외부에서 빈 객체를 만드는 걸 막을 수 있습니다.
- [시간 관리]
  `BaesEntity`에 `@CreationTimestamp`, `@UpdateTimestamp` 잘 구성하셨습니다.
- [categories 테이블 및 자기 참조 구조]
  `parent_id` + `@ManyToOne(LAZY)` 매핑 잘 되어 있습니다. 여기에 자식 컬렉션(`children`)을 추가하면 부모→자식 방향 탐색도 가능해지고, DDL에 `parent_id` FK 제약도 같이 걸어 주시면 좋겠습니다.
- [양방향 연관관계 편의 메서드]
  아직 편의 메서드가 없는 상태입니다. `addChild()`, `addOption()` 같은 메서드로 양쪽 상태를 동기화하는 부분을 추가해 주시면 좋겠습니다.
- [상태 Enum 문자열 저장]
  `@Enumerated(EnumType.STRING)` 설정이 잘 되어 있습니다.
- [product_options 테이블 무결성 및 삭제 정책]
  FK 제약, CHECK 제약, `ON DELETE CASCADE`가 빠져 있어서 DB 차원의 무결성이 부족한 상태입니다. 이 부분 보강해 주시면 좋겠습니다.
- [옵션 생명주기 관리]
  `cascade = ALL` + `orphanRemoval = true` 잘 설정하셨습니다. DDL 쪽에도 `ON DELETE CASCADE`를 넣어서 JPA 설정과 방향을 맞춰 주시면 더 좋을 것 같습니다.

개선 포인트
- [엔티티-DDL 정합성]
  이 부분이 가장 먼저 챙겨야 할 곳입니다. `Product.status`는 엔티티에만 있고 DDL에 컬럼이 없고, `Product.price`는 엔티티가 `BigDecimal`인데 DDL은 `INTEGER`라 타입이 안 맞고, `ProductOption.stock`도 엔티티가 `BigDecimal`인데 재고는 정수니까 `Integer`가 더 자연스럽습니다. 엔티티 필드와 DDL 컬럼을 하나씩 대조해 보시면 금방 정리될 거예요.
- [Flyway 마이그레이션 순서]
  V1에서 `products`를 만들면서 `category_id NOT NULL`을 넣고 있는데, `categories`는 V2에서 생성됩니다. FK를 추가하면 에러가 나고, 안 걸어도 논리적으로 순서가 안 맞으니 categories를 먼저 만들도록 순서를 조정해 주시면 좋겠습니다.
- [DDL에 FK + CHECK 제약 추가]
  세 테이블 모두 FK가 없는 상태입니다. `categories.parent_id`, `products.category_id`, `product_options.product_id`에 FK를 걸어 주시고, 가격/재고 컬럼에 `CHECK (>= 0)` 제약도 넣어 주시면 좋겠습니다. 참고로 `@Min(0)`은 Jakarta Validation을 명시적으로 트리거해야 동작하기 때문에, DB CHECK이 더 확실한 안전망이 됩니다.
- [카테고리 children 컬렉션 + 편의 메서드]
  `Category`에 `@OneToMany(mappedBy = "parent")` children 컬렉션을 두고, `addChild()` 편의 메서드로 양쪽 상태를 동기화해 주시면 좋겠습니다. 상품-옵션도 마찬가지로 `addOption()` 같은 메서드가 있으면 좋습니다.
- [엔티티 비즈니스 메서드]
  지금은 매핑만 있고 상태 변경 메서드가 없는데, 생성자에서 불변식 검증(`name` 필수, `price >= 0`)을 하고 `changePrice()` 같은 메서드를 추가해 주시면 엔티티가 좀 더 규칙을 가진 객체로 바뀝니다.
- [`@NoArgsConstructor` 접근 수준]
  JPA 기본 생성자는 `protected`면 충분하니, `@NoArgsConstructor(access = AccessLevel.PROTECTED)`로 바꿔 주시면 됩니다.

질문에 대한 답변 — 옵션 조합과 재고 관리
현재 `option_type`(COLOR) + `option_value`(빨강) + `stock` 구조면, "빨강"과 "L사이즈"의 재고가 따로 존재하게 되어 "빨강+L사이즈" 조합의 재고를 관리할 곳이 애매해집니다.

가장 단순하고 흔한 방식은, 옵션 속성을 쪼개지 않고 **판매 가능한 조합 자체를 하나의 row로** 표현하는 겁니다. "빨강/L", "빨강/XL", "파랑/L" 이런 식으로요. 이러면 재고 차감 단위가 한 row에 딱 대응되어서 관리가 깔끔합니다. 옵션 그룹이나 Variant 테이블 분리는 조합이 동적으로 대량 생성되어야 하는 규모에서 고려하면 되고, 지금 단계에서는 이 방식이 적합합니다.

- 기능 추가보다 **테이블과 엔티티가 1:1로 정확히 대응되는 것**이 먼저입니다. DDL 컬럼과 엔티티 필드를 나란히 놓고 비교하는 습관을 들이면 런타임 에러를 많이 줄일 수 있습니다.

수고하셨습니다!!
