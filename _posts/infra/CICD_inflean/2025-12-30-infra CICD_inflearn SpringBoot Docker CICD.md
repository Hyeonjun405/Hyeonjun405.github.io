---
title: 04 SpringBoot Docker CICD
date: 2025-12-30 10:00:00 +09:00
categories: [infra, CICDInflearn]
tags: [ infra, CICDInflearn ]
---

## 1. SpringBoot + ECR + GitHub Actions
### 1. 전체적인 흐름
  - ![내 그림](assets/img/infra/CICD_inflean/04/4흐름.png "이미지")
  - 개발자가 푸쉬를 하면,
  - Github Actions에서 캐치를 해서,
  - ECR에 접근해서 이미지 빌드 및 저장
  - SSH로 EC2에 접근하여
  - 기존 도커 파일 지우고 새로 run처리  



### 2. EC2에 도커설치
  ```
    sudo apt-get update && \
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
    sudo apt-key fingerprint 0EBFCD88 && \
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
    sudo apt-get update && \
    sudo apt-get install -y docker-ce && \
    sudo usermod -aG docker ubuntu && \
    sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
    sudo chmod +x /usr/local/bin/docker-compose && \
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
  ```

### 3. GitHub Actions용 사용자등록
#### 1. note
 - GitHub Actions가 ECR에 접근을 해야하기때문에 별도 사용자 등록이 필요함

#### 2. 등록(기사용중인 권한이 있으면 그대로 사용)
  - ![내 그림](assets/img/infra/CICD_inflean/04/4깃허브액션정책추가.png "이미지")
  - ![내 그림](assets/img/infra/CICD_inflean/04/4깃허브액션정책추가2.png "이미지")
  - ![내 그림](assets/img/infra/CICD_inflean/04/4깃허브액션정책추가3.png "이미지")

### 4. ECR 생성
  - 기존 패턴 동일(특이사항X)

### 4. EC2 - ECR
#### 1. EC2의 ECR 인증 (공식 사이트 제공)
  ```
   # 이 프로그램을 사용하라고 권장됨
   sudo apt install amazon-ecr-credential-helper 
  ```

#### 2. config.json 생성(기준파일)
 - `~` 경로에서 `.docker`라는 폴더 만들고, `config.json` 생성
 ```
    {
    	"credsStore": "ecr-login"
    }
 ```

#### 3. IAM 에서 역할부여
 - config.json이 있으면, iam에서 특정 역할을 부여해둘 경우 ECR접근이 가능하게됨(공식사이트 제공)
 - 부여 방법


