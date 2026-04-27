---
title: 05 Inner Class
date: 2026-04-27 10:00:00 +09:00
categories: [Sparta, SpartaNote]
tags: [ Java, Spring Framework ]
---

## 1. Note
- 어떻게 나눠 써야할지가 고민
  - 기존 프로젝트들에서 DTO를 분리해서 다 만들었더니 복잡해짐
  - 그런데 다 안에 넣자니 공통부분인데 반복적임.
  - 어떻게 풀어나가야할까.
- Outer Class에서 지정한 변수로 JSON 구조가 Response
- 일단 어떤 상황에서도 From을 잘써야할듯!!
 
## 2. Response DTO(Inner Class)
### 1. DTO
```
@Getter
public class ProductResponse {

    private Long id;
    private String name;
    private int price;
    private Seller seller;

    
    @Getter
    @AllArgsConstructor
    // 여기에서 Inner 클래스를 안쓰면 외부에 별도 Class 생성해서 DTO가 늘어남
    public static class Seller {
        private Long id;
        private String name;
    }

    public ProductResponse(Product product) {
        this.id = product.getId();
        this.name = product.getName();
        this.price = product.getPrice();
        this.seller = new Seller(
            product.getSeller().getId(),
            product.getSeller().getName()
        );
    }
}
```
### 2. 계층 구조
```
{
  "id": 1,
  "name": "상품",
  "price": 10000,
  "seller": { -- Inner 클래스 부분
    "id": 10,
    "name": "판매자"
  }
}
```

## 3. Inner Class Static
### 1. 사용이유
- 외부 클래스 의존 제거

  ```
  // Non-Static
  Seller seller = productResponse.new Seller();
  // Static
  // Seller는 독립적인 데이터 구조
  // ProductResponse 없이도 생성 가능
  Seller seller = new Seller();
  ```
  
- 메모리 효율(불필요한 outer 참조 제거)
  ```
  // Non-Static → ProductResponse 인스턴스 참조를 내부적으로 들고 있음
  // ProductResponse가 GC 대상이 되어도 Seller가 살아있으면 메모리 해제 안됨
  // Static → 외부 참조 없음 → ProductResponse 없어도 독립적으로 GC 가능
  ```
  
- 직렬화 안정성 (JSON 변환 시 안전)
  ```
  // Non-Static → 직렬화 시 외부 클래스 인스턴스 없으면 에러
  // com.fasterxml.jackson.databind.exc.InvalidDefinitionException 발생 가능

  // Static → 외부 참조 없으니 Jackson이 독립적으로 직렬화 가능
  ```

## 3. Request/Response DTO + Pageable
### 1. Note
- 공통 / 같은규격으로 반복적으로 사용한다면 별도로 클래스로 만들어두는 것도 방법
- 특정 Request만 쓴다면 Inner Class
- 그러나 스프링에서는 기본값적으로 제공은 해서, 상황에 따라 숑

### 2. Request / Response
#### 1. Request
```
@Getter
@NoArgsConstructor
public class ProductSearchRequest {
    private String name;
    private Pageable pageable;

    @Getter
    @NoArgsConstructor
    public static class Pageable {
        private int page = 0; // 기본값 설정 가능
        private int size = 20; // 기본값 설정 가능
        private String sort;
    }
}
```

#### 2. Response
```
// Response
@Getter
@NoArgsConstructor
public class ProductListResponse {
    private List<ProductResponse> contents;
    private Pageable pageable;

    @Getter
    @NoArgsConstructor
    public static class Pageable {
        private int page = 0;
        private int size = 20;
        private int totalPages;
        private long totalElements;
        private boolean isLast;
    }
}
```

### 3. SpringBoot - Controller
```
//개별 적용
public ResponseEntity<?> getProducts(
    @PageableDefault(page = 0, size = 20, sort = "createdAt") Pageable pageable) {
}
```
