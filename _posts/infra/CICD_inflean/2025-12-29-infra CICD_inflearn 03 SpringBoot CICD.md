---
title: 03 SpringBoot CICD
date: 2025-12-29 10:00:00 +09:00
categories: [infra, CICDInflearn]
tags: [ infra, CICDInflearn ]
---

## 1. SpringBoot CI/CD
### 1. SpringBoot CI/CD
  ```
   Github -(push)-> Github Action -(ssh)-> AWS EC2  
  ```
  - 개발자가 git에 커밋을 하면, 
  - Git에서 push 이벤트를 체크하여,
  - Github Action에서 SSH로 AWS EC2에 접근
  - 설정된 방법으로 반영 처리함.

### 2. note
  - AWS에서 ssh로 반영이 가능하도록 셋팅이 필요함.
  - 굳이 AWS가 아니여도 다른 방법으로 반영한다면 조절 가능.
  - 단지, EC2에서 한다는 전제하에 셋팅
   
### 3. 전체 흐름
 - 스프링부트 프로젝트 셋팅

## 2. 스프링부트 프로젝트 + 직접 실행
### 1. 스프링프로젝트 생성/작업
 - 프로젝트명 : study-server
 - EC2 명 : study-server
 - 빌드후에 접근 가능한지 확인 필요.

### 2. SSH로 직접 스프링 실행
 - deploy.yml
  ```
    name: Deploy To EC2

    on:
      push:
        branches:
          - main
    
    jobs:
      deploy:
        runs-on: ubuntu-latest
        steps:
          - name: SSH로 EC2에 접속하기
            uses: appleboy/ssh-action@v1.0.3 # 라이브러리명
            with:
              host: 호스트값 # SSH 접근주소 / IP주소 등
              username: 유저네임 # SSH 접근 유저이름
              key: 키값 # SSH 접근 키페어
              script_stop: true # 아래 script 중 실패하는 명령이 하나라도 있으면 실패로 처리
              script: |
                cd /home/ubuntu/study-server # 프로젝트 저장공간.
                git pull origin main # 파일 최신화
                ./gradlew clean build # gradlew 빌드
                sudo fuser -k -n tcp 8080 || true  # 기존에 켜져있던 서버 제거
                nohup java -jar build/libs/*SNAPSHOT.jar > ./output.log 2>&1 & # 로그 남기고, jar 실행
  ```

### 3. 결과
 - 직접 빌드하고 실행하는 경우에는 작업이 완료됨.
 - 개발자가 직접 타이핑하는 경우에는 이 패턴으로 작업이 가능. 

### 4. 시크릿 키값을 AP에 생성
 - 프로퍼티스 등을 보안상의 이유로 커밋을 안하는 경우, AP실행시에 파일이 없는 상태로 있게됨
 - 따라서 시크릿에 값을 등록해두고, AP가 실행되기전 프로퍼티스 파일을 생성해버림.
 - deploy.yml
   ```
    name: SSH로 EC2에 접속하기
     uses: appleboy/ssh-action@v1.0.3
     env:
       APPLICATION_PROPERTIES: APPLICATION_PROPERTIES 시크릿 값 ${{ secrets.APPLICATION_PROPERTIES }} # 등록한 프로퍼티스값 로드  
     with:
       host: 호스트값  
       username: 유저네임 
       key: 키값  
       env: APPLICATION_PROPERTIES # env에서 로드한 값을 변수에 담음.
       script_stop: true
       script: |
         cd /home/ubuntu/study-server 
         rm -rf src/main/resources/application.yml #기존 yml삭제
         git pull origin main
         echo "$APPLICATION_PROPERTIES" > src/main/resources/application.yml # 여기서 프로퍼티스 파일을 생성해버림.
         ./gradlew clean build
         sudo fuser -k -n tcp 8080 || true  
         nohup java -jar build/libs/*SNAPSHOT.jar > ./output.log 2>&1 &
   ```

## 3. SCP + EC2 
### 1. Note
 - 깃 허브액션에서 테스트를 진행해서 문제가 없으면 
 - 빌드 파일을 EC2로 넘기는 방법(SCP)으로 진행함.