### 5. 깃허브액션 설정
#### 1. deploy.yaml
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
  
        - name: JDK 21버전 설치
          uses: actions/setup-java@v4
          with:
            distribution: temurin
            java-version: 21
  
        - name: application.yml 파일 만들기
          run: echo "${{ secrets.APPLICATION_PROPERTIES }}" > ./src/main/resources/application.yml
  
        - name: 테스트 및 빌드하기
          run: |
            chmod +x ./gradlew
            ./gradlew clean build
  
        - name: AWS Resource에 접근할 수 있게 AWS credentials 설정
          uses: aws-actions/configure-aws-credentials@v4
          with:
            aws-region: ap-northeast-2
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  
        - name: ECR에 로그인하기
          id: login-ecr
          uses: aws-actions/amazon-ecr-login@v2
  
        - name: Docker 이미지 생성
          run: docker build -t study-server .
  
        - name: Docker 이미지에 Tag 붙이기
          run: docker tag study-server ${{ steps.login-ecr.outputs.registry }}/study-server:latest
          # 로그인(ECR에 로그인하기)하면 결과값이 나오는데 steps.login-ecr의 outputs.registry 값!
  
        - name: ECR에 Docker 이미지 Push하기
          run: docker push ${{ steps.login-ecr.outputs.registry }}/study-server:latest
  
        - name: SSH로 EC2에 접속하기
          uses: appleboy/ssh-action@v1.0.3
          with:
            host: ${{ secrets.EC2_HOST }}
            username: ${{ secrets.EC2_USERNAME }}
            key: ${{ secrets.EC2_PRIVATE_KEY }}
            script_stop: true
            script: |
              # 기존 서버가 있다면 정지
              docker stop study-server || true 
              # 기존 서버 서버 삭제
              docker rm study-server || true 
              # 신규 서버 데이터 Pull
              docker pull ${{ steps.login-ecr.outputs.registry }}/study-server:latest 
              # 신규 서버 run
              docker run -d --name study-server -p 8080:8080 ${{ steps.login-ecr.outputs.registry }}/study-server:latest 
  ```

## 2. SpringBoot + ECR + S3 + CodeDeploy + GitHub Actions
### 1. 전체 흐름
  - ![내 그림](assets/img/infra/CICD_inflean/04/4흐름2.png "이미지")
  - 개발자가 푸쉬를 하면,
  - Github Actions에서 캐치를 해서,
  - ECR에 접근해서 이미지 빌드 및 저장
  - codeDeploy 사용에 필요한 (appspec.yml, scripts) 파일 압축 
  - 압축 파일 S3에 업로드(도커를 사용하기 때문에 빌드파일 압축안해도됨) 
  - github Actions에서 CodeDeploy 작업 요청
  - codeDeploy는 S3에서 파일을 받아서 
  - EC2에서 도커에서 기존 파일 지우고 새로받아서 컨테이너 띄우도록 요청함 

### 2. 깃허브 액션 파일
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
  
        - name: AWS Resource에 접근할 수 있게 AWS credentials 설정
          uses: aws-actions/configure-aws-credentials@v4
          with:
            aws-region: ap-northeast-2
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  
        - name: ECR에 로그인하기
          id: login-ecr
          uses: aws-actions/amazon-ecr-login@v2
        - name: Docker 이미지 생성
          run: docker build -t study-server .
  
        - name: Docker 이미지에 Tag 붙이기
          run: docker tag study-server ${{ steps.login-ecr.outputs.registry }}/study-server:latest
  
        - name: ECR에 Docker 이미지 Push하기
          run: docker push ${{ steps.login-ecr.outputs.registry }}/study-server:latest
  
        - name: 압축하기
          run: tar -czvf $GITHUB_SHA.tar.gz appspec.yml scripts
  
        - name: S3에 프로젝트 폴더 업로드하기
          run: aws s3 cp --region ap-northeast-2 ./$GITHUB_SHA.tar.gz s3://study-server-bucket/$GITHUB_SHA.tar.gz
  
        - name: Code Deploy를 활용해 EC2에 프로젝트 코드 배포
          run: aws deploy create-deployment
            --application-name study-server
            --deployment-config-name CodeDeployDefault.AllAtOnce
            --deployment-group-name Production
            --s3-location bucket=study-server,bundleType=tgz,key=$GITHUB_SHA.tar.gz
  ```

### 3. appspec.yml(변동X)
  ```
    version: 0.0
    os: linux
    
    files:
      # CodeDeploy가 S3로부터 가져온 파일 중 destination으로 이동시킬 대상을 지정한다.
      # / 이라고 지정하면 S3로부터 가져온 전체 파일을 뜻한다.
      - source: /
        # CodeDeploy가 S3로부터 가져온 파일을 EC2의 어떤 경로에 저장할 지 지정한다.
        destination: /home/ubuntu/study-server
    
    permissions:
      - object: /
        owner: ubuntu
        group: ubuntu
    
    hooks:
      ApplicationStart:
        - location: scripts/start-server.sh
          timeout: 60
          runas: ubuntu
  ```

### 4. start.sh
  ```
  #!/bin/bash
  
  echo "--------------- 서버 배포 시작 -----------------"
  docker stop study-server || true # 도커 기존작업 정지
  docker rm study-server || true # 삭제
  docker pull {ECR Repository 주소}/study-server:latest #ECR에서 URL 확인하고 최신버전을 pull
  docker run -d --name study-server -p 8080:8080 {ECR Repository 주소}/study-server:lates # 실행!
  echo "--------------- 서버 배포 끝 -----------------"
  ```
