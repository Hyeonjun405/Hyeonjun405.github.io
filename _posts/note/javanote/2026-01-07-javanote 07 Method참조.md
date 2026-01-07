---
title: 07 Method 참조
date: 2026-01-07 10:00:00 +09:00
categories: [note, JavaNote]
tags: [Java]
---

## 1. Method 참조
### 1. 메소드 참조
 - 메서드 참조는 이미 존재하는 메서드를 람다 표현식 대신 함수형 인터페이스의 구현으로 전달하기 위한 문법
 - 메서드 참조는 함수형 인터페이스 타입이 기대되는 위치에서, 특정 메서드 호출을 람다 표현식 형태로 축약해 표현한 것

### 2. 기본 형태
 - 메서드 참조는 :: 연산자를 사용
 - 구분
   - 클래스명::메서드명
   - 객체참조::메서드명
 - 람다와 비교
  ```
   x -> someMethod(x) // 람다
   SomeClass::someMethod // 메서드 참조   
  ```

### 3. 사용하는 타이밍
 - 람다 내부가 “단일 메서드 호출”일 때
 - 로직이 추가되지 않을 때
 - 의미가 더 명확해질 때
   
## 2. 메소드참조의 종류
### 1. static 메서드 참조
 - 함수형 인터페이스의 메서드 파라미터와 static 메서드 파라미터가 동일해야함
 - 방법
  ```
   (x, y) -> Math.max(x, y) // 람다
   Math::max // 메서드 참조
  ``` 
 
 - 예시
  ```
     BiFunction<Integer, Integer, Integer> max = Math::max; // 
     Integer value = max.apply(3, 7);
     System.out.println("높은 값은 : " + value) // 높은 값은 : 7
  ```

### 2. 인스턴스 메서드 참조 (특정 객체)
 - 방법
   ```
    user -> user.isActive() // 람다 
    user::isActive // 메서드 참조   
   ```
 - 예시
   ```
   User user = new User();
   Supplier<Boolean> s = user::isActive; // User라는 타입에는 isActive라는 메소드가 존재함. 
   ```


### 3. 인스턴스 메서드 참조 (임의 객체)
 - 방법
  ```
   (user) -> user.getName() // 람다
   User::getName // 메서드 참조
  ```
 - 예시 
   ```
    // 람다의 첫 번째 파라미터가 메서드를 호출하는 “대상 객체(this)”
    Function<User, String> f = User::getName;
    user -> user.getName()
   ```

### 4. 생성자 참조
 - 파라미터 없이 생성
  ```
   () -> new User() // 람다
   User::new // 메서드 참조
  ```
 - 파라미터 있는 생성자
  ```
   (name) -> new User(name) // 람다
   User::new // 메서드 참조
  ```










