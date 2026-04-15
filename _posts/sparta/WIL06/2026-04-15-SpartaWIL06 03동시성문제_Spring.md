---
title: 03 동시성 문제 & Spring
date: 2026-04-15 10:00:00 +09:00
categories: [Sparta, SpartaWIL06]
tags: [ Java, Spring Framework ]
---

## 1. Note
- 동시에 접근할때 어떤 트랜잭션을 쓰고 관리해야하는가?
- 생각을 생각보다 많이 해야한다..!

## 2. Spring Transactional 격리 레벨 설정
### 1. 문제

| 구분                  | 무엇이 바뀌나 | 문제           |
| ------------------- | ------- | ------------ |
| Dirty Read          | 커밋 안된 값 | 존재하지 않는 데이터  |
| Non-repeatable Read | 값 변경    | 같은 조회 결과 달라짐 |
| Phantom Read        | 행 개수 변경 | 없던 데이터 등장    |


### 2. 트랜잭션 격리레벨 설정
```
@Transactional(isolation = Isolation.READ_COMMITTED)
public void getOrder() {
    // 다른 트랜잭션이 commit한 데이터만 읽음
}

@Transactional(isolation = Isolation.REPEATABLE_READ)
public void processPayment() {
    // 같은 데이터를 여러 번 읽어도 값이 바뀌지 않음
}

@Transactional(isolation = Isolation.SERIALIZABLE)
public void decreaseStock() {
    // 동시에 실행되면 순차적으로 처리됨
}
```

### 3. 언어별 비교
- MYSQL

  | 격리 수준           | 동작 방식                | 락 사용  | 특징               |
  | --------------- | -------------------- | ----- | ---------------- |
  | READ_COMMITTED  | MVCC                 | 거의 없음 | 커밋된 데이터만 읽음      |
  | REPEATABLE_READ | MVCC + Gap Lock      | 일부 사용 | Phantom 상당 부분 방지 |
  | SERIALIZABLE    | 락 기반 (Next-Key Lock) | 매우 많음 | SELECT에도 락 발생    |

- PostgreSQL
  
  | 격리 수준           | 동작 방식           | 락 사용  | 특징                      |
  | --------------- | --------------- | ----- | ----------------------- |
  | READ_COMMITTED  | MVCC            | 없음    | 기본 격리 수준                |
  | REPEATABLE_READ | MVCC (Snapshot) | 없음    | 트랜잭션 동안 동일 데이터 보장       |
  | SERIALIZABLE    | SSI (충돌 감지)     | 거의 없음 | 충돌 시 트랜잭션 실패 (retry 필요) |

- Oracle

  | 격리 수준           | 동작 방식            | 락 사용  | 특징                   |
  | --------------- | ---------------- | ----- | -------------------- |
  | READ_COMMITTED  | MVCC             | 없음    | 기본 격리 수준             |
  | REPEATABLE_READ | 지원 안함            | -     | READ_COMMITTED처럼 동작  |
  | SERIALIZABLE    | Snapshot + 충돌 감지 | 거의 없음 | 충돌 시 ORA-08177 에러 발생 |


## 2. Dirty Read
### 1. 케이스
 - A가 데이터를 롤백 하고 Sleep
 - B가 A가 슬립한동안 데이터를 조회함.
 - A의 슬립이 끝나고 커밋
 - 최종값 확인

### 2. 조회 메소드 구분
#### 1. 서비스
```
private final EntityManager entityManager;
private final ProductRepository productRepository;
private final CategoryRepository categoryRepository;

@Transactional
public void updateStockAndForceRollback(Long productId, int newStock) {
  
  // 1. 상품을 조회하고 재고를 변경
  Product product = productRepository.findById(productId).orElseThrow();
  log.info("Thread A: 재고를 " + product.getStock() + "에서 " + newStock + "으로 변경 시도");
  product.setStock(newStock);

  // 2. DB 세션에 변경사항을 즉시 반영(flush)
  // COMMIT은 아니지만, DB에 UPDATE 쿼리를 보내 변경사항이 적용
  productRepository.saveAndFlush(product);

  // 3. 다른 트랜잭션이 이 'COMMIT되지 않은' 데이터를 읽을 시간을 줌
  try {
    Thread.sleep(5000); // 5초 대기
  } catch (InterruptedException e) {
    Thread.currentThread().interrupt();
  }

  // 4. 의도적으로 트랜잭션을 롤백
  log.info("Thread A: 작업을 롤백합니다.");
  TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
}
```