### 2. deploy.yml
 ```
 steps:  
  - name: Github Repository 파일 불러오기
    uses: actions/checkout@v4

  - name: JDK 21버전 설치
    uses: actions/setup-java@v4
    with:
      distribution: temurin
      java-version: 21

  - name: application.yml 파일 만들기
    run: echo "${{ secretsE.APPLICATION_PROPERTIS }}" > ./src/main/resources/application.yml

  - name: 테스트 및 빌드하기
    run: |
      chmod +x ./gradlew
      ./gradlew clean build

  - name: 빌드된 파일 이름 변경하기
    run: mv ./build/libs/*SNAPSHOT.jar ./project.jar

  - name: SCP로 EC2에 빌드된 파일 전송하기
    uses: appleboy/scp-action@v0.1.7
    with:
      host: 호스트명 # 호스트명
      username: 유저이름 # 유저이름
      key: 키값 # SSH 접근 키페어
      source: project.jar # 프로젝트 파일
      target: /home/ubuntu/study-server/tobe # 경로
   
  - name: SSH로 EC2에 접속하기
    uses: appleboy/ssh-action@v1.0.3
    with:
      host: 호스트명 # 호스트명
      username: 유저이름 # 유저이름
      key: 키값 # SSH 접근 키페어
      script_stop: true
      script: |
         # 기존 파일삭제
         rm -rf /home/ubuntu/study-server/current 
         # 동일한 디렉토리 생성
         mkdir /home/ubuntu/study-server/current
         # 옮겨진 파일을 이동함.
         mv /home/ubuntu/study-server/tobe/project.jar /home/ubuntu/study-server/current/project.jar
         # 실행 디렉토리 접근
         cd /home/ubuntu/study-server/current
         # 기존 서버 삭제
         sudo fuser -k -n tcp 8080 || true
         nohup java -jar project.jar > ./output.log 2>&1 & 
         rm -rf /home/ubuntu/study-server/tobe
 ```

## 4. AWS CodeDeploy 패턴
### 1. codeDeploy
 - ![내 그림](assets/img/infra/CICD_inflean/03/3codedeply 아키텍처.png "이미지")
 - 이미 만들어진 애플리케이션을 서버나 서비스에 안전하게 배포해주는 AWS의 배포 전용 서비스
 - CD(배포)에 특화된 서비스
 - CodeDeploy는 “배포 절차와 순서”를 책임짐
   - 어느 서버에 배포할지
   - 어떤 순서로 배포할지
   - 배포 중 실패 시 어떻게 처리할지
   - 배포 전/후에 어떤 스크립트를 실행할지

### 2. Note
- EC2에 해당 빌드파일을 두지 않고,
- Github Actions에서 빌드를 하고, AWS S3에 빌드파일을 저장함.
- Github ACtions에서 직접 EC2에서 가져오라고 하지 않고,
- AWS CodeDeploy에 반영을 요청. (역할이 분리됨)
- codeDeploy에서 EC2에게 S3에서 빌드파일을 받아서 배포하라고 요청함.
 
### 3. codeDeploy 설정
#### 1. IAM 역할 부여
 - 역할
   - AWS에서는 기본 권한 만 부여되어있고,
   - 서비스마다 간섭을 못하게 놓여있음.
   - 그래서 다른쪽에 접근을 하려면 할 수 있는 역할을 줘야함.
   - 무엇인가 할 수 있는 역할임, 어떻게 보면 권한에 가까움,
 - 기존에는 사용자만 등록헀고, 지금은 역할을 부여해야함.
 - 등록과정
   - ![내 그림](assets/img/infra/CICD_inflean/03/3iam역할생성.png "이미지")
   - ![내 그림](assets/img/infra/CICD_inflean/03/3iam역할생성2.png "이미지")
   - ![내 그림](assets/img/infra/CICD_inflean/03/3iam역할생성3.png "이미지")

#### 2. CodeDeploy
  - ![내 그림](assets/img/infra/CICD_inflean/03/3codeDeploy생성.png "이미지")
  - ![내 그림](assets/img/infra/CICD_inflean/03/3codeDeploy생성2.png "이미지")
  - ![내 그림](assets/img/infra/CICD_inflean/03/3codeDeploy생성3.png "이미지")
  - ![내 그림](assets/img/infra/CICD_inflean/03/3codeDeploy생성4.png "이미지")

### 4. EC2 역할 부여
  - 역할 생성
    - ![내 그림](assets/img/infra/CICD_inflean/03/3EC2역할부여.png "이미지")
    - ![내 그림](assets/img/infra/CICD_inflean/03/3EC2역할부여2.png "이미지")
    - ![내 그림](assets/img/infra/CICD_inflean/03/3EC2역할부여3.png "이미지")
    - ![내 그림](assets/img/infra/CICD_inflean/03/3EC2역할부여4.png "이미지")
  - EC2에 역할 부여(생성한 역할 부여함)
    - ![내 그림](assets/img/infra/CICD_inflean/03/3EC2역할부여5.png "이미지")

  
### 5. EC2에서 CodeDeploy 설치
#### 1. EC2에서 설치함 (버전에 따라 조금씩 다름)
  ```
    sudo apt update && \
    sudo apt install -y ruby-full wget && \
    cd /home/ubuntu && \
    wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install && \
    chmod +x ./install && \
    sudo ./install auto
  ```
  - ![내 그림](assets/img/infra/CICD_inflean/03/3EC2역할부여6.png "이미지")

