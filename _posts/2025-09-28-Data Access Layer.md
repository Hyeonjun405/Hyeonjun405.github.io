---
title: Data Access Layer
date: 2025-09-28 10:00:00 +09:00
categories: [Spring Framework, SpringData]
tags: [ JAVA, Spring Framework, SpringData, Data Access Layer]
---

## 1. Data Access Layer
 ``애플리케이션에서 DB와 직접 데이터를 주고받는 모든 기술을 통칭``

### 1. Data Access Layer 종류

| 범주          | 예시                                  |
| ----------- | ----------------------------------- |
| SQL Mapper  | MyBatis, iBatis 등                   |
| ORM         | Hibernate, EclipseLink, JPA 등       |
| 단순 JDBC     | JdbcTemplate, 순수 JDBC               |
| Spring Data | Spring Data JPA, Spring Data JDBC 등 |

### 2. Data Access Layer 변화
#### 1. JDBC 등장 (1997, Java 1.1)
 - 자바에서 DB와 직접 연결하기 위한 표준 API 등장
 - 특징: SQL 직접 작성, Connection/PreparedStatement/ResultSet 관리 모두 수동
 - 장점: 모든 DB 연동 기술의 기반
 - 단점: 반복 코드 많음, 예외 처리 및 리소스 관리 번거로움

#### 2. JdbcTemplate (2003~ 스프링 초창기)
 - 스프링에서 제공하는 JDBC 편의 라이브러리
 - 특징: Connection, PreparedStatement, ResultSet 관리 자동화
 - 장점: 반복 코드 제거, SQL과 결과 매핑에 집중 가능, 예외 통합 처리
 - 단점: 여전히 SQL 직접 작성 필요

#### 3. ORM (1990년대 후반 ~ 2000년대 초반 등장, Hibernate가 2001년)
 - Hibernate, EclipseLink 등
 - 특징: 객체(Entity) ↔ DB 테이블 자동 매핑, SQL 자동 생성
 - 장점: 객체 중심 개발, SQL 작성 최소화
 - 단점: 복잡한 SQL이나 성능 최적화 필요 시 자동 SQL이 비효율적
 - JPA(2006) 등장: ORM 구현체 표준화

#### 4. SQL 매퍼 (2000년대 초반)
 - 철학: “SQL은 내가 직접 짤게, 대신 실행과 매핑만 도와줘”
 - 특징: SQL 중심 개발, 매핑 자동화
 - 장점: 성능/제어 안정적, 대규모 서비스에 적합
 - 단점: 객체 중심 개발은 제한적
 
#### 5. Spring Data 계열 (2010년대 이후)
 - Spring Data JPA 등
 - 특징: JPA/Hibernate 위에 추상화 + 편의 기능 제공
 - 장점: Repository 인터페이스만 정의하면 CRUD 가능, 페이징/정렬 지원, 반복 코드 제거
 - 결과: ORM 장점 유지 + 개발 편의성 극대화
 - 비고: SQL 매퍼(MyBatis)도 여전히 사용 가능
 
## 2. JdbcTemplate
### 1. JdbcTemplate
 - 스프링 프레임워크에서 제공하는 JDBC용 편의 라이브러리
 - 반복적인 JDBC 코드(Connection, PreparedStatement, ResultSet 관리 등)를 제거하고, SQL 실행과 결과 매핑에 집중할 수 있게 해주는 객체

### 2. 목적
 - 반복적인 JDBC boilerplate 코드 제거
 - 개발자가 SQL과 결과 매핑에만 집중하도록 지원
 - 예외 처리 통합 및 리소스 관리 자동화

