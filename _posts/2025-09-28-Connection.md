---
title: Spring Connection
date: 2025-09-28 10:00:00 +09:00
categories: [Spring Framework, SpringData]
tags: [ JAVA, Spring Framework, SpringData, Connection]
---

## 1. DB Connection
``DB와의 연결 자체를 관리하는 역할``

### 1. DB Connection
 - DB 커넥션 관리 객체 / 인프라 계층
 - 흔히 Connection Pool / DataSource / DB 인프라라고 부름
 - Data Access Layer 와는 다르게, DB와의 연결 자체를 관리하는 역할

### 2. 종류

| 이름                           | 역할                             |
| ---------------------------- | ------------------------------ |
| DataSource                   | DB Connection을 제공하는 표준 객체      |
| HikariCP, Tomcat JDBC Pool 등 | DataSource 구현체, 커넥션 풀 관리       |
| 커넥션 풀                        | 미리 DB 연결을 만들어두고 재사용, 성능 최적화    |
| 트랜잭션 관리                      | 커넥션과 연계하여 commit/rollback 등 처리 |

## 2. DataSource
### 1. DataSource
 - 정의: javax.sql.DataSource는 JDBC에서 제공하는 인터페이스로, DB Connection을 얻는 표준 방법을 정의한 것.
 - 목적: 개발자가 DriverManager.getConnection() 같은 저수준 API를 직접 쓰지 않고, 더 유연하고 확장성 있게 DB 연결을 다룰 수 있도록 도와줌.

### 2. DataSource의 역할
 - Connection 제공자
   - dataSource.getConnection() 호출 → Connection 객체 반환.
   - 내부적으로 어떤 방식으로 연결을 제공할지는 구현체에 따라 다름.
 - 추상화
   - 개발자는 DataSource 인터페이스만 의존 → 실제 구현체(풀링인지, 단순 연결인지)는 바꿔 끼울 수 있음.

### 3. DataSource의 종류
 - 단순 연결용
   - DriverManagerDataSource (스프링 제공)
   - getConnection() 할 때마다 DriverManager.getConnection() 호출 → 매번 새 DB 연결 생성.
   - 주로 테스트용.
 - 커넥션 풀링용
   - HikariDataSource (HikariCP, 스프링부트 기본)
   - BasicDataSource (Apache Commons DBCP)
   - PoolDataSource (Oracle UCP)
   - 미리 여러 커넥션을 만들어둔 풀에서 빌려주고, 반납받으면 재사용.
   - 실무에서는 거의 이 방식 사용.

### 4. Spring에서 DataSource
 ``스프링 컨테이너가 알아서 빈으로 등록``

#### 1. Spring(XML)
  ```
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/mydb" />
        <property name="username" value="root" />
        <property name="password" value="1234" />
    </bean>
  ```
   - applicationContext.xml 같은 설정 파일에 DataSource를 등록
   - 스프링이 dataSource라는 이름으로 DataSource 객체를 관리
   - DAO나 Repository에서 그냥 주입받아 사용

#### 2. 스프링 부트 (Spring Boot)
 ```
 // application.properties
 spring.datasource.url=jdbc:mysql://localhost:3306/mydb
 spring.datasource.username=root
 spring.datasource.password=1234
 spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
 ```
 ```
    private final DataSource dataSource;

    // 생성자 주입
    public UserRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }
 ```
 - application.properties나 application.yml에 DB 설정
 - 스프링 부트가 알아서 HikariDataSource 객체를 만들어서 빈으로 등록

## 3. Connection Pool
### 1. Connection Pool
 - DB Connection을 미리 만들어서 풀(Pool)에 보관해두고, 필요할 때 꺼내서 쓰고 다시 돌려보내는 구조
 - 목적: DB 연결 생성 비용 절감 + 성능 향상

### 2. 필요한 이유
 - DriverManager.getConnection() 호출 → 매번 새로운 DB 연결 생성 → 비용 큼
 - 커넥션 생성과 종료는 네트워크 + DB 자원 소모가 많음
 - 따라서 자주 쓰는 커넥션을 미리 만들어두고 재사용하면 성능 최적화 가능

### 3. 특징

| 항목      | 내용                                        |
| ------- | ----------------------------------------- |
| 초기 생성   | 애플리케이션 시작 시 N개의 Connection 미리 생성          |
| 재사용     | Connection을 사용 후 다시 풀에 반환                 |
| 최대 연결 수 | 풀에 보관 가능한 최대 Connection 수 설정 가능           |
| 성능      | 새 Connection 생성 비용 제거 → DB 접근 성능 향상       |
| 구현체     | HikariCP, Apache DBCP, Tomcat JDBC Pool 등 |

### 4. HikariCP
 - 스프링부트 2.x 이후 기본 DataSource 구현체로 HikariCP 자동 등록
 - 특징

   | 항목  | 특징                           |
   | --- | ---------------------------- |
   | 성능  | 빠른 커넥션 획득/반환, 낮은 오버헤드        |
   | 안정성 | 누수 감지, 유효성 검사, 자동 회복         |
   | 간편함 | 스프링부트에서 자동 설정, 최소 설정으로 사용 가능 |

### 5. 반납
 - 커넥션 풀 사용 시
   - Connection.close() 호출 → 풀로 반환
   - 실제 DB 연결은 끊기지 않고 유지
   - 다음 요청에서 재사용 가능
 - 풀을 사용하지 않는 경우 (DriverManager 등)
   - Connection.close() 호출 → 실제 DB 연결 종료
   - 다음 요청 시 새 연결 생성 필요 → 비용 큼
