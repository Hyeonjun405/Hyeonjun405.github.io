---
title: 06 Optional
date: 2025-10-03 10:00:00 +09:00
categories: [00Spring Framework, SpringNote]
tags: [ Java, Spring Framework, Spring Note ]
---

## 1. Optional
### 1. Optional
 - 자바 8에서 도입된 클래스(java.util.Optional)
 - 값이 있을 수도 있고 없을 수도 있는 경우를 안전하게 처리하기 위한 래퍼(wrapper) 객체
 - NullPointerException(NPE)을 예방하기 위해 설계됨
 - Null 일경우 비어있는 Optional이 됨. 

### 2. tryCatch와 Optional

| 구분             | try-catch 방식                                                                              | Optional 방식                                                                                |
  | -------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 목적         | NullPointerException 발생 후 처리                                                              | 값이 없을 수 있는 상황을 설계적으로 처리                                                                    |
| Null 체크 방식 | 예외 발생 시 catch 블록에서 처리                                                                     | Optional을 사용해 안전하게 값 여부 확인                                                                 |
| 코드 가독성     | null 체크 로직이 흩어지고 예외 처리 코드가 복잡                                                             | null 여부를 명시적으로 표현 → 깔끔하고 읽기 쉬움                                                             |
| 예외 처리 시점   | 런타임 시점에 NPE 발생 후 처리                                                                       | 컴파일 시점에 null 가능성 인지 및 처리                                                                   |
| 성능         | 예외 발생 시 오버헤드가 크고 비용이 높음                                                                   | null 체크 자체를 설계적으로 처리 → 비교적 효율적                                                             |
| Null 안전성   | 예외가 발생할 수 있어 완전하지 않음                                                                      | Optional 자체가 null 안전성을 보장                                                                  |
| 코드 유지보수    | try-catch 블록이 많아지면 복잡성 증가                                                                 | Optional API로 통일된 null 처리 로직 가능                                                            |
| 사용 권장 상황   | 예외가 예상되는 비정상 상황                                                                           | 값이 없을 수 있는 정상적인 상황                                                                         |
| 예시 코드      | `try { String name = getName().toUpperCase(); } catch (NullPointerException e) { // 처리 }` | `Optional.ofNullable(getName()).map(String::toUpperCase).ifPresent(System.out::println);` |

### 3. 사용패턴
  ```
  Optional<String> optName = Optional.ofNullable(getName()); // null이라면 empty생성처리 
  
  String greeting = optName
      .map(name -> "Hello " + name) // empty가 아니라면 hello "name" 반환
      .orElse("Hello Guest"); // empty라면 Hello Guest 반환
  
  System.out.println(greeting); 
  ```

#### 1. 생성

  | 메서드                     | 설명                                         | 예시                          |
  | ----------------------- | ------------------------------------------ | --------------------------- |
  | empty()             | 값이 없는 Optional 생성                          | `Optional.empty()`          |
  | of(T value)         | null이 아닌 값을 감싸 Optional 생성 (null이면 NPE 발생) | `Optional.of("Hello")`      |
  | ofNullable(T value) | 값이 null일 수 있는 경우 Optional 생성               | `Optional.ofNullable(name)` |

#### 2. 꺼내기

  | 메서드                                                      | 설명                      | 예시                                              |
  | -------------------------------------------------------- | ----------------------- | ----------------------------------------------- |
  | get()                                                | 값 꺼내기 (값이 없으면 NPE)      | `opt.get()`                                     |
  | isPresent()                                          | 값 존재 여부 확인              | `opt.isPresent()`                               |
  | orElse(T other)                                      | 값이 없으면 기본값 반환           | `opt.orElse("Default")`                         |
  | orElseGet(Supplier<? extends T> supplier)            | 값이 없으면 Supplier로 기본값 반환 | `opt.orElseGet(() -> "Default")`                |
  | orElseThrow(Supplier<? extends X> exceptionSupplier) | 값이 없으면 예외 발생            | `opt.orElseThrow(() -> new RuntimeException())` |

#### 3. 값처리

  | 메서드                                                  | 설명                                       | 예시                                                |
  | ---------------------------------------------------- | ---------------------------------------- | ------------------------------------------------- |
  | ifPresent(Consumer<? super T> consumer)          | 값이 있으면 처리                                | `opt.ifPresent(name -> System.out.println(name))` |
  | map(Function<? super T, ? extends U> mapper)     | 값이 있으면 변환, 없으면 Optional.empty            | `opt.map(String::toUpperCase)`                    |
  | flatMap(Function<? super T, Optional<U>> mapper) | map과 유사하지만 Optional을 반환하는 함수 처리          | `opt.flatMap(this::findName)`                     |
  | filter(Predicate<? super T> predicate)           | 조건을 만족하면 Optional 유지, 아니면 Optional.empty | `opt.filter(name -> name.length() > 3)`           |


## 2. 스프링에서의 옵셔널
### 1. JPA
#### 1. Repository
  ```
  public interface UserRepository extends JpaRepository<User, Long> {}
  // JPA는 별도로 옵셔널 처리를 하지 않음.
  // JpaRepository<T, ID> 인터페이스에 findById(ID id) 메서드가 이렇게 정의가 " Optional<T> findById(ID id); "
  ```

#### 2. Service
  ```
    ~~~~~
    public String getUserGreeting(Long id) {
        return userRepository.findById(id) // Optional<User> 반환
                .map(user -> "Hello " + user.getName()) // 값이 있으면 변환
                .orElse("Hello Guest"); // 값이 없으면 기본값
    }
  ```

### 2. Optional 직접 반환
#### 1. 메소드 
  ```
  public Optional<User> findUserByEmail(String email) {
    return userRepository.findAll().stream()
        .filter(user -> user.getEmail().equals(email))
        .findFirst(); // Optional<User> 반환
  }
  ```

#### 2. Service
  ```
  public String getUserEmailGreeting(String email) {
      return findUserByEmail(email)
              .map(user -> "Welcome back, " + user.getName())
              .orElse("Welcome, Guest");
  }
  ```
### 3. 기타
#### 1. ifPresent()
  ```
  userRepository.findById(1L)
  .ifPresent(user -> System.out.println("User exists: " + user.getName()));
  // 값이 있는 경우 User exists : get name
  // 없으면 통과
  ```

#### 2. orElseThrow()
  ```
  User user = userRepository.findById(1L)
          .orElseThrow(() -> new RuntimeException("User not found"));
  ```
