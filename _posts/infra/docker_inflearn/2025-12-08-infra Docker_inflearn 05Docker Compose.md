---
title: 05 Docker Compose
date: 2025-12-07 10:00:00 +09:00
categories: [infra, dockerInflearn]
tags: [ infra, dockerInflearn ]
---

## 1. Docker Compose
### 1. Docker Compose
 - 여러 개의 컨테이너로 이루어진 복잡한 애플리케이션을 한 번에 관리할 수 있게 해줌
 - 여러 컨테이너를 하나의 환경에서 실행하고 관리 하는 툴(☆)

### 2. 사용하는 이유
 - 여러 개의 컨테이너로 이루어진 복잡한 애플리케이션을 하나의 환경에서 실행하고 관리 하게함. 
 - 복잡한 명령어로 실행시키던 걸 간소화 시킬 수 있음

## 2. 흐름
### 1. 생성
 - 파일명 : compose.yml // 최상위에
 - 내용
   - compose방식
    ```
     services: # compose 파일 내에서 서비스를 식별하기 위한 이름 
      my-web-server: # 컨테이너에 붙이고 싶은 이름, 서비스의 이름을 붙이는 기능 
        container_name: web-server # 컨테이너를 띄울 때 붙이는 별칭이다. CLI에서 --name web-server 역할과 동일
        image: nginx # 어떤이미지?
        ports:
          - 80:80 # 포트
    ```
   - 기존방식 : ``docker run --name webserver -d -p 80:80 nginx``

### 2. 실행 / 종료
 - yml파일의 위치에서 실행
   ```
   docker compose up -d
   ```
 - 종료
  ```
  docker compose down
  ```

## 3. 명령어 구분
### 1. docker compose up
 - compose.yml 경로에서 하면 compose.yml에 설정한 컴포즈를 실행함.
 - `-d`는 백그라운드
 - `--build` 이미지를 다시 빌드해서 실행시켜야할때

### 2. docker compose down

### 3. docker compose ps
 - `docker ps`는 실행중인 전체 컨테이너
 - `docker compose ps`는 compose.yml에서 실행한 컴포즈중, 실행중인 것만.
 - `-a` 전체 보는 것도 가능

### 4. docker compose logs
 - 로그 보기

### 5. docker compose pull
 - image의 업데이트가 필요한 경우 사용함.
 - 기본적으로는 설치되어있는 이미지를 사용함.

## 4. 예시
### 1. redis
  ```
  services:
    my-cache-server:
      image: redis
      ports: 
        - 6379:6379
  ```

### 2. MySQL
  ```
  services:
    my-db:
      image: mysql
      environment:
        MYSQL_ROOT_PASSWORD: pwd1234 # 환경은 여기에
      volumes:
        - ./mysql_data:/var/lib/mysql # 볼륨 설정
      ports:
        - 3306:3306
  ```

### 3. SpringBoot
  ```
  services:
    my-server:
      build: . # 기존에 작성한 Dockerfile로 실행을 하고, 그리고 그 위치가 compose위치를 기준으로 상대경로를 작성  
      ports:
        - 8080:8080
  ```

## 5. 다중 컨테이너 동시에 띄우기
### 1. mysql + redis
  ```
  services:
    my-db:
      image: mysql
      environment:
        MYSQL_ROOT_PASSWORD: pwd1234 
      volumes:
        - ./mysql_data:/var/lib/mysql
      ports:
        - 3306:3306
    my-cache-server:
      image: redis
      ports:
        - 6379:6379
  ```
### 2. SpringBoot + mysql
- 체크포인트
  - 스프링부트가 먼저 실행이 되면, DBSource가 연결이 안되서 오류가 발생함. 따라서 db가 먼저 올라와야하는 상황
  - 스프링부트에서 localhost 대신 컨테이너의 이름을 입력해야 연결이됨.(☆☆☆☆☆)
  ```
    spring:
     datasource:
       # localhost를 하는경우 스프링컨테이너 내부에서의 3306을 찾는 상황이됨.
       url: jdbc:mysql://my-db:3306/mydb
       username: root
       password: pwd1234
     driver-class-name: com.mysql.cj.jdbc.Drive
  ```
  
 - Compose.yml

  ```
   services:
    my-server:
      build: .
      ports:
      - 8080:8080
      # my-db의 컨테이너가 생성되고 healthy 하다고 판단 될 때, 해당 컨테이너를 생성한다. 
      depends_on:
        my-db:
          condition: service_healthy
          
    my-db:
      image: mysql
      environment:
        MYSQL_ROOT_PASSWORD : pwd1234
        MYSQL_DATABASE : mydb # MySQL 최초 실행 시 mydb라는 데이터베이스를 생성해준다.
      volumes:
        - ./mysql_data:/var/lib/mysql
      ports:
       - 3306:3306
      healthcheck:
        test: [ "CMD", "mysqladmin", "ping" ] # MySQL이 healthy 한 지 판단할 수 있는 명령어
        interval: 5s # 5초 간격으로 체크
        retries: 10 # 10번까지 재시도
  ```
