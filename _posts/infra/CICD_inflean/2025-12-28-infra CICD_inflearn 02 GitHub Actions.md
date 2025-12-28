---
title: 02 GitHubActions
date: 2025-12-28 10:00:00 +09:00
categories: [infra, CICDInflearn]
tags: [ infra, CICDInflearn ]
---

## 1. GitHub Actions
### 1. Note
  - GitHub Action -> GitHub Actions
  - https://docs.github.com/ko/actions

## 2. GitHun Actions 기본형
### 1. 패키지 구조
- `.github/workflows`가 기본 디렉토리
- 오타나 이름이 다를 경우 진행X
- 최상위 폴더 구조에 생성

### 2. deploy.yaml
 - 파일명 변경가능함, 변경하면 잡이 나뉨
 - 파일 내용
 ```
  # Workflow의 이름
  # Workflow : 하나의 yml 파일을 하나의 Workflow
  name: Github Actions 실행시켜보기

  # Event : 실행되는 시점을 설정
  # main이라는 브랜치에 push 될 때 아래 Workflow를 실행
  on:
    push:
      branches:
        - main

  # 하나의 Workflow는 1개 이상의 Job으로 구성
  # 여러 Job은 기본적으로 병렬적으로 수행
  jobs: #Job의 내역
    
    My-Deploy-Job: # Job을 식별하기 위한 이름
      runs-on: ubuntu-latest # Github Actions를 실행시킬 서버 종류 선택
      
      # Step : 특정 작업을 수행하는 가장 작은 단위
      # Job은 여러 Step들로 구성되어 있음
      steps:
        - name: Hello World 찍기 # Step에 이름 붙이는 기능
          run: echo "Hello World" # 실행시킬 명령어 작성 
 ```

### 3. 결과물 확인
  - ![내 그림](assets/img/infra/CICD_inflean/02/02깃허브액션결과.png "이미지")
  - ![내 그림](assets/img/infra/CICD_inflean/02/02깃허브액션결과2.png "이미지")
  - ![내 그림](assets/img/infra/CICD_inflean/02/02깃허브액션결과3.png "이미지")

## 2. GitHub Actions Step
### 1. 여러 명령어 문장 작성
 - yaml => 스텝

   ```  
   steps:
     - name: Hello World 찍기 # Step에 이름 붙이는 기능
       run: echo "Hello World" # 실행시킬 명령어 작성 
  
     - name: 여러 명령어 문장 작성하기
     run: |
       echo "Good"
       echo "Morning"
   ```
 - ![내 그림](assets/img/infra/CICD_inflean/02/02명령어2문장.png "이미지")

### 2. gitHub Actions 변수 사용
  - yaml => 스텝
 
    ```
     ~~~~~~  
     - name: Github Actions 자체에 저장되어 있는 변수 사용해보기
       run: |
         echo $GITHUB_SHA # 깃허브 커밋 코드
         echo $GITHUB_REPOSITORY #레퍼지토리명 
    ```
   - ![내 그림](assets/img/infra/CICD_inflean/02/02명령어저장변수.png "이미지")

### 3. secret 값
  - 시크릿값 설정
   - ![내 그림](assets/img/infra/CICD_inflean/02/02시크릿값등록.png "이미지")
   - ![내 그림](assets/img/infra/CICD_inflean/02/02시크릿값등록2.png "이미지")

   - yaml => 스텝

     ```
       - name: Github Actions Secret 변수 사용해보기
       run: |
         # 시크릿 키값 : `$중괄호열기중괄호열기 시크릿.키값 중괄호닫기중괄호닫기`
         echo 문법키값 # 시크릿(secret) 쩜(.) 키값(MY_NAME)   
         echo 문법키값
     ```
  - ![내 그림](assets/img/infra/CICD_inflean/02/02시크릿값사용.png "이미지")
