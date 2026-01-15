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
### 1. static 메서드 
   ```
    Class A {
     static void mA(int x) { // 스테틱 메서드
       System.out.println(x);
     }
    }  
   ```
   ```
    // 람다 
    IntConsumer intConsumer = x -> System.out.println(x);
    
    // 람다 + 정적메소드 이기 때문에 객체 생성 불필요해짐X
    IntConsumer intConsumer = x -> a.mA(X); // 스테틱 메소드를 참조함
    
    // 메서드 참조 
    IntConsumer intConsumer = A:mA; // 스테틱 메소드를 참조함
   ```

### 2. 인스턴스 메서드 참조 (특정 객체)
   ```
    Class A {
      static void mB(long y) { // 일반 메서드
       System.out.println(y);
      }
    }  
   ```
   ```
    // 람다 
    LongConsumer longConsumer = y -> System.out.println(y);
    
    // 스테틱이 아니기 때문에 사용할 객체생성함.
    // 람다 + 메서드 
    A a = new A();
    LongConsumer longConsumer = a.mB(y);

    // 메서드 참조
    // A aa = new A(); 
    LongConsumer longConsumer = aa::mB;
   ```

### 3. 인스턴스 메서드 참조 (임의 객체)
   ```
    Class A {
      static void mC(dobule z) { // 일반 메서드
       System.out.println(z);
      }
    }  
   ```
   ```
    // 람다
    objDoubleConsumer<A> objDOulbeCOnsumer = (obj, z) -> obj.mC(z);
   
    // 클래스 자체를 파라미터로 넘겨줘서 사용하는 형태
    // + 제네릭으로 클래스 타입을 정해줘야함.
    objDoubleConsumer<A> objDOulbeCOnsumer = A:mc
   ```

### 4. 생성자 참조
   ```
    Class A {
      A(){
       System.out.println("A 생성자 호출");
      }
      
      A(int i){
        System.out.println(i + "를 문자열로");
      }
      
    }  
   ```
   ```
   // 람다 
   Supplier<A> supplier = () -> new A(); // 생성자
   
   // 제네릭에서 정해준 클래스 타입
   // new를 통해 그냥 생성자 호출함.
   Supplier<A> supplier = A::new;
   
   supplier.get();
         
   //람다
   IntFunction<A> intFunction = i -> new A(i);
   
   // 위 방식과 동일
   // IntFunction<A>는 Int를 받아서 A타입으로 리턴한다는 의미를 암묵적으로 가지고 있음.
   // 따라서 별도의 타입을 안써줘도 알아서 처리.
   IntFunction<A> intFunction = A::new; 

   intFunction.apply(10);
   ```










