---
title: 02 Docker CLI
date: 2025-12-03 10:00:00 +09:00
categories: [infra, dockerInflearn]
tags: [ infra, dockerInflearn ]
---

## 1. DockerHub
 - https://hub.docker.com/
 - 도커 이미지를 저장/다운로드 할 수 있는 저장소 역할하는 곳.
 - pull로 당기면 도커허브에서 다운로드됨.

## 2. image
#### 1. image 다운
  ```
  # docker pull 이미지명 (최신버전 다운로드)
  $ docker pull nginx 
  
  # docker pull 이미지명:태그명 (특정 버전 다운로드)
  $ docker pull nginx:stable-perl
  ```

#### 2. image List
 ```
 # 설치된 도커 이미지 조회
 $ docker image ls 
 ```
#### 3. image remove
 ```
 # 특정 이미지 삭제 (컨테이너 사용중이 XX 이미지)
 $ docker image rm [이미지 ID 또는 이미지명]
 
 # 중단된 컨테이너 이미지 삭제 (기존에 사용중인 이미지) -> 종료된 이미지 
 $ docker image rm -f [이미지 ID 또는 이미지명]
 
 # 컨테이너에서 사용하고 있지 않은 이미지만 전체 삭제
 $ docker image rm $(docker images -q)

 # 컨테이너에서 사용하고 있는 이미지를 포함해서 전체 이미지 삭제
 $ docker image rm -f $(docker images -q)
 ```

## 3. container
### 1. container 생성
  ```
  # docker create 이미지명[:태그명]
  # 이미지를 가지고 컨테이너를 생성
  $ docker create nginx
  $ docker ps -a # 모든 컨테이너 조회
  ```

### 2. contianer 조회와 시작
 ```
  # docker start 컨테이너명[또는 컨테이너 ID]
  $ docker start 컨테이너명[또는 컨테이너 ID]
  $ docker ps # 실행중인 컨테이너 조회
 ```

### 3. docker run
  ```
  # 이미지를 바탕으로 컨테이너를 생성한 뒤, 컨테이너를 실행까지 시킨다
  # docker run 이미지명[:태그명]
  $ docker run nginx # 포그라운드에서 실행 
  $ docker run -d nginx # 백그라운드에서 실행
  ```

### 4. 컨테이너 관리
  ```
  # docker run -d --name [컨테이너 이름] 이미지명[:태그명]
  $ docker run -d --name my-web-server nginx
   ```

### 3. 포트 연결
  ```
  # 컨테이너에 접근하기 위해서는 host 컴퓨터를 통해서 들어가야됨.
  # 그래서 호스트 컴퓨터에 특정 포트를 설정하여 컨테이너포트로 연결해주는 작업이 필요. 
  
  # docker run -d -p [호스트 포트]:[컨테이너 포트] 이미지명[:태그명]
  $ docker run -d -p 4000:80 nginx
  ```

## 4. container 관리
### 1. docker ps
  ```
  # 실행 중인 컨테이너들만 조회
  $ docker ps
  
  # 모든 컨테이너 조회 (작동 중인 컨테이너 + 작동을 멈춘 컨테이너)
  $ docker ps -a
  ```

### 2. docker stop
  ```
  $ docker stop 컨테이너명[또는 컨테이너 ID]
  $ docker kill 컨테이너명[또는 컨테이너 ID]
  ```

### 3. docker 삭제
  ```
  # 중지되어 있는 특정 컨테이너 삭제
  $ docker rm 컨테이너명[또는 컨테이너 ID]
  
  #실행되고 있는 특정 컨테이너 삭제
  $ docker rm -f 컨테이너명[또는 컨테이너 ID]

  #중지되어 있는 모든 컨테이너 삭제
  $ docker rm $(docker ps -qa)
  
  #실행되고 있는 모든 컨테이너 삭제
  $ docker rm -f $(docker ps -qa)
  ```

## 5. docker 운영
### 1. 로그
  ```
  # docker logs [컨테이너 ID 또는 컨테이너명]
  $ docker logs [nginx가 실행되고 있는 컨테이너 ID]
  
  # dokcer logs --tail [로그 끝부터 표시할 줄 수] [컨테이너 ID 또는 컨테이너명]
  $ dokcer logs --tail 10 [컨테이너 ID 또는 컨테이너명]
  ```

### 2. 로그 구분 조회
  ```
  # docker logs -f [컨테이너 ID 또는 컨테이너명]
  # Nginx의 컨테이너에 실시간으로 쌓이는 로그 확인하기
    $ docker logs -f
  
  # 기존 로그는 조회하지 않기 + 생성되는 로그를 실시간으로 보고 싶은 경우
  # 0을 넣으면 마지막 저장된 로그가 X
  $ docker logs --tail 0 -f [컨테이너 ID 또는 컨테이너명] 
  ```

### 3. 컨테이너 내부에 접근
 ```
  $ docker run -d nginx
  $ docker exec -it [Nginx가 실행되고 있는 컨테이너 ID] bash
 
  $ ls # 컨테이너 내부 파일 조회
  $ cd /etc/nginx
  $ cat nginx.conf
 ```
