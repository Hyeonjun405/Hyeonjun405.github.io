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

