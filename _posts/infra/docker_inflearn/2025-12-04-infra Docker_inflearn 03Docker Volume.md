---
title: 03 Docker Volume
date: 2025-12-04 10:00:00 +09:00
categories: [infra, dockerInflearn]
tags: [ infra, dockerInflearn ]
---

## 1. Docker Volume
### 1. 컨테이너의 문제점
 - 도커는 컨테이너를 수정할때 특정 부분만 수정하지 않고, 새로운 컨테이너를 만들어서 교체하는 방식을 채택함.
 - 기존 컨테이너를 새로운 컨테이너로 교체하면, 기존 컨테이너 내부에 AP, DB등에서 기록한 데이터도 같이 삭제됨.
 - 그래서 도커는 컨테이너 내부에 저장된 데이터가 삭제되면 안 되는 경우에는 볼륨(Volume)이라는 개념을 제공함.

### 2. 도커볼륨 
 - 도커의 볼륨(Volume)이란 도커 컨테이너에서 데이터를 영속적으로 저장하기 위한 방법
 - 볼륨(Volume)은 컨테이너 자체의 저장 공간을 사용하지 않고, 호스트 자체의 저장 공간을 공유해서 사용하는 형태

### 3. 볼륨 명령어
  ```
  $ docker run -v [호스트의 디렉토리 절대경로]:[컨테이너의 디렉토리 절대경로] [이미지명]:[태그명]
  ```

## 2. 실습
### 1. Image 확인
 ![내 그림](assets/img/infra/docker_inflearn/도커허브 캡처.png "이미지")
 - 도커 허브에서 필요한 이미지를 확인하고, 관련된 설정을 확인 가능함.
 - MYSQL의 경우에는 기본설정을 위한 명령어가 존재함.

### 2. Image 실행 확인
  ```
   -- 이렇게 진행할 경우 초기비밀번호 password123, root사용자, 포트번호 3306으로 생성됨.
    docker run -e MYSQL_ROOT_PASSWORD=password123 -p 3306:3306 -d mysql
     -- 도커 컨테이너 ID를 확인하고
    docekr ps -a
   -- 도커에 진입해서
    docker exec -it b19d0d bash   
   -- 실제로 패스워드 등록 되어있는거 확인 가능 
    echo $MYSQL_ROOT_PASSWORD
  ```
 - 이 상태는 데이터를 저장해도 DB데이터가 컨테이너 내부에 들어있기 때문에 교체하면 바로 삭제되는 상황.
 
### 3. 도커 볼륨 적용
 ```
  docker run -e MYSQL_ROOT_PASSWORD=password123 -d -p 3306:3306 -v C:\docker\mysql:/var/lib/mysql mysql
 ```
 - 3306:3306까지가 기본 실행 그 이후가 볼륨 설정
 - C:\docker\mysql가 로컬 호스트에서 설정한 경로
 - /var/lib/mysql가 mysql 설치된 위치
 - 도커허브 내에서 설명서 보면 나와있음.
  ![내 그림](assets/img/infra/docker_inflearn/마이SQL설치위치.png "이미지")
 
## 3. 노트
### 1. 볼륨 작업
 - 기존에 데이터가 있거나 상황이 있는 경우 설치 자체가 문제가 있을 수 있음.
 - 볼륨을 쓰는데 호스트에 기존 mysql 기본 데이터가 기존에 잔류하고 있다면, 생성이 안될 수도 있음.
 - 호스트 OS의 우선순위가 높아서 컨테이너에서 만들고 나서 기존 Host의 데이터가 있따면 그걸로 덮어씌울수 있음.

