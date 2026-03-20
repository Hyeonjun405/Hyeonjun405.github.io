---
title: 02 Builder_MapStruct
date: 2026-03-20 10:00:00 +09:00
categories: [Sparta, SpartaWIL02]
tags: [ Java, Spring Framework]
---

## 1. Builder
### 1. Note 
 - 생성자를 만들때 오버로딩을 해야하는데, 이를 보완한 방법
 - 생각보다 많이 깔끔한 패턴으로, 필요에 따라서 조율할 수 있어서 좋음!
 - 그런데 빌더 자체를 자주 사용할 상황이 있을까 하는 의문도 들기도 함.

### 2. 체인패턴
 ```
 // 계속 자기 자신을 리턴해서 체인형태로 사용이 가능해짐
 builder.name("A")   // 설정하고 자기 자신 반환
        .age(10)     // 같은 객체에서 계속 설정
        .build();    // 마지막 결과 생성
 ```

### 3. 소스 패턴
#### 1. 서비스단
 ```
  // 체인 패턴으로 필요한 파라미터 등록하여 사용함.
  User user = User.builder()
    .name("홍길동") 
    .age(20)
    .build();
 ```

#### 2. 애노테이션 패턴
 ```
  // Lombok 있을 경우
  @Builder
  @Getter
  public class User {
      private String name;
      private int age;
  }
 ```

#### 3. 롬복이 없는 경우
 ```
   public class User {
  
      private String name;
      private int age;
  
      private User(Builder builder) {
          this.name = builder.name;
          this.age = builder.age;
      }
      // 이너클래스
      public static class Builder {
          private String name;
          private int age;

          public Builder name(String name) {
              this.name = name;
              return this;
          }
  
          public Builder age(int age) {
              this.age = age;
              return this;
          }
  
          public User build() {
              return new User(this);
          }
      }
  }
  ```

## 2. MapStruct
### 1. Note
 - DTO 또는 별도 컴포넌트를 만들어서 변환하던 작업을 해결해주는 방법
 - 소스가 지저분해지던 부분들을 막음
 - 이걸 알았으면 전환할떄 사용했을텐데...

### 2. MapStruct
 - 객체와 객체 변환 코드를 “컴파일 시점에 자동 생성해주는 매핑 라이브러리”
 - 파라미터 값을 리턴타입으로 전환함.
 - 구현체는 MapStruct가 자동 생성
 - 주요 사용처
   ```
    Controller → DTO → Entity
    Entity → DTO → Response
   ```

### 3. 소스 패턴
#### 1. Mapper 선언
 ```
  // 스프링만의 Lib가 아니기 때문에 Model을 Spring으로 표기함
  @Mapper(componentModel = "spring") // Spring Bean으로 등록
  public interface UserMapper {
    
    // User Entity -> UserResponse DTO 변환
    UserResponse toResponse(User user);
  
    // UserRequest DTO -> User Entity 변환
    // 이 상황에서는 리턴타입의 password는 encodedPassword 파라미터에서 가져오게됨.
    // 안쓰는건 증발.
    @Mapping(target = "password", source = "encodedPassword")
    User toEntity(UserRequest request, String encodedPassword);
    
    // Source가 파라미터, target이 리턴타입
    // 컬럼명이 다를 경우 자동으로 변경처리 진행
    // 여러 컬럼 매핑이 필요한 경우 쌓아서 사용하는 것이 현명함.
    @Mapping(source = "userName", target = "name")
    User toEntity(UserRequest request);
  }
 ```

#### 2. 소스단에서 사용
 ``` 
  private final UserMapper userMapper; // 매퍼선언
  
  @Transactional
  public UserResponse save(UserRequest request) {
   
   ~~~~~~~~~~~~~~~~

    // reqeust로 들어온 Reqeust DTO 객체를 Entity로 전환함.    
    User user = userMapper.toEntity(request, encodedPassword);
    User savedUser = userRepository.save(user);

    // 만들어진 DTO를 Response용으로 전환
    return userMapper.toResponse(savedUser);
    
  }
 ```


