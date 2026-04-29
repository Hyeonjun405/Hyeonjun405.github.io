---
title: 02 Docker
date: 2026-04-29 09:00:00 +09:00
categories: [Sparta, SpartaWIL08]
tags: [ Java, Spring Framework ]
---

## 1. Note
### 1. Note
 - 다시 복습!

## 2. Docker
### 1. Docker란?
- 컨테이너 기술을 활용하여 애플리케이션을 실행하는 플랫폼
- 가상 머신과 비교하여 가볍고 빠른 환경 제공

### 2. Docker의 핵심 개념
- 이미지 (Image): 실행 가능한 애플리케이션 패키지
- 컨테이너 (Container): 실행 중인 독립된 환경
- Dockerfile: 컨테이너를 정의하는 설정 파일
- Docker Hub: 공개된 Docker 이미지를 저장하는 레파지토리

### 3. Docker Architecture
![내 그림](assets/img/sparta/wil08/docker.png "이미지")
- Docker Client (CLI, REST API, UI)
  - 사용자가 Docker와 상호작용할 수 있도록 명령을 전달하는 인터페이스
  - CLI(Command Line Interface) 또는 REST API를 사용하여 컨테이너를 실행하거나 관리할 수 있음
- Docker Daemon (dockerd)
  - Docker의 핵심 엔진으로, 컨테이너와 이미지를 관리하는 역할
  - Clinet 로 부터 명령을 받아 컨테이너 생성, 네트워크 관리, 스토리지 할당 등을 처리
- Docker Images
  - 컨테이너를 실행하는 데 필요한 파일 시스템과 애플리케이션 코드가 포함된 템플릿
  - Layer 기반의 구조로, 같은 이미지에서 여러 개의 컨테이너 생성 가능
- Containers
  - 독립적으로 실행되는 애플리케이션 인스턴스
  - 각 컨테이너는 자체 파일 시스템과 네트워크 인터페이스를 가지며, 다른 컨테이너와 격리됨
  - `docker run` 명령으로 생성 가능
- Docker Registry (Docker Hub, Private Registry)
  - Docker 이미지를 저장하고 배포하는 서비스
  - Docker Hub(공개), Amazon ECR, Google GCR 등의 사설 레지스트리를 사용 가능
  - `docker pull` 또는 `docker push` 명령으로 이미지를 업로드/다운로드
- Storage & Networking
  - Storage: 컨테이너의 데이터를 지속적으로 저장하기 위해 Volumes, Bind Mounts, Tmpfs 사용
  - Networking: 브리지(Bridge), 호스트(Host), 오버레이(Overlay) 등의 네트워크 드라이버를 통해 컨테이너 간 통신을 제공

### 4. Docker 기본명령어
  ```
  # 현재 시스템에서 사용 가능한 모든 Docker 이미지 목록 확인
  $ docker images
  
  # 특정 이미지 다운로드
  $ docker pull <이미지 이름>
  
  # 이미지 정보 확인
  $ docker inspect <이미지 이름>
  
  # 이미지 삭제
  $ docker rmi <이미지 이름>
  ```

### 5. 컨테이너 관련
  ```
  # 컨테이너 실행 (제일 중요)
  docker run -d -p 8080:80 nginx
  
  # 실행 중 컨테이너 조회
  docker ps
  # 전체 컨테이너 조회
  docker ps -a
  # 컨테이너 정지
  docker stop <컨테이너ID>
  
  # 컨테이너 재시작
  docker start <컨테이너ID>
  
  # 컨테이너 삭제
  docker rm <컨테이너ID>
  ```


## 3. Docker Compose
### 1. Docker Compose
- 여러 개의 컨테이너를 하나의 서비스로 관리하는 도구
- YAML 파일로 컨테이너 정의 및 실행 가능
- 개발 및 배포 환경에서 다중 컨테이너 애플리케이션을 쉽게 설정할 수 있습니다

### 2. 기본구조
  ```
  version: "3.8"
  
  services: # 컨테이너 정의 영역
    app:
      build: .
      ports:
        - "8080:8080"
      depends_on:
        - db
  
    db:
      image: mysql:8
      environment:
        MYSQL_ROOT_PASSWORD: root
        MYSQL_DATABASE: test
      ports:
        - "3306:3306"
  ```

### 3. 주요 명령어
  ```
  # 전체 실행(최신버전은 하이픈이 없음)
  docker-compose up
  docker compose up
  
  # 백그라운드 실행
  docker compose up -d
  
  # 종료
  docker compose down
  
  # 로그 확인
  docker compose logs
  docker compose logs app # 특정서비스 실행
  
  # 재빌드
  docker compose up --build
  ```

