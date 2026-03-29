---
title: 05 QueryDsl
date: 2026-03-22 10:00:00 +09:00
categories: [Sparta, SpartaWIL02]
tags: [ Java, Spring Framework, JPA ]
---

## 1. Note
 - JPA에서 자동화 방식으로 사용하다가,
 - 조건이나 뭔가 복잡한 상황에서 개발자가 직접 사용하는 패턴.
 - 뭔가 구분이 잘 안되기는 하지만,
   - 레파지토리는 DB접근, 쿼리수행, 엔티티 저장 및 조회 등의 역할
   - 엔티티는 상태랑 비즈니스규칙 관련, 값 변경 할 경우 사용하는 역할
 

## 2. Spring Data JPA / JPQL / Query DSL
### 1. Spring Data JPA / JPQL / Query DSL
  
  | 방식              | 작성 위치               | 장점                        | 단점                  | 동적 조건 처리 |
  | --------------- | ------------------- | ------------------------- | ------------------- | -------- |
  | 메서드 이름 기반       | JpaRepository 인터페이스 | 코드 간단, 별도 쿼리 작성 불필요       | 조건이 복잡하면 메서드 이름 길어짐 | 어려움      |
  | JPQL (`@Query`) | JpaRepository 인터페이스 | 엔티티 기준, SQL과 유사, 직관적      | 문자열 기반, 동적 조건 복잡    | 제한적      |
  | QueryDSL        | 커스텀 레포지토리 구현체       | 타입 세이프, 동적 조건 자유, 유지보수 용이 | 설정 필요, 구현 복잡        | 매우 용이    |


### 2. 메서드 이름 기반 쿼리 (Spring Data JPA)
- JpaRepository에서 제공하는 기본 기능을 이용해, 메서드 이름으로 쿼리를 자동 생성합니다.
  - 장점 : 간단하고 코드가 짧음, 별도의 쿼리 작성 불필요
  - 단점 : 복잡한 조건이나 동적 쿼리는 어렵습니다.
  ```
  public interface ProductRepository extends JpaRepository<Product, Long> {
     List<Product> findByName(String name);
     List<Product> findByCategoryIdAndPriceLessThan(Long categoryId, BigDecimal price);
  }
  ```
  
### 3. JPQL 사용 (Query annotation)
 - @Query 애노테이션으로 JPQL을 직접 작성해서 조회합니다.
   - 장점 : SQL과 유사하지만 엔티티 기준으로 작성, 단순 조회는 명확하고 직관적
   - 단점 : 문자열 기반이므로 동적 조건 처리 시 복잡해짐
  ```
   public interface ProductRepository extends JpaRepository<Product, Long> {
    @Query("SELECT p FROM Product p WHERE p.name = :name AND p.category.id = :categoryId")
    List<Product> findByNameAndCategory(@Param("name") String name, @Param("categoryId") Long categoryId);
   }
  ```

### 4. QueryDSL 사용 (타입 세이프, 동적 쿼리)
 - QueryDSL 라이브러리를 이용해 엔티티 기준으로 동적 쿼리를 작성합니다.
   - 장점: 조건 조합 자유롭고, 컴파일 시점에서 타입 체크 가능, 유지보수 용이
   - 단점: 설정과 구현체가 필요, 조금 더 복잡
  ```
   public class ProductRepositoryImpl implements ProductRepositoryCustom {
        @PersistenceContext
        private EntityManager em;
    
        QProduct product = QProduct.product;
    
        @Override
        public List<Product> search(String name, Long categoryId, BigDecimal maxPrice) {
            JPAQuery<Product> query = new JPAQuery<>(em);
            return query.select(product)
                        .from(product)
                        .where(
                            name != null ? product.name.eq(name) : null,
                            categoryId != null ? product.category.id.eq(categoryId) : null,
                            maxPrice != null ? product.price.loe(maxPrice) : null
                        )
                        .fetch();
        }
    }
  ```

## 3. Query DSL
### 1. 의존성 추가 필요
 ```
 // build.gradle
 dependencies {
    // QueryDSL JPA 라이브러리
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'

    // QueryDSL 어노테이션 프로세서 (Q클래스 생성)
    annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"
 }
 ```

### 2. Q 클래스 생성을 위한 gradle 설정
 ```
  //그래들 최하단에 등록
  def querydslDir = layout.buildDirectory.dir("generated/querydsl").get().asFile

  tasks.withType(JavaCompile).configureEach {
      options.getGeneratedSourceOutputDirectory().set(file(querydslDir))
  }
  
  clean {
      delete querydslDir
  }
 ```

### 3. JPQL 빈등록
 ```
  @Configuration
  public class QueryDslConfig {
  
      @PersistenceContext
      private EntityManager entityManager;
  
      @Bean
      public JPAQueryFactory jpaQueryFactory() {
          return new JPAQueryFactory(entityManager);
      }
  }
 ```

