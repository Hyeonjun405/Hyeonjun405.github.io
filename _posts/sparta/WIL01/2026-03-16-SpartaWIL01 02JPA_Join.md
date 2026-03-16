---
title: 02 JPA Entity Join 설정
date: 2026-03-16 10:00:00 +09:00
categories: [Sparta, SpartaWIL01]
tags: [ Java, Spring Framework, JPA ]
---

## 1. Note
### 1. Note
  - 기존에는 SQL로 Join으로 작업을 진행했고 디비에서 알아서 필터가 되는 상황이였다면, JPA는 엔티티에서 Join을 설정하고 진행함.

### 2. N+1 패턴
  - Join을 사용하다보면 N+1 패턴이 발생하고 그와 관련된 여러가지 개념이 필요
  - 상황
    - user(1)와 Order(N) 테이블이 있을때 Lazy 전략을 사용할 경우에,
    - 초기에 User를 조회하면, 캐시에 user 테이블정보만 있고 Order는 없음
    - 이때 Order가 필요한 경우가 생겨버리면,
    - 필요한 Order만큼 N번씩 조회하는 상황이 생김.
    - 나는 조회를 1번했지만 캐시에 전체 Order가 없기 때문에 필요한 order 엔티티를 계속 조회하는 상황.
 
## 2. JPA Entity Join
### 1. 관계정의
  
  | 애노테이션       | 설명                                     | 주로 사용되는 곳          |
  | ----------- | -------------------------------------- | ------------------ |
  | @ManyToOne  | 현재 엔티티가 FK를 가지고 있고, 대상 엔티티가 N:1 관계일 때  | Order → User       |
  | @OneToMany  | 대상 엔티티가 FK를 가지고 있는 N:1 관계를 역방향에서 접근할 때 | User → Order       |
  | @OneToOne   | 1:1 관계, FK를 어느 쪽에 둘지에 따라 주체/참조를 결정     | User → UserProfile |
  | @ManyToMany | N:N 관계, 중간 테이블을 통해 조인                  | Student ↔ Course   |

### 2. 조인 컬럼 관련
  
  | 애노테이션                | 설명                                        | 사용 예시                                                    |
  | -------------------- | ----------------------------------------- | -------------------------------------------------------- |
  | @JoinColumn          | FK 컬럼을 직접 지정, 주로 ManyToOne, OneToOne에서 사용 | `@JoinColumn(name="user_id")`                            |
  | referencedColumnName | FK가 참조할 대상 컬럼명 지정 (기본: 대상 테이블 PK)         | `@JoinColumn(name="user_id", referencedColumnName="id")` |
  | nullable             | FK 컬럼 null 허용 여부                          | `@JoinColumn(nullable=false)`                            |
  | unique               | FK 컬럼의 유니크 제약                             | `@JoinColumn(unique=true)`                               |

### 3. 중간 테이블
  
  | 애노테이션              | 설명                 |
  | ------------------ | ------------------ |
  | @JoinTable         | N:N 관계에서 중간 테이블 정의 |
  | joinColumns        | 현재 엔티티 FK 컬럼 정의    |
  | inverseJoinColumns | 상대 엔티티 FK 컬럼 정의    |

  ```
  @ManyToMany
  @JoinTable(
      name="student_course",
      joinColumns=@JoinColumn(name="student_id"),
      inverseJoinColumns=@JoinColumn(name="course_id")
  )
  private List<Course> courses;
  ```

### 4. 기타 옵션 관련
  
  | 애노테이션/옵션      | 설명                                                         |
  | ------------- | ---------------------------------------------------------- |
  | fetch         | LAZY / EAGER 로딩 전략 지정 (`@ManyToOne(fetch=FetchType.LAZY)`) |
  | cascade       | 연관 엔티티에 대한 영속성 전파 (ALL, PERSIST, REMOVE 등)                 |
  | mappedBy      | 양방향 관계에서 “FK가 있는 필드명” 지정, OneToMany/OneToOne에서 주로 사용       |
  | orphanRemoval | 부모 엔티티에서 제거하면 자식 엔티티도 삭제 여부 결정                             |

 - fetch의 Lazy, Eager

   | 구분     | LAZY   | EAGER                  |
   | ------ | ------ |------------------------|
   | 조회 시점  | 필요할 때  | 즉시                     |
   | 초기 SQL | 부모만 조회 | 부모 + 연관                |
   | 객체 상태  | 프록시    | 실제 객체                  |
   | 성능     | 보통 유리  | 상황에 따라 비효율             |
    | 기본값 | @OneToMany / @ManyToMany | @ManyToOne / @OneToOne |


## 3. Join 설정
### 1. Note
 - Order 와 User기준으로,
 - User는 여러주문이 가능할때,
 - Order N : user 1 => N:1 구조

### 2. ManyToOne
 - Order(Many)는 user(One) 한명에게 여러값을 가짐(2번과 3번)
 - Order Entity
   ```
    @Entity
    public class Order {

    @Id
    @GeneratedValue
    private Long id;

    // Order Entity에서는 user_id라는 컬럼이 생기고
    // user의 id값을 FK로하고
    // 엔티티에서 필드는 user로 설정함
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")      
    private User user;                 
   
    }
   ```
   
### 3. OneToMany
 - user(One)는 여러 Order(Many)를 가짐(2번과 3번)
 - User Entity
  ```
   @Entity
   public class User {

    @Id
    @GeneratedValue
    private Long id;

    // User테이블은 FK로 별도 컬럼 생성 X
    // 무엇을 FK할지는 ManytoOne에서 결정하고,
    // FK를 건쪽의 필드값(Order 엔티티의 user라는 필드값이 mappedABY로)
    // 여러가지 Order을 가지기때문에 List로 받게됨.
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>(); 
   }
  ```

### 4. OneToOne
 - User라는 테이블과 userInfo 테이블등 1:1 매핑
 - userProfile Entity(FK가 있는 쪽)
  ```
  @Entity
  public class UserProfile {
  
      @Id
      @GeneratedValue
      private Long id;
  
      @OneToOne
      @JoinColumn(name = "user_id") // FK UserProfile 테이블에 존재
      private User user;
  }
  ```
 - User Entity(FK가 없는 쪽)
  ```
  @Entity
  public class User {
  
      @Id
      @GeneratedValue
      private Long id;
  
      @OneToOne(mappedBy = "user")
      private UserProfile profile;
  }
  ```

### 5. ManyToMany
 - N:N인 상황에서는 중간 테이블을 두고 조인을 함.
 - Student Entity
  ```
    @Entity
    public class Student {
    
        @Id
        @GeneratedValue
        private Long id;
    
        private String name;
    
        // StudentCourse 중간 테이블을 통해 Course와 연결
        @OneToMany(mappedBy = "student", cascade = CascadeType.ALL, orphanRemoval = true)
        private List<StudentCourse> studentCourses = new ArrayList<>();
    }
  ```

 - Cource Entity
  ```
    @Entity
    public class Course {
    
        @Id
        @GeneratedValue
        private Long id;
    
        private String title;
    
        // StudentCourse 중간 테이블을 통해 Student와 연결
        @OneToMany(mappedBy = "course", cascade = CascadeType.ALL, orphanRemoval = true)
        private List<StudentCourse> studentCourses = new ArrayList<>();
    }
  ```
 - StudentCoure(중간 테이블)
  ```
    @Entity
    public class StudentCourse {
    
        @Id
        @GeneratedValue
        private Long id;
    
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "student_id")
        private Student student;
    
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "course_id")
        private Course course;
    
        private LocalDate enrolledDate; // 선택적: 수강일 등 추가 정보
    }
  ```

