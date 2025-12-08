---
title: 04 Docker File
date: 2025-12-07 10:00:00 +09:00
categories: [infra, dockerInflearn]
tags: [ infra, dockerInflearn ]
---

## 1. Docker File
### 1. Dockerfile
 - Docker 이미지를 만들게 해주는 파일
 - Dockerhub에 올려놓은 Docker 이미지가 아닌, 나만의 Docker 이미지를 만들고 싶을 때 사용
 - 직접 만든 Spring Boot 프로젝트 자체를 Docker 이미지로 만드는 용도

### 2. Note
 - 사용자가 도커파일이라는 스크립트를 작성함.
 - 도커프로그램에서 도커파일을 읽어서 도커이미지를 만듬.
 - 도커는 도커이미지를 가지고 컨테이너를 띄워서 쓸 수 있게 만듬.


## 2. From
### 1. From
 - 베이스 이미지를 생성하는 역할
 - Docker 컨테이너를 특정 초기 이미지를 기반으로 추가적인 셋팅함.
 - 여기서 ‘특정 초기 이미지ʼ가 곧 베이스 이미지

### 2. 흐름
 - 파일명 : `Dockerfile`이라는 파일명으로 고정.
 - 내용 
   ```
    //JDK를 설치함.
    FROM eclipse-temurin:17-jdk
   ```
 - 생성
   ```
   // 맨 마지막에 경로 표기함
   // 지금 명령어에서는 .
   docker build -t my-jdk17-server .
   ```
 - 실행
   ```
   docker run my-jdk17-server
   ```
 - 결과
   ```
   // 실행후 바로 종료
   // 도커입장에서는 JDK17를 태우라고 명령했고, 그다음에 별도 명령이 없어서 종료됨.
   ```

## 3.Copy
### 1. copy
 - 호스트 컴퓨터에 있는 파일을 복사해서 컨테이너로 전달한다.
 - 
 - `COPY [호스트 컴퓨터에 있는 복사할 파일의 경로] [컨테이너에서 파일이 위치할 경로]`

### 2. 흐름
 - 파일 기준 : 도커파일 내 명령어로 존재
 - 생성
  ```
   FROM ubuntu

   // 이부분의 app.txt를 /app.txt에 복사함.
   COPY app.txt /app.txt 

   ENTRYPOINT ["/bin/bash", "-c", "sleep 500"]
  ```
 - 결과
  ```
  // 도커내에 /app.txt 생성완료
  ```

### 3. copy 방법
 ```
  // 특정파일
  COPY app.txt /app.txt 

  // 디렉토리 전체
  COPY myapp /myapp/
 
  // 특정파일의 저장+이동
  COPY *.txt /mytext/
 ```

## 3. dockerignore
### 1. dockerignore
 - 특정 파일/ 경로를 지속적으로 무시하고 싶을 때 사용함.
 - 별도 처리없이 그냥 등록해두면 깃처럼 자동으로 읽고 처리함.

### 2. 흐름
 - 파일명 : `.dockerignore` // 앞에 쩜 붙음.
 - 생성
  ```
   readme.txt
  ```

## 4. ENTRYPOINT
### 1. ENTRYPOINT
 - 컨테이너가 생성되고 최초로 실행할 때 수행되는 명령어를 뜻
 ``` 
   ENTRYPOINT [명령문...]
   ENTRYPOINT ["node", "dist/main.js"]
 ```

### 2. 흐름
 - 파일명 : 도커파일 내에 입력
 - 생성
 ```
  FROM ubuntu
  // 기본적으로 리눅스 명령어로!
  ENTRYPOINT ["/bin/bash", "-c", "echo hello"]
 ```


## 5. 스프링 부트실행
### 1. 테스트 환경
 - 스프링 프로젝트 빌드(그레들 형태)

### 2. 흐름
 - 그레들 빌드를 통해서 jar파일을 생성함.
 - dockerfile에서 jar파일을 컨테이너로 이동함.
  ```
  FROM eclipse-temurin:21-jdk
  COPY build/libs/*SNAPSHOT.jar app.jar
  ```
 - 실행 후 작업으로 
  ```
   ENTRYPOINT ["java", "-jar", "/app.jar"]
  ```

## 6. Run
### 1. run
 - 이미지 생성 과정에서 명령어를 실행시켜야 할 때 사용
 - RUN -> ‘이미지 생성 과정 ʼ에서 필요한 명령어를 실행시킬 때 사용
 - ENTRYPOINT -> 생성된 이미지를 기반으로 컨테이너를 생성한 직후에 명령어를 실행시킬 때 사용
 - 문법
  ```
   RUN [명령문]
   RUN npm install
  ```

### 2. 흐름
 - Dockerfile 내 설정
 - 흐름
  ```
   //깃 설치함.
   RUN apt update && apt install -y git
  ```

## 7. WORKDIR
### 1. WORKDIR
 - WORKDIR 으로 작업 디렉터리를 전환하면 그 이후에 등장하는 모든 `RUN`,`CMD`,`ENTRYPOINT`,`COPY`,`ADD` 명령문은 해당 디렉터리를 기준으로
   실행함.
 - 섞여있는 경우에

### 2. 흐름
 - Docker파일 내에 실행
 - 흐름

  ```
  FROM ubuntu
  
  //my-dir내에서 작업하는 것으로 변경
  WORKDIR /my-dir  
  COPY ./ ./
  ENTRYPOINT "/bin/bash", "-c", "sleep 500"]
  ```

 - 결과
   - 해당 도커파일에서는 모든 파일이 /my-dir에 생성
   - 최초 접근 dir도 /my-dir로 변경됨.

## 8. EXPOSE
### 1. expose
 - 컨테이너 내부에서 어떤 포트에 프로그램이 실행되는 지를 문서화하는 역할
 - `docker -p 8080:8080`와 같은 명령어의 `-p` 옵션과 같은 역할은 일체 하지 않음.
 - 쉽게 표현하자면 EXPOSE 명령어는 쓰나 안 쓰나 작동하는 방식에는 영향을 미치지 않는다. 
