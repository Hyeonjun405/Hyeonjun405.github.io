---
title: 03 Docker Image 관리
date: 2026-04-30 09:00:00 +09:00
categories: [Sparta, SpartaWIL08]
tags: [ Java, Spring Framework ]
---

## 1. Note
### 1. Note
- 이미지를 2개를 쓰는 패턴
  - 딱 필요한 것만 컨테이너를 띄워서 사용한다는게 많이 괜찮은 것 같았는데 
  - 생각해보니 젠킨스로 CI/CD 각각 빌드/배포용 이미지를 띄우고, 
  - 각 이미지에서 타겟 컨테이너로 디플로이하는 방식을 가지고 있었음.
  - 띄우려는 환경 자체에 빌드/배포용 이미지를 띄워서 방화벽이나 보안쪽을 덜 신경쓰는 줄 알았는데
  - 그 외에도 메리트가 많은 것 같음
- 어떤 방식으로 이미지를 띄우는가도 중요한 포인트 같음!
  - 근데 정답은 최소한의 용량과 메모리를 잡아먹는 
  - 굉장히 경량화된 컨테이너 정확히 띄우는게 중요하기는 할듯

## 2. Docker Image
### 1. Docker Image
- 도커이미지에 포함되는 내용
  - 애플리케이션 코드
  - 실행에 필요한 라이브러리
  - OS 레벨 설정 (파일 시스템 일부)
  - 환경설정

### 2. 컨테이너와 구분

| 구분       | Image          | Container |
| -------- | -------------- | --------- |
| 의미       | 설계도            | 실행된 상태    |
| 변경 가능 여부 | 불변 (immutable) | 변경됨       |
| 역할       | 실행 준비          | 실제 실행     |

### 3.전반적인 흐름

```
Dockerfile → Image 생성 → Container 실행

# Dockerfile - “어떻게 만들지” 정의
# Image - 만들어진 결과물
# Container - 실행된 상태
```
 
### 4. 이미지들
  
  | 명칭                 | 설명                  | 예시                  | 특징                           |
  | ------------------ | ------------------- | ------------------- | ---------------------------- |
  | Base Image         | 이미지의 시작점 (`FROM`)   | `openjdk:17-jdk`    | OS + 런타임 포함, 모든 이미지의 뿌리      |
  | Parent Image       | 현재 이미지 바로 아래 레이어    | `openjdk → my-app`  | Base와 거의 동일하게 쓰지만 레이어 개념 강조  |
  | Builder Image      | 빌드용으로만 사용하는 이미지     | `gradle:8.5-jdk17`  | 컴파일/빌드 전용, 최종 이미지엔 보통 포함 안 함 |
  | Stage Image        | 멀티 스테이지의 각 단계 이미지   | `AS builder`        | 단계별로 나뉜 이미지                  |
  | Intermediate Image | 빌드 중 자동 생성되는 중간 이미지 | (Docker 내부 생성)      | 캐시용, 직접 쓰진 않음                |
  | Final Image        | 빌드 결과로 나온 최종 실행 이미지 | `my-app:1.0`        | 컨테이너 실행에 사용                  |
  | Tagged Image       | 태그가 붙은 이미지          | `my-app:latest`     | 버전 관리 가능                     |
  | Dangling Image     | 태그 없는 이미지           | `<none>:<none>`     | 정리 대상 (`prune`)              |
  | Local Image        | 내 PC에 저장된 이미지       | `docker images` 목록  | 실행 속도 빠름                     |
  | Remote Image       | 레지스트리에 있는 이미지       | Docker Hub, ECR     | pull 필요                      |
  | Pulled Image       | 외부에서 내려받은 이미지       | `docker pull nginx` | Local Image로 저장됨             |


## 3. DockerFile
### 1. 구조

```
# 1. 기본 이미지 선택
FROM python:3.9

# 2. 작업 디렉토리 설정
WORKDIR /app

# 3. 필요 패키지 복사
COPY requirements.txt ./app

# 4. 패키지 설치
RUN pip install -r requirements.txt

# 5. 소스 코드 복사
COPY . .

# 6. 컨테이너 시작 시 실행할 명령어 지정
CMD ["python", "app.py"]
```

### 2. 빌드 및 실행
```
# 현재 디렉토리에서 Dockerfile을 기반으로 이미지 빌드 (-t 옵션: 태그 지정)
# .은 (빌드 컨텍스트) -> 빌드시에 사용가능한 파일 경로
# 스프링 프로젝트에서 . 대신에 /docker을 했다면 src접근이  불가능해짐
docker build -t my-python-app .

# 생성된 이미지 확인
docker images

# Docker 컨테이너 실행
docker run -d -p 5001:5000 my-python-app
```


### 3. Note
#### 1. COPY
```
# project root 디렉토리에서 실행
docker build -f docker/Dockerfile . 
 
# DockerFile에서
WORKDIR /app
~~~~
COPY . . 
 
 
# Project root 디렉토리의 파일을 
# 현재 빌드 중인 이미지의 /app에 복사한다
```

