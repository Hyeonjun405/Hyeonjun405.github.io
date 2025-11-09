---
title: 06 mybatis interceptor
date: 2025-11-09 10:00:00 +09:00
categories: [00Spring Framework, SpringData]
tags: [ Java, Spring Framework, SpringData, mybatis]
---

## 1. mybatis

## 2. mybatis 인터셉터 흐름
### 1. 스프링 컨트롤러 / 서비스 호출
  - 서비스에서 Mapper 메소드를 호출
  - 예: userMapper.findById(10)
  - 이때는 단순히 메소드 호출일 뿐, DB와 아직 연결되지 않음.

### 2. Mapper 프록시 호출
  - 마이바티스는 Mapper 인터페이스를 동적으로 프록시로 만들어서, 호출이 들어오면 내부 SqlSession로 전달
  - Mapper 메소드 호출 → SqlSession에서 해당 SQL ID와 파라미터 확인
  - Note
    - 인터페이스 자체는 실행하면 런타임 오류가 발생하므로 마이바티스에서 해당 인터페이스 호출을 하면 가로채서 해당하는 작업을 하려고 프록시 형태로 사용함.
    - 즉 프록시는 호출된 메소드 이름과 파라미터를 읽고, 내부적으로 MappedStatement → BoundSql → Executor 순으로 SQL 실행을 연결하는 역할

### 3. MappedStatement 조회
  - SQL ID를 기반으로 Mapper XML 또는 어노테이션에 정의된 SQL을 찾아 MappedStatement 객체 생성
  - 여기에는 SQL 문자열, 파라미터 타입, 결과 매핑 정보 등이 담겨 있음
  - Note
    - MappedStatement 객체는 쉽게 말해서 SQL을 어떻게 써야하는지 파라미터는 어디에 뭘넣는지정보
    - SQL 정보 / 파라미터 정보 / 결과 매핑 정보(ResultMap) / 기타 실행 옵션

### 4. BoundSql 생성
  - 실제 SQL 실행 전, 파라미터를 바인딩할 준비를 함
  - SQL 안의 #{} 플레이스홀더가 ?로 바뀌고, BoundSql에 파라미터 매핑 정보가 함께 저장됨
  - Note
    - BoundSql은 MappedStatement에서 정의된 SQL과 파라미터 매핑 정보를 바탕으로 실제 실행 가능한 형태의 SQL을 준비한 객체

### 5. Executor로 전달 → Statement 생성
  - Executor가 Connection을 받아 PreparedStatement 혹은 CallableStatement 생성
  - SQL이 DB에 맞게 준비됨 (? 포함 상태)
  - **이 과정에서 인터셉터가 끼어들 수 있음** → SQL 확인, 파라미터 조작, 권한 필터 등

### 6. 파라미터 바인딩
  - ? 위치에 실제 파라미터 값을 채움
  - 마이바티스 내부에서 ParameterHandler가 담당
  - **인터셉터를 사용하면 이 단계 전후에도 조작 가능**
  - Note
    - 마이바티스 자체는 일반적으로 “실제 값이 채워진 SQL 문자열”을 직접 제공하지 않음
    - 보통 로그용으로 만들 때는 BoundSql + ParameterMapping + ParameterObject를 활용해서 수동으로 치환하는 방식으로 완성된 SQL을 만들어야 함

### 7. SQL 실행
  - PreparedStatement.executeQuery() 또는 executeUpdate() 호출
  - DB에서 실제 결과가 나옴

### 8. 결과 매핑
  - ResultSet을 자바 객체로 매핑 (ResultMap)
  - Mapper 메소드의 반환 타입에 맞춰 객체, 리스트, Map 등으로 변환

### 9. Mapper 메소드 종료 → 서비스/컨트롤러로 반환
  - 이제 결과 객체가 서비스로 넘어가고, 컨트롤러로 넘어가 화면에 전달 가능

