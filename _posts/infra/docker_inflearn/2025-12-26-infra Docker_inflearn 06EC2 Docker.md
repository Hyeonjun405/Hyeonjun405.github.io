---
title: 06 Docker ECR
date: 2025-12-26 10:00:00 +09:00
categories: [infra, dockerInflearn]
tags: [ infra, dockerInflearn ]
---

## 1. EC2에 도커 환경구성
### 1. EC2에서 도커설치(정해진 명령어)
  ```
    sudo apt-get update && \
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
    sudo apt-key fingerprint 0EBFCD88 && \
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
    sudo apt-get update && \
    sudo apt-get install -y docker-ce && \
    sudo usermod -aG docker ubuntu && \
    newgrp docker && \
    sudo curl -L "https://github.com/docker/compose/releases/download/2.27.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
    sudo chmod +x /usr/local/bin/docker-compose && \
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
  ```

## 2. AWS ECR
### 1. AWS ECR
 - 도커 이미지를 보관하는 저장소
 - 로컬에서 만든 이미지나 CI 파이프라인에서 빌드된 이미지를 푸시해두고,
 - EC2, ECS, EKS(쿠버네티스), 배치 작업 등이 그 이미지를 pull 해서 실행

### 2. Docker Hub와의 차이
 - Docker Hub: 퍼블릭 중심, 계정 없이도 접근 가능
 - ECR: AWS 계정 단위의 프라이빗 레지스트리, IAM으로 접근 제어

### 3. ECR 동작 흐름
 - 개발자가 로컬에서 Dockerfile로 이미지 빌드
 - AWS ECR에 로그인 (aws ecr get-login-password …)
 - 이미지 태깅 (계정ID.dkr.ecr.region.amazonaws.com/리포지토리:태그)
 - docker push 로 ECR에 업로드
 - ECS/EKS/EC2가 해당 이미지를 pull 해서 컨테이너 실행

## 3. AWS ECR 활용
### 1. 로컬에 ECR 설치필요
 - 윈도우/우분투/맥 상이함.
 - 우분투 설치
   ```
    $ sudo apt install unzip
    $ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    $ unzip awscliv2.zip
    $ sudo ./aws/install
    $ aws --version # 잘 출력된다면 정상 설치된 상태  
   ```

### 2. IAM 생성
 - 접근을 위한 IAM 생성- 설정방법
   - ![내 그림](assets/img/infra/docker_inflearn/9iam생성.png "이미지")
 - 생성된 IAM 접근하여 엑세스키 생성
   - ![내 그림](assets/img/infra/docker_inflearn/9iam생성2.png "이미지")
   - ![내 그림](assets/img/infra/docker_inflearn/9iam생성3.png "이미지")
 - 생성된 Key값 보관 필요하고 별다른 설정 없음 

### 3. 컴퓨터 내 AWS ECR 설정(윈도우/우분투 동일)
 ```
  $ aws configure
  AWS Access Key ID [None]: <IAM에서 발급한 Key id>
  AWS Secret Access Key [None]: <IAM에서 발급한 Secret Access Key>
  Default region name [None]: ap-northeast-2 #서울 지역
  Default output format [None]:
 ```

### 4. ECR 생성 
 - ECR 접근
   - ![내 그림](assets/img/infra/docker_inflearn/9ECR생성.png "이미지")
 - ECR 생성
   - 1 레파지토리에는 1이미지가 들어가고 여러버전이 들어있음.
   - 여러 이미지를 사용할 경우 여러 레파지토리 생성필요
   - ![내 그림](assets/img/infra/docker_inflearn/9ECR생성2.png "이미지")
 - 바로 생성처리 (별다른 설정없음)

### 5. ECR 이미지생성
 - ECR 이미지 생성 명령어
   - ![내 그림](assets/img/infra/docker_inflearn/9ECR이미지생성.png "이미지")
 - Window랑 macOS / Linux랑 상이함.
 
### 6. 실행
 - 실행은 기존 Dokcer에서 관리 및 사용하는 패턴 그대로 사용하게됨.
 - 가장 초기에는 오류가 발생함.
 - 그것은 기본 설정이 기존 dokcerHub로 접근하기떄문,
 - 따라서 명령어 내역에서 ECR을 접근 후에 사용 필요