### 6. Github Actions 권한(사용자) 등록
#### 1. Note
 - 사용자는 외부 시스템이 내부 시스템을 사용할때
 - AWS 내부시스템이 AWS시스템 사용할떄 역할을 부여함.

#### 2. 부여
  - 키생성
    - ![내 그림](assets/img/infra/CICD_inflean/03/3githubActions 권한부여.png "이미지")
    - ![내 그림](assets/img/infra/CICD_inflean/03/3githubActions 권한부여2.png "이미지")
    - ![내 그림](assets/img/infra/CICD_inflean/03/3githubActions 권한부여3.png "이미지")
    - ![내 그림](assets/img/infra/CICD_inflean/03/3githubActions 권한부여4.png "이미지")
    - ![내 그림](assets/img/infra/CICD_inflean/03/3githubActions 권한부여5.png "이미지")
  - 만들어진 키값을 깃 ACCESS에 등록
    - ![내 그림](assets/img/infra/CICD_inflean/03/3githubActions 권한부여6.png "이미지")

### 7. S3 생성
 - 기존 패턴 동일

### 8. Project 셋팅
#### 1. appspec.yml(코드디플로이가 실행)
 - 코드디플로이가 실행할때 기준이 되는 파일로 최상단에 위치
 - `S3에서 파일을 다운받고, start-server.sh를 실행한다`라는 패턴
 - 소스
  ```
    version: 0.0
    os: linux
    
    files:
      - source: / # S3에 저장한 전체 파일
        destination: /home/ubuntu/study-server  # EC2의 어떤 경로에 저장할 지 지정
    
    # 작업을 할때 가진 권한
    permissions:
      - object: /
        owner: ubuntu
        group: ubuntu
    
    hooks:
      ApplicationStart: # AP가 시작할때에 어떤 작업?
        - location: scripts/start-server.sh # 프로젝트에서의 경로의 쉘파일을 진행
          timeout: 60 # 스크립트를 실행하는데 걸리는 소요시간
          runas: ubuntu # 우분투라는 유저로 실행
  ```

#### 2. start-server.sh(코드디플로이가 실행)
  ```
  #!/bin/bash
  
  echo "--------------- 서버 배포 시작 -----------------"
  cd /home/ubuntu/study-server # 여기들어가서
  sudo fuser -k -n tcp 8080 || true # 기존 실행 AP 죽이기
  nohup java -jar project.jar > ./output.log 2>&1 & # 실행
  echo "--------------- 서버 배포 끝 -----------------"
  ```

#### 3. deploy.yml
  ```
  name: Deploy To EC2

  on:
    push:
      branches:
        - main
  
  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - name: Github Repository 파일 불러오기
          uses: actions/checkout@v4
  
        - name: JDK 17버전 설치
          uses: actions/setup-java@v4
          with:
            distribution: temurin
            java-version: 17
  
        - name: application.yml 파일 만들기
          run: echo "${{ secrets.APPLICATION_PROPERTIES }}" > ./src/main/resources/application.yml
  
        - name: 테스트 및 빌드하기
          run: ./gradlew clean build
  
        - name: 빌드된 파일 이름 변경하기
          run: mv ./build/libs/*SNAPSHOT.jar ./project.jar
  
        - name: 압축하기
          run: tar -czvf $GITHUB_SHA.tar.gz project.jar appspec.yml scripts
               #tar로 압축하고, 커밋명.tar.gz로 하는데 (project.jar appspec.yml scripts)파일이 들어있음.
               
        - name: AWS Resource에 접근할 수 있게 AWS credentials 설정
          uses: aws-actions/configure-aws-credentials@v4
          with:
            aws-region: ap-northeast-2
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  
        - name: S3에 프로젝트 폴더 업로드하기
          run: aws s3 cp --region ap-northeast-2 ./$GITHUB_SHA.tar.gz s3://study-server-bucket/$GITHUB_SHA.tar.gz
  
        - name: Code Deploy를 활용해 EC2에 프로젝트 코드 배포
          #aws deploy에서 작업하기떄문에 자동으로 s3버킷이됨.
          run: aws deploy create-deployment # CodeDeploy 의 그룹안에 배포 생성 버튼
            --application-name study-server # CodeDeploy의 study-server에 배포를 하겠다.
            --deployment-config-name CodeDeployDefault.AllAtOnce # 모든것을 한번에 함.(인스턴스가 여러개일경우 순차/다중 등 방법)
            --deployment-group-name Production # 그룹은 Production (생성했던 그룹)
            --s3-location bucket=study-server-bucket,bundleType=tgz,key=$GITHUB_SHA.tar.gz # AWS의 bucket이름
  ```
