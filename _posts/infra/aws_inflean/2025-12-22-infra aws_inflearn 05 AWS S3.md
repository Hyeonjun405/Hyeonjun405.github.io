---
title: 05 AWS S3
date: 2025-12-22 10:00:00 +09:00
categories: [infra, awsInflearn]
tags: [ infra, awsInflearn ]
---

## 1. S3
### 1. Amazon S3 (Simple Storage Service)
 - S3는 서버에 디스크를 붙이는 게 아니라, 인터넷을 통해 접근하는 객체(Object) 저장소
 - 파일 저장 뿐만 아니라 파일을 다운받는 것에 대해서도 최적화되어 있는 서비스
 - S3는 서버와 분리된 파일 저장소
 - AP 서버 터져도 파일은 안전함.
 

### 2. S3 저장 대상
 - 이미지, 영상
 - 로그 파일
 - 백업 파일
 - 정적 웹 파일
 - 업로드된 첨부파일

### 3. 특징
 - 용량 제한 사실상 없음(비용은 발생함.)
 - 매우 높은 내구성 (데이터 유실 거의 없음)
 - 서버 없이 파일 저장/조회 가능
 - HTTP(S)로 접근

### 4. 사용패턴
 - 업로드
   - 사용자가 EC2의 무엇인가(AP 등)을 통해서 파일을 업로드하면,
   - EC2가 S3에 업로드를 하고, S3가 그 경로를 EC2에 리턴
   - EC2는 RDS(DB)에 저장 url을 insert함.
 - 다운로드
   - 사용자가 특정 객체의 다운로드를 시도
   - EC2에서 해당 객체의 URL을 RDS에서 조회
   - RDS는 url을 리턴
   - EC2가 클라이언트에게 다운로드가능한 URL을 제공
   - 클라이언트는 해당 URL에서 다운로드 하게됨.

### 5. 관련 개념
 - 버킷(Bucket)
   - 버킷은 S3에서 파일을 담는 최상위 컨테이너
   - 개념적으로는 “폴더가 아니라 저장소 자체”에 가까움(깃의 레파지토리)
   - 리전 단위로 생성
   - 접근 권한, 정책, 암호화 설정은 버킷 단위로 관리
 - 오브젝트 (Object)
   - 오브젝트는 S3에 저장되는 실제 파일 하나
   - 오브젝트 구성
     - 파일 데이터
     - 키(Key): 파일 경로처럼 보이는 이름
     - 메타데이터

## 2. S3 생성/사용
### 1. S3 생성
  - 버킷 생성
    - ![내 그림](assets/img/infra/aws_inflean/5버킷생성.png "이미지")
  - 버킷 퍼블릭 액세스 차단 (외부에서 접근 가능 여부 판단)
    - ![내 그림](assets/img/infra/aws_inflean/5버킷액세스차단설정.png "이미지")

### 2. S3 정책 추가(다운로드)
#### 1. 버킷보안정책
 - 생성할때 퍼블릭을 허용한다고 하더라도, 추가로 어떻게 허용할 것인지 설정이 필요함.
 - 권한(Permission)을 정의하는 JSON 문서를 정책이라고 표현함.
 - 정책 작성은 예문이 제공됨.

#### 2. 정책 설정
 - ![내 그림](assets/img/infra/aws_inflean/5버킷보안정책.png "이미지")

#### 3. 정책편집
  - 새문 추가를 누르면 내용이 자동 작성됨
    - ![내 그림](assets/img/infra/aws_inflean/5버킷보안정책2.png "이미지")
  - 필요한 정보를 검색 및 작성 
    - ![내 그림](assets/img/infra/aws_inflean/5버킷보안정책3.png "이미지") 
    - 경로 설정
      - ![내 그림](assets/img/infra/aws_inflean/5버킷보안정책4.png "이미지")
    - 정책 JSON 문서
      ```
       {
         "Version": "2012-10-17",
         "Statement": [ -- 정책의 실제 규칙 목록
          {
            "Sid": "Statement1", -- Statement ID로, 규칙을 구분하기 위한 식별자 / 관리용 ID 
            "Principal": "*", -- 이 정책을 적용받는 대상 / "*"는 전체 대상
            "Effect": "Allow", -- 허용(Allow)인지 거부(Deny)인지
            "Action": [ -- 허용할 AWS 작업
              "s3:GetObject" -- 액션에 대한 정보
            ],
            "Resource": [ -- 권한이 적용되는 대상 리소스
              "arn:aws:s3:::instagram-static-files-youhj14/*"  -- arn:aws:s3:::버킷명/파일대상
            ]
          }
        ]
      }
      ```

### 3. IAM등록(업로드)
#### 1. IAM(Identity and Access Management)
 - IAM은 AWS 리소스에 대해 사용자·서비스의 신원과 권한을 제어하는 서비스
 - 역할
   - 사용자 인증 (누구인가)
   - 권한 부여 (무엇을 할 수 있는가)
   - 접근 통제 (어디까지 가능한가)

#### 2. note
 - AWS SDK(= 스프링에서 쓰는 라이브러리)는 Credential Provider Chain부터 시작되고,
 - 이 과정에서 IAM에 관한 값들을 사용함.
 - 그리고 이 IAM에 관한 값이 등록이 되어있다면,
 - 값 인증을 하고 권한이 있으면 해당 작업을 진행함.
 - 즉, S3에 업로드할때 스프링부트의 경우에는 프로퍼티스로 저값을 설정해두고,
 - S3 접근라이브러리에서 이값을 가지고 권한을 확인,
 - 그다음에 접근을 시도함.

#### 3. 등록
  - IAM으로 접근
    - ![내 그림](assets/img/infra/aws_inflean/5IAM등록.png "이미지")
  - 사용자 등록에서 직접 정책연결(S3Full - S3에 대한 모든 접근)
    - ![내 그림](assets/img/infra/aws_inflean/5IAM등록2.png "이미지")
  - 생성한 사용자 접근후 보안자격증명 - 액세스키
    - ![내 그림](assets/img/infra/aws_inflean/5IAM등록3.png "이미지")
  - Access 키 종류 선택
    - ![내 그림](assets/img/infra/aws_inflean/5IAM등록4.png "이미지")
  - 생성하면, 액세스키와 비밀 액세스키가 존재함.(이걸 AP에서  액세스키 + 비밀액세스키 검증으로 확인)
    - ![내 그림](assets/img/infra/aws_inflean/5IAM등록5.png "이미지")
