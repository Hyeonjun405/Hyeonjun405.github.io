---
title: 06 lambda
date: 2026-01-03 10:00:00 +09:00
categories: [note, JavaNote]
tags: [Java]
---

## 1. lambda - 일반 클래스
### 1. 인터페이스

  ```
    @FunctionalInterface
    interface Calculator {
        int calculate(int a, int b);
    }
  ```

### 2. 익명클래스

  ```
    public static void main(String[] args) {

      Calculator calculator = new Calculator() {
          @Override
          public int calculate(int a, int b) {
              return a + b;
          }
      };
      
      int result = calculator.calculate(1, 2);
    }

  ```

### 3. lambda로 변환

  ```
    public static void main(String[] args) {

      /*
      Calculator calculator = new Calculator() {
          @Override
          public int calculate(int a, int b) {
              return a + b;
          }
      };
      */

      // 람다로 전환
      // 변수 a,b를 a+b로 처리함.
      Calculator calculator = (a, b) -> a + b;    

      int result = calculator.calculate(1, 2);
    }
  ```

## 2. lambda - 자료구조
### 1. User Class

  ```
  public class User {
      String name;
      boolean active;
  
      public User(String name, boolean active) {
          this.name = name;
          this.active = active;
      }
  
      public boolean isActive() {
          return active;
      }
  }
  ```


### 2. Filter Interceptor

  ```
  // oneMethod만 정의함.
  public interface UserFilter {
      boolean oneMethod(User user);
  }
  ```

### 3. UserService

  ```
  public class UserService {
  
    /*
     파라미터를 (List<User> users, UserFilter filter)로 받아서
     filter.oneMethod(user)가 true면 users(1번파라미터)에 넣고
     최종 List<User>를 return함.
     */
    List<User> customFilter(List<User> users, UserFilter filter) {
        List<User> result = new ArrayList<>();

        for (User user : users) {
            if (filter.oneMethod(user)) {
                result.add(user);
            }
        }

        return result;
    }
}
  ```

### 3. 람다 전환 전

  ```
    public static void main(String[] args) {

        List<User> users = List.of(
                new User("A", true),
                new User("B", false),
                new User("C", true),
                new User("D", false)

        );

        UserService service = new UserService();

        List<User> activeUsers =
                // users안에 있는 객체들을 순회하기 위한 for문은 customFilter의 역할
                // 메인에서는 UserFilter을 어떻게 구현할 것인가를 정의해야 하기때문에 익명클래스.    
                service.customFilter(users, new UserFilter() {  
                    @Override
                    public boolean oneMethod(User user) {
                        return user.isActive(); 
                    }
                });
        System.out.println("전체 사이즈 : " + activeUsers.size());
    }
  ```

### 4. 람다 전환

  ```
  public static void main(String[] args) {

        List<User> users = List.of(
            new User("A", true),
            new User("B", false)
            new User("C", true),
            new User("D", false)
        );

        UserService service = new UserService();

      /*
       user -> user.isActive()는
       문맥상에서 컴파일러가 UserFilter타입의 자리(filter의 두번째 파라미터) 추측 가능하고,
       UserFilter 타입의 인터페이스에서 추상 메서드가 1개로 정의되었을때,       
       UserFilter 인터페이스를 구현한 이름 없는 클래스 하나를 컴파일러가 대신 만들어 주는 표현
       */
        List<User> activeUsers =
            service.filter(users, user -> user.isActive());

        System.out.println(activeUsers.size());
    }
  ```

## 3. 표준 함수형 인터페이스

### 1. 표준 함수형 인터페이스
 - 자바가 “람다를 제대로 쓰라고” 미리 만들어 둔 표준 함수형 인터페이스 묶음
 - 필요한 것들을 활용해서 람다를 만들어서 사용하면 됨.
 - 각기 인터페이스마다 메소드가 1개씩 정해져있음.

### 2. Consumer<T>
 - 받아서 소비만 하는 인터페이스
 - 파라미터가 있고 리턴이 없음
 - 용도
   - 출력
   - 저장
   - 로그
   - 상태 변경
 - 예시

   ```
    Consumer<Integer> consumer = num -> System.out.println(num);
    consumer.accept(5); // 콘솔에 5가 나옴. 
        
    Consumer<String> consumer = str -> System.out.println(str);
    consumer.accept("a"); // 콘솔에 a가 나옴.
   ```

### 3. Supplier<T>
 - 받아오는 것 없이 값을 제공 
 - 파라미터가 없고, 리턴값이 있음.
 - 용도
   - 객체 지연 생성
   - 기본값 공급
   - 팩토리 역할
 - 예시

   ```
    Supplier<Double> supplier = () -> new Random().nextInt(10);
    
    System.out.println(supplier.get()); // new Random().nextInt(10)으로 나온 값.
   ```

### 4. Function<T, R>
 - T를 받아서 R로 변환
 - 파라미터와 리턴값이 있음.
 - 용도
   - 매핑
   - 변환
   - DTO → Entity
   - 값 가공
 - 예시

   ```
     // 타입변환이 가능함!
     // str값을주면 -> 이후 작업을 해서 두번째 타입으로 리턴함.
     Function<String, Interger> function = str -> Integer.parseInt(str);

     int result = function.apply("1234");
     System.out.println(result); //콘솔에 1234가 나옴 
   ``` 

### 5. Operator
 - 자기 자신 타입으로 연산
 - 실체는 Function의 특수 케이스 / 연산이라는 의미를 코드에 드러냄
 - Function에서 제약을 더 걸어서 만들어진거라고 봐야함.
 - 예시

  ```
   // 타입변환이 불가함, 단순 계산용
   //UnaryOperator<T> extends Function<T, T>
   UnarayOperator<Integer> operator1 = num -> num * num;
   int result = operator1.apply(2); //콘솔에 4
   
   //BinaryOperator<T> extends BiFunction<T, T, T>
   BinaryOperator<Integer> operator2 = (num1, num2) -> num1 + num2;
   int result = operator2.apply(1,2); //콘솔에 3 
    
  ```

### 6. Predicate<T>
 - 조건 검사하는 인터페이스
 - 작업후에 결과로 참/거짓 판단함.
 -  용도 
    - 필터링
    - 검증
    - 조건 분기
 - 예시

  ```
   // 결과 값이 맞냐 틀리냐 판단
   Predicate<String> predicate = str -> str.length() > 5;
   predicate.test("text"); // false
  ```