#### 2. Docker Ignore (불필요한 파일 제거)
  - `.dockerignore` 파일은 Docker 빌드 과정에서 제외할 파일과 디렉터리를 지정하는 역할
  - Docker는 docker build를 실행할 때, 빌드 컨텍스트의 모든 파일을 Docker 컨텍스트에 복사가 가능함.
  - 이때 .dockerignore 파일을 사용하면 불필요한 파일을 제외하여 빌드 속도를 향상시키고, 보안성을 강화할 수 있음
  - `.dockerignore` 파일이 필요한 이유
    - 빌드 속도 향상 → 불필요한 파일을 제외하여 Docker 컨텍스트 크기를 줄임
    - 보안 강화 → 환경 변수 파일(`.env`), 인증 정보(`.git`, `node_modules`) 등이 이미지에 포함되지 않도록 방지
    - 이미지 크기 최소화 → 최적화된 Docker 이미지를 생성하여 배포 용량을 줄임

#### 3. 레이어캐싱
- 레이어 캐싱
  - 명령어의 내용이 이전 빌드와 동일하다면, 새로운 레이어를 만들지 않고 기존에 만들어 두었던 레이어를 재사용함
  - 자주 변경되지 않는 내용은 앞부분에, 자주 변경되는 내용은 뒷부분에 배치하는 것이 효율적
- 반복되는 RUN 
  ```
  #AS-IS
  FROM ubuntu:22.04
  RUN apt-get update # 캐싱1
  RUN apt-get install -y curl #캐싱 2
  RUN rm -rf /var/lib/apt/lists/* # 캐싱 3
  
  #TO-BE
  FROM ubuntu:22.04
  RUN apt-get update && \   # 한번에 캐싱해서 메모리 감소 
  apt-get install -y curl && \
  rm -rf /var/lib/apt/lists/*
  ```

## 4. From 2개 이미지
### 1. Docker File
```
# 1단계: 빌드 환경
# 빌드용 이미지
# Gradle과 JDK가 설치된 큰 이미지를 'builder'로 지정
FROM gradle:8.5-jdk21 AS builder 

# 작업 공간은 /app
WORKDIR /app  

# 캐시 최적화를 위해 Gradle 관련 파일 먼저 복사
COPY build.gradle settings.gradle gradlew ./
COPY gradle ./gradle
# 💡 build.gradle 파일이 변경되지 않으면, 아래 의존성 다운로드 과정은 캐시를 사용해 건너뜁니다!

# gradlew에 실행 권한 부여
RUN chmod +x gradlew

# 의존성 캐시 다운로드 (소스코드 없이 의존성만 미리 받아둠)
RUN ./gradlew dependencies --no-daemon

# 전체 소스 복사 후 빌드
COPY . .

# (만약을 위해) 다시 gradlew에 실행 권한 부여
RUN chmod +x gradlew

# Spring Boot JAR 빌드 (테스트는 생략하여 빌드 속도 향상)
RUN ./gradlew clean bootJar -x test --no-daemon


# From을 N개를 쓰면 최종 베이스이미지로 빌드됨.
# 실행환경에서는 실행만 됨., 그레들이나 메이븐 이런게 없음
# 스프링은 Jar만 있으면 실행 O
# 2단계: 실행 환경
FROM amazoncorretto:21-alpine
# Java 17 실행 환경만 있는 초경량 이미지 사용

WORKDIR /app 
# 작업 공간은 /app

# 실행 시 사용될 프로파일 지정 (기본값: dev)
ARG PROFILE=dev 
# 빌드 시 `--build-arg`로 값을 받을 수 있는 변수 선언
ENV SPRING_PROFILES_ACTIVE=${PROFILE} 
# 컨테이너 내부 환경변수 설정

# 'builder' 스테이지에서 빌드된 JAR 파일만 복사해와서 app.jar로 이름 변경
COPY --from=builder /app/build/libs/*.jar app.jar

# 8080 포트를 외부에 노출할 것이라고 명시
EXPOSE 8080

# 컨테이너 시작 시 실행될 최종 명령어
ENTRYPOINT ["sh", "-c", "java -jar -Dspring.profiles.active=${SPRING_PROFILES_ACTIVE} app.jar"]
```

### 2. Docker 이미지 빌드 및 실행
```
#dev
docker build --build-arg PROFILE=dev -t spring-docker:dev .
docker run --name spring-docker-dev -e SPRING_PROFILES_ACTIVE=dev -p 8000:8080 -d spring-docker:dev

#prod
docker build --build-arg PROFILE=prod -t spring-docker:prod .
docker run --name spring-docker-prod -e SPRING_PROFILES_ACTIVE=prod -p 8000:8080 -d spring-docker:prod
```