## 3. BoundSql + ParameterMapping + ParameterObject
### 1. 흐름
  ```
     1) XML / Annotation Mapper → SQL 템플릿
     2) MappedStatement 생성 (SQL + 파라미터명 위치 정보 보유)
     3) SqlSession 실행 시 → BoundSql 생성 (SQL + parameterMappings + parameterObject)
     4) Executor가 PreparedStatement 생성
     5) ParameterHandler가 parameterObject에서 값 꺼내 parameterMappings 순서대로 바인딩```
  ```
### 2. Source
#### 1. SQL
  ```
   <select id="findUser" parameterType="map" resultType="User">
    SELECT id, name, age
    FROM users
    WHERE name = #{name} AND age > #{age}
   </select>
  ```
#### 2. Service
  ```
  Map<String, Object> param = new HashMap<>();
  param.put("name", "Hyeonjun");
  param.put("age", 20);
  
  userMapper.findUser(param);
  ```

#### 3. BoundSql
  ```
  BoundSql {
    sql: "SELECT id, name, age
          FROM users
          WHERE name = ? AND age > ?"
    parameterMappings: List<ParameterMapping>
    parameterObject: Map<String, Object> param
  }
  ```

#### 4. ParameterMapping
  ```
  parameterMappings = [
    ParameterMapping { property: "name", javaType: String.class },
    ParameterMapping { property: "age", javaType: Integer.class }
  ]
  ```

#### 5. ParameterObject
  ```
  parameterObject = {
    "name" : "Hyeonjun",
    "age"  : 20
  }
  ```

## 4. interceptor
#### 1. interceptor override method
  ```
  @Override
  public Object intercept(Invocation invocation) throws Throwable {
  
      Object[] args = invocation.getArgs();
  
      // 1) MappedStatement 꺼냄
      MappedStatement ms = (MappedStatement) args[0];
  
      // 2) ParameterObject 꺼냄 (select/update/insert/… 다 동일)
      Object parameterObject = null;
      if (args.length > 1) {
          parameterObject = args[1];
      }
  
      // 3) 완성된 SQL 문자열 생성
      String sql = getCompleteSql(ms, parameterObject);
  
      System.out.println("실행되는 SQL = " + sql);
  
      // 4) 다음 체인 실행
      return invocation.proceed();
  }
  ```

#### 2. getCompleteSql
  ```
  public static String getCompleteSql(MappedStatement ms, Object parameterObject) {
      BoundSql boundSql = ms.getBoundSql(parameterObject);
      String sql = boundSql.getSql().replaceAll("\\s+", " ").trim(); // 보기 좋게 정리
  
      List<ParameterMapping> paramMappings = boundSql.getParameterMappings();
      TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();
  
      for (ParameterMapping paramMapping : paramMappings) {
          String propertyName = paramMapping.getProperty();
  
          Object value;
  
          // 파라미터가 단일 값인지 DTO/Map인지 구분
          if (boundSql.hasAdditionalParameter(propertyName)) {
              value = boundSql.getAdditionalParameter(propertyName);
          } else if (parameterObject == null) {
              value = null;
          } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
              value = parameterObject;
          } else {
              MetaObject metaObject = ms.getConfiguration().newMetaObject(parameterObject);
              value = metaObject.getValue(propertyName);
          }
  
          // 값이 문자열이면 작은따옴표 처리
          String valueStr = (value instanceof String) ? "'" + value + "'" : String.valueOf(value);
  
          // 첫 번째 ? 만 치환
          sql = sql.replaceFirst("\\?", valueStr);
      }
  
      return sql;
  }
  ```


## 5. Note - 중복 생성
  ```  
    // 스레드별 마지막 실행 Mapper ID 저장
    private static final ThreadLocal<String> lastMapperId = new ThreadLocal<>();
  
   @Override
   public Object intercept(Invocation invocation) throws Throwable {

        Object[] args = invocation.getArgs();

        // 1) MappedStatement 꺼냄
        MappedStatement ms = (MappedStatement) args[0];

        // 2) ParameterObject 꺼냄
        Object parameterObject = null;
        if (args.length > 1) {
            parameterObject = args[1];
        }

        // 3) 완성된 SQL 문자열 생성
        String sql = getCompleteSql(ms, parameterObject);

        // 4) 중복 로그 방지 (ThreadLocal 기반)
        String currentId = ms.getId();
        if (!currentId.equals(lastMapperId.get())) {
            System.out.println("실행 Mapper: " + currentId);
            System.out.println("실제 SQL: " + sql);
            lastMapperId.set(currentId);
        }

        // 5) 다음 체인 실행
        Object result = invocation.proceed();

        // ThreadLocal 초기화
        lastMapperId.remove();

        return result;
  ```
 - 마이바티스 내부 실행 흐름상 2번 호출됨 (?) 
   - 2025.11.09. 아무리 봐도 gpt도 여기저기 그냥 흐름상 발생한다는데 이해를 못하겠네.. 일단 너무 기술의 깊은 부분 같기도하고 JPA 위주로 사용하는 흐름이니 지금 중요한건 흐름을 파악하고 잘 활용하는 것이므로!, 조금 더 흐름타보고 언젠가 기회가 되면 이해할 수 있기를!!
 - 스레드로컬(쓰레드 고유의 저장공간)이라는 변수를 사용해서 아이디값을 저장해놓고 같은거는 통과