### 3. 특징
 - 반복 코드 제거
    - Connection, PreparedStatement, ResultSet 생성과 종료를 자동 처리
    - try-catch-finally 같은 반복적인 JDBC boilerplate 최소화
 - SQL 실행과 매핑 집중
   - 개발자는 SQL과 결과 객체 매핑(RowMapper 등)만 신경 쓰면 됨
   - 반복적인 리소스 관리, 예외 처리 신경 쓸 필요 없음
 - 예외 통합 처리
   - SQLException을 스프링 전용 DataAccessException으로 변환
   - DB 벤더별 예외 차이를 통일시켜서 코드 단순화
 - 다양한 편의 메소드 제공
   - query, queryForObject, update, batchUpdate 등
   - 단일 조회, 리스트 조회, 업데이트, 배치 처리 등 대부분 JDBC 작업 지원
 - 스프링과 통합 용이
   - 트랜잭션, DI(Dependency Injection)과 자연스럽게 결합 가능
   
### 3. 소스
```
@Autowired
private JdbcTemplate jdbcTemplate; // jdbcTemplate 주입 받음.

public User findUserById(int id) {
String sql = "SELECT id, name, age FROM users WHERE id = ?";

    return jdbcTemplate.queryForObject( 
        sql,
        new Object[]{id},
        (rs, rowNum) -> new User(
            rs.getInt("id"),
            rs.getString("name"),
            rs.getInt("age")
        )
    );
}
```

## 3. ORM
### 1. ORM
 - 객체 지향 프로그래밍 언어의 객체와 관계형 DB의 테이블을 매핑하는 기술
 - DB 테이블과 자바 클래스(Entity)를 1:1로 연결해서, SQL 없이 객체 중심으로 데이터 처리 가능하게 해줌
### 2. 목적
 - 객체와 테이블 간의 데이터 불일치 문제 해결 (Object-Relational Impedance Mismatch)
 - SQL 직접 작성 없이도 CRUD 가능
 - 객체 중심 개발 철학 유지

### 2. 특징
 - 객체 중심
    - DB 레코드를 자바 객체로 표현
    - SELECT → 객체, INSERT/UPDATE → 객체 상태 반영
 - SQL 자동 생성
   - CRUD, 연관관계(Join) 처리 등을 자동으로 SQL로 변환
 - 연관관계 처리
   - 객체 간 관계(1:1, 1:N, N:M)를 DB FK로 매핑 가능
 - 캐싱 및 성능 최적화
   - 1차 캐시, 지연 로딩(Lazy Loading), Eager Fetch 등 지원

### 3. 대표 구현체
 - Hibernate: 가장 유명한 자바 ORM
 - EclipseLink, OpenJPA: JPA 표준 구현체
 - JPA (Java Persistence API): ORM을 위한 자바 표준 인터페이스

### 4. 예시
### 1. User 엔티티 (User.java)
```
@Entity
public class User {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private int age;
    // getter, setter
}
```
### 2. UserService (UserService.java)
```
@PersistenceContext
private EntityManager entityManager;

@Transactional
public User createUser(String name, int age) {
    User user = new User();
    user.setName(name);
    user.setAge(age);
    entityManager.persist(user);  // 직접 persist 호출
    return user;
}
```
 - 엔티티 객체 관리 주체
   - @Entity가 붙은 클래스는 JPA/Hibernate가 관리
   - 영속 컨텍스트(Persistence Context) 안에서 객체 상태를 추적
   - persist(), merge(), remove() 등의 메소드로 DB와 동기화
 - 스프링의 역할
   - DI(Dependency Injection): EntityManager, Repository를 스프링이 주입
   - 트랜잭션 관리: @Transactional로 commit/rollback 자동 처리
   - 엔티티가 스프링 빈처럼 관리되진 않지만, 스프링이 JPA/Hibernate 사용 환경을 관리
 - 영속 컨텍스트와 스프링
   - 스프링 컨테이너가 EntityManagerFactory를 관리
   - Repository나 Service 계층에서 엔티티를 사용하면, JPA가 상태를 추적하고 스프링이 트랜잭션을 적용