#### 2. 구분 메소드
```
// Dirty Read를 허용하는 메서드
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public int getStockWithDirtyRead(Long productId) {
    System.out.println("Thread B: READ_UNCOMMITTED 트랜잭션에서 재고 조회 시도");
    Product product = productRepository.findById(productId).orElseThrow();
    return product.getStock();
}

// Dirty Read를 방지하는 메서드
@Transactional(isolation = Isolation.READ_COMMITTED)
public int getStockWithReadCommitted(Long productId) {
    System.out.println("Thread B: READ_COMMITTED 트랜잭션에서 재고 조회 시도");
    Product product = productRepository.findById(productId).orElseThrow();
    return product.getStock();
}
```

### 3. 현상 구현
```
@SpringBootTest
class ProductServiceTest {

    @Autowired private ProductService productService;
    @Autowired private ProductRepository productRepository;

    @Test
    @DisplayName("READ_UNCOMMITTED에서는 Dirty Read가 발생한다")
    void testDirtyReadAllowed() throws InterruptedException {
        // Given: 초기 재고는 20
        Long productId = 1L;

        // Thread A: 재고를 10으로 바꾸고 롤백
        Thread threadA = new Thread(() -> {
            productService.updateStockAndForceRollback(productId, 10);
        });

        // Thread B: Thread A가 작업하는 도중에 재고 조회
        Thread threadB = new Thread(() -> {
            try {
                Thread.sleep(1000);
                 
                // 이 타이밍은 아직 10개로 바꾸고 커밋되기전 상황이 됨
                
                // case1) Isolation.READ_UNCOMMITTED               
                int stock = productService.getStockWithDirtyRead(productId);
                System.out.println(">>>> Dirty Read 발생: 읽은 재고 = " + stock);
                Assertions.assertEquals(10, stock); // 10 => 스레드A의 커밋 전 데이터
                
                // case2) Isolation.READ_COMMITTED
                int stock = productService.getStockWithReadCommitted(productId);
                System.out.println(">>>> Read Committed: 읽은 재고 = " + stock);
                Assertions.assertEquals(20, stock); // 20 => 커밋전 데이터               
               
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        // When
        threadA.start();
        threadB.start();
        threadA.join();
        threadB.join();

        // Then: Thread A가 롤백되었으므로 최종 재고는 원상 복구되어야 함
        Product finalProduct = productRepository.findById(productId).orElseThrow();
        System.out.println(">>>> 최종 실제 재고 = " + finalProduct.getStock());
        Assertions.assertEquals(20, finalProduct.getStock());
    }
} 
```

## 3. Non-Repeatable Read
### 1. 케이스
 - A가 1차 읽기를 수행함
 - B가 커밋을 진행
 - A가 2차읽기를 진행