### 4. 사용 패턴
#### 1. 기본형태
   ```
    @Repository
    @RequiredArgsConstructor
    public class ProductQueryRepository {
    
      private final JPAQueryFactory queryFactory;
    
      public List<Product> findProducts(String name, Double minPrice, Double maxPrice) {
      
        QProduct product = QProduct.product; //선언해도 가능하지만 필드로 해도됨.
        
        return queryFactory
            .selectFrom(product) // SELECT * FROM product
            .where(
                // 각 조건이 null이면 where절이 생성 되지 않음.
                // 쉼표로 나누는 경우 and로 연결
                name != null ? product.name.contains(name) : null,
                minPrice != null ? product.price.goe(minPrice) : null, // goe: >=
                maxPrice != null ? product.price.loe(maxPrice) : null  // loe: <=
                // or 연결
                name != null ? product.name.contains(name).or(product.description.contains(name)) : null
            )
            .fetch();
      }
    }
   ```
   
#### 2. where절의 메소드화
  ```
   public List<Product> findProducts(String name, Double minPrice, Double maxPrice) {
      return queryFactory
              .selectFrom(product)
              .where(
                  nameContains(name), // 메서드 호출
                  priceGoe(minPrice),
                  priceLoe(maxPrice)
              )
              .fetch();
  } 
  
  private BooleanExpression nameContains(String name) {
      return name != null ? product.name.contains(name) : null;
  }
  ```
#### 3. join
 ```
 public List<Product> findProducts(String name, Double minPrice, Double maxPrice) {
    // Q클래스 인스턴스 선언 필수
    QProduct product = QProduct.product;
    QCategory category = QCategory.category;

    return queryFactory
        .selectFrom(product)
        .join(product.category, category) //left, right 다먹음! 
        .where(
            name != null ? product.name.contains(name) : null,
            minPrice != null ? product.price.goe(minPrice) : null,
            maxPrice != null ? product.price.loe(maxPrice) : null,
            category.name.eq("도서")
        )
        .fetch();
 }
 ```

#### 4. Paging(SpartaNote_01에 별도 내용 심화)
  - 소스
   ```
   List<Product> products = queryFactory
      .selectFrom(product)
      .orderBy(product.price.desc()) // 정렬
      .offset(page * size)           // 시작 위치
      .limit(size)                   // 가져올 데이터 수
      .fetch();                      // 실행
   ```
 - 구분

   | page | size | offset | 데이터 범위    |
   | ---- | ---- | ------ | --------- |
   | 0    | 50   | 0      | 1 ~ 50    |
   | 1    | 50   | 50     | 51 ~ 100  |
   | 2    | 50   | 100    | 101 ~ 150 |
   | 3    | 50   | 150    | 151 ~ 200 |
  
## 4. QueryDSL에서 DTO 조회
### 1. Constructor / fields / bean / queryProjection 

| 방식              | 매핑 기준       | 컴파일 안전성 | 런타임 에러 위험 | 성능        | 코드 간결성 | QueryDSL 의존성 |
| --------------- | ----------- | ------- | --------- | --------- | ------ | ------------ |
| constructor     | 생성자 파라미터 순서 | ❌       | 높음        | 좋음        | 좋음     | 없음           |
| fields          | 필드명         | ❌       | 중간        | 보통 (리플렉션) | 좋음     | 없음           |
| bean            | setter      | ❌       | 중간        | 보통 (리플렉션) | 보통     | 없음           |
| QueryProjection | 생성자(Q타입)    | ✔       | 낮음        | 좋음        | 보통     | 있음           |

### 2. 각기 패턴
#### 1. Constructor
 ```
  .select(Projections.constructor(ProductDto.class,
      product.name,
      product.price
  ))
   // 생성자 기반
   // 순서 맞아야 함 (중요)
   // 컴파일 체크 X (런타임 에러)  
 ```


#### 2. fields 방식
 ```
  .select(Projections.fields(ProductDto.class,
      product.name,
      product.price
  ))
   // 필드명 기준 매핑
   // 리플렉션 사용
   // private 필드도 접근 가능
 ```

#### 3. bean
 ```
  .select(Projections.bean(ProductDto.class,
    product.name,
    product.price
  ))
  // setter 기반
  // JavaBean 규칙 필요 (get/set) 
 ```

#### 4. @QueryProjection 방식
 ```
  .select(new QProductDto(
    product.name,
    product.price
  ))
  // 컴파일 시점 체크
  // 타입 안전성 최고
  // 대신 QueryDSL 의존성 생김  
 ```


## 5. QueryProjection 
### 1. 프로젝션
 - 필요한 컬럼들만 선별하여 처음부터 DTO(데이터 전송 객체)에 담아 조회하는 기술
 - QueryDSL에서 DTO를 타입 안전하게 조회하기 위해 사용하는 기능
 
### 2. 사용방법
#### 1. 사용할 DTO에 애노테이션 추가 
 ```
  public class ProductDto {
  
      private String name;
      private int price;

      // DTO 생성자 애노테이션 추가
      @QueryProjection
      public ProductDto(String name, int price) {
          this.name = name;
          this.price = price;
      }
  }
 ``` 

#### 2. QueryDsl impl
 ```
 // Query DSL 사용하는 impl
 return queryFactory
    .select(new QProductDto(
        product.name,
        product.price
    ))
    .from(product)
    .fetch();
 ```