## 4. SQL 매퍼
### 1. SQL 매퍼
 - 개발자가 직접 작성한 SQL을 실행하고, 결과를 자바 객체와 매핑해주는 라이브러리
 - SQL 제어권은 개발자에게 있고, 매핑과 반복 코드를 자동화

### 2. 목적
 - SQL 작성과 실행을 개발자가 직접 제어하면서도, 결과 매핑과 반복적인 JDBC 처리는 자동화
 - ORM의 자동 SQL이 비효율적일 때 성능/제어를 확보

### 3. 특징
 - SQL 직접 작성
    - 개발자가 INSERT, UPDATE, SELECT, DELETE SQL을 직접 작성
    - 복잡한 쿼리, JOIN, 서브쿼리 등 제어 가능
 - 자동 매핑
    - SQL 결과(ResultSet)를 자바 객체로 자동 매핑
    - XML Mapper 또는 Annotation 기반 설정 가능
 - 반복 코드 제거
    - JDBC Connection, PreparedStatement, ResultSet 관리 자동 처리
    - try-catch-finally, 리소스 close 불필요
 - 유연한 매개변수 처리
    - 파라미터 바인딩, Map 또는 객체 전달 가능
    - 동적 SQL 작성 지원 (조건별 SQL 분기 가능)
 - 트랜잭션 통합 가능
    - Spring 트랜잭션 관리와 자연스럽게 결합
 - SQL 중심 철학
    - ORM처럼 SQL을 숨기지 않고, 개발자가 원하는 SQL을 그대로 사용
    - 성능 최적화 및 DB 제어권 확보

### 4. 예시
#### 1. XML
```
<select id="findUser" parameterType="int" resultType="User">
    SELECT id, name, age FROM users WHERE id = #{id}
</select>
```

#### 2. JAVA
```
User user = userMapper.findUser(1);
```

## 4. Spring Data
### 1. Spring Data
 - JPA 위 추상화 레이어로, Repository 인터페이스만 정의하면 CRUD, 페이징, 정렬 등을 자동으로 처리해주는 스프링 라이브러리
 - EntityManager를 직접 다루지 않고도 객체 중심 CRUD와 쿼리를 수행 가능

### 2. 목적
 - 반복적인 CRUD 코드 제거
 - 선언적 트랜잭션과 페이징/정렬 등 편의 기능 제공
 - JPA/Hibernate를 더 쉽게 사용하게 함

### 3. 특징

| 특징            | 설명                                                                                     |
| ------------- | -------------------------------------------------------------------------------------- |
| Repository 기반 | JpaRepository, CrudRepository 인터페이스 상속만으로 CRUD 메소드 제공                                  |
| 자동 CRUD 구현    | `save()`, `findById()`, `findAll()`, `deleteById()` 등 기본 CRUD 자동 처리                    |
| 커스텀 쿼리        | 메소드 이름으로 쿼리 자동 생성 (`findByName`, `findByAgeGreaterThan`), 필요 시 `@Query`로 JPQL 직접 작성 가능 |
| 반복 코드 제거      | EntityManager 직접 호출 불필요, merge/persist/remove, Optional 처리 등 자동 처리                     |
| 트랜잭션 통합       | `@Transactional`로 commit/rollback 자동 처리, 스프링 트랜잭션과 자연스럽게 결합                            |
| 페이징/정렬 지원     | `Pageable`, `Sort` 인터페이스만으로 페이징과 정렬 적용 가능                                              |
| 스프링 통합        | DI, AOP, 트랜잭션 관리 등 스프링 기능 활용 가능                                                        |

### 4. 예시
#### 1. Repository
```
public interface UserRepository extends JpaRepository<User, Long> {
    User findByName(String name);
}
```

#### 2. Servie
```
@Autowired
private UserRepository userRepository;

User user = new User();
user.setName("홍길동");
user.setAge(20);
userRepository.save(user);

User u = userRepository.findById(1L).orElse(null);
User byName = userRepository.findByName("홍길동");
```
