---
title: 04 AWS RDS
date: 2025-12-21 10:00:00 +09:00
categories: [infra, awsInflearn]
tags: [ infra, awsInflearn ]
---

## 1. RDS
### 1. RDS(Relational Database Service)
 - AWS가 대신 운영해 주는 관리형 데이터베이스 서비스
 - 별도의 EC2 안에 RDS를 설치해서 사용해도 무관하기는 함.
 - 하지만 관리하는데 비용이나 예산이 들기때문에
 - AWS환경만 쓴다면 이래저래 비용이 발생하니 그냥 RDS를 사용함. 

### 2. RDS에서 쓸 수 있는 DB
 - MySQL
 - PostgreSQL
 - MariaDB
 - Oracle
 - SQL Server

### 3. RDS가 대신 해주는 것
 - DB 설치
 - 자동 백업
 - 장애 시 자동 복구
 - 패치/버전 관리
 - 모니터링
 - 멀티 AZ 이중화

## 2. RDS 생성
### 1. 데이터베이스 생성
  - ![내 그림](assets/img/infra/aws_inflean/4데이터베이스접근.png "이미지")

### 2. DB선택
  - ![내 그림](assets/img/infra/aws_inflean/4DB선택.png "이미지")

### 3. 최초 관리자 설정
  - ![내 그림](assets/img/infra/aws_inflean/4설정.png "이미지")

### 4. 연결-퍼블릭 access 설정(외부접근 가능 여부)
  - ![내 그림](assets/img/infra/aws_inflean/4연결.png "이미지")

### 5. EC2 내에서 보안 그룹 신규 생성
  - ![내 그림](assets/img/infra/aws_inflean/4보안그룹.png "이미지")

### 6. EC2와 데이터베이스 연결(데이터베이스 접근하여 보안 그룹 추가 필요)
  - ![내 그림](assets/img/infra/aws_inflean/4보안그룹등록.png "이미지")

## 3. 파라미터그룹
### 1. 파라미터 그룹
 - 파라미터 그룹은 DB에 대한 설정/옵션 값들을 파라미터그룹에 등록해두면
 - 알아서 설정 및 사용됨.

### 2. 생성
#### 1.접근
  - ![내 그림](assets/img/infra/aws_inflean/4파라미터그룹생성.png "이미지")

#### 2. 생성(별다른 설정없음)
  - ![내 그림](assets/img/infra/aws_inflean/4파라미터그룹생성2.png "이미지")

#### 3. 파라미터 그룹 설정
  - ![내 그림](assets/img/infra/aws_inflean/4편집.png "이미지")
  - 2025.12.20.인프런 강의
  
   ```
     # utf8mb4
      - character_set_client
      - character_set_connection  
      - character_set_database
      - characater_set_filesystem
      - characater_set_results
      - character_set_server
     
     # utf8mb4_unicode_ci
      - collation_connection
      - collation_serve
      
     # Asia/Seoul 
      - time_zone
   ```

### 3. 파라미터 그룹 적용
  - ![내 그림](assets/img/infra/aws_inflean/4파라미터그룹설정.png "이미지")

## 4. 엔드포인트
  - ![내 그림](assets/img/infra/aws_inflean/4엔드포인트.png "이미지")