### 2. 조회 메소드 구분
```
// 격리 수준만 차이 발생

// 커밋된 데이터만 읽음 
//@Transactional(isolation = Isolation.READ_COMMITTED)

// 첫번째 조회 데이터를 유지하려고함 - REPEATABLE_READ
// @Transactional(isolation = Isolation.REPEATABLE_READ)
public void demonstrateNonRepeatableRead(Long productId) {

     // 1. 첫 번째 데이터 조회
    Product product1 = productRepository.findById(productId).orElseThrow();
    System.out.println("Thread A - First Read: Stock = " + product1.getStock());
  
    // 2. 다른 트랜잭션이 데이터를 변경할 시간을 줌
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
  
    // 3. 동일 트랜잭션 내에서 데이터 다시 조회
    Product product2 = productRepository.findById(productId).orElseThrow();
    System.out.println("Thread A - Second Read: Stock = " + product2.getStock());
  
    // 4. 두 조회 결과가 다른지 확인
    if (product1.getStock() != product2.getStock()) {
        System.out.println(">>>> Non-Repeatable Read가 발생했습니다!");
    }
}

// 즉시 커밋    
@Transactional
public void updateStock(Long productId, int newStock) {
  Product product = productRepository.findById(productId).orElseThrow();
  System.out.println("Thread B: 재고를 " + newStock + "으로 변경하고 커밋합니다.");
  product.setStock(newStock);
  // 메서드가 종료되면서 변경 사항이 COMMIT됨
}
```
### 3. 현상구현
```
@Test
@DisplayName("READ_COMMITTED에서는 Non-Repeatable Read가 발생한다")
void testNonRepeatableReadAllowed() throws InterruptedException {
    // Given: 초기 재고는 20
    Long productId = 1L;

    // Thread A: 데이터를 두 번 읽는 긴 트랜잭션
    Thread threadA = new Thread(() -> {
        productService.demonstrateNonRepeatableRead(productId);
    });

    // Thread B: 중간에 데이터를 수정하는 짧은 트랜잭션
    Thread threadB = new Thread(() -> {
        try {
            // Thread A가 첫 번째 읽기를 수행
            Thread.sleep(1000); 
            // Thread B에서 업데이트처리
            productService.updateStock(productId, 5);
            // Trhead A가 두번쨰 읽기 수행 
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    });

    // When
    threadA.start();
    threadB.start();
    threadA.join();
    threadB.join();

    // Then: 최종 재고는 5가 되어야 함
    Product finalProduct = productRepository.findById(productId).orElseThrow();
    Assertions.assertEquals(5, finalProduct.getStock());
}
```

## 4. Phantom Read
### 1. 케이스
- 스레드 A가 수량 조회
- 스레드 B에서 데이터 추가
- 스레드 C에서 수량이 어떤지 확인

### 2. 조회 메소드 구분
```
// 격리 수준만 차이 발생

// MYSQL이 MVCC 떄문에 자동으로 스냅샷을 찍어서 READ_COMMITTED로 비교
@Transactional(isolation = Isolation.READ_COMMITTED)

// 순차적으로 읽음
@Transactional(isolation = Isolation.SERIALIZABLE)
public void demonstratePhantomRead() {
    
    // 1. 첫 번째 범위 조회
    List<Product> products1 = productRepository.findAllByStockGreaterThan(5);
    System.out.println("Thread A - First Read: " + products1.size() + " products found.");

    // 2. 다른 트랜잭션이 데이터를 INSERT할 시간을 줌
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }

    // 3. 동일 트랜잭션 내에서 다시 범위 조회
    List<Product> products2 = productRepository.findAllByStockGreaterThan(5);
    System.out.println("Thread A - Second Read: " + products2.size() + " products found.");

    if (products1.size() != products2.size()) {
        System.out.println(">>>> Phantom Read가 발생했습니다!");
    }
}


@Transactional
public void insertNewProduct(String name, int stock) {
    System.out.println("Thread B: 재고(" + stock + ")를 가진 신상품 추가 시도");
    Product product = new Product(name, stock);
    productRepository.save(product);
    System.out.println("Thread B: 신상품 추가 및 커밋 완료");
}
```

### 3. 현상구현
```
@Test
@DisplayName("REPEATABLE_READ에서는 Phantom Read가 발생할 수 있다")
void testPhantomReadOccurs() throws InterruptedException {
    // Thread A: 특정 범위의 데이터를 두 번 읽는 긴 트랜잭션
    Thread threadA = new Thread(() -> {
        productService.demonstratePhantomRead();
    });

    // Thread B: 중간에 그 범위에 해당하는 데이터를 추가하는 트랜잭션
    Thread threadB = new Thread(() -> {
        try {
            Thread.sleep(1000);
            // 이 메서드는 Thread A가 끝날 때까지 대기(block) 상태에 빠짐
            productService.insertNewProduct("유령신상품", 20);
        } catch (Exception e) {
            // DB에 따라 LockWaitTimeoutException 등이 발생할 수 있음
            System.err.println("Thread B: 헉! 락 때문에 작업을 못했어요! " + e.getMessage());
        }
    });

    // When
    threadA.start();
    threadB.start();
    threadA.join();
    threadB.join();
}
```
