---
title: 06 AWS Cloud Front
date: 2025-12-23 10:00:00 +09:00
categories: [infra, awsInflearn]
tags: [ infra, awsInflearn ]
---

## 1. AWS CloudFront
### 1. AWS CloudFront
 - CloudFront는 AWS에서 제공하는 CDN(콘텐츠 전송 네트워크) 서비스
 - 원본(Origin): S3, ALB, EC2 등 CloudFront가 원본 데이터를 받아 엣지 로케이션에 캐시 사용자는 가장 가까운 엣지에서 응답
 - AWS에서 S3를 Https를 사용하기 위해서는 CloudFront가 필요.

### 2. CDN(Content Delivery Network)
 - CDN은 전 세계 여러 지역에 서버를 두고, 콘텐츠를 사용자와 가장 가까운 서버에서 전달하는 구조
 - 구성
   - 원본 서버(Origin): 실제 데이터가 있는 곳
   - CDN 엣지 서버: 캐시된 복사본 보관
 - 사용자 → 가장 가까운 CDN 서버
   - 캐시 있음 → 바로 응답
   - 캐시 없음 → 원본에서 가져와 캐시 후 응답
 - CDN 제공회사
 
   | 회사         | 서비스명           | 특징                                | 주 사용 환경        |
   | ---------- | -------------- | --------------------------------- | -------------- |
   | AWS        | CloudFront     | AWS 서비스와 강한 연동, S3·ALB·ACM·WAF 통합 | AWS 기반 서비스     |
   | Cloudflare | Cloudflare CDN | DNS + CDN + 보안 통합, 설정 간단          | 웹 서비스 전반       |
   | Akamai     | Akamai CDN     | CDN 원조, 초대규모 글로벌 트래픽 처리           | 엔터프라이즈, 대기업    |
   | Google     | Cloud CDN      | 구글 글로벌 네트워크 기반                    | GCP 환경         |
   | Microsoft  | Azure CDN      | Azure 서비스와 연동, 다중 백엔드 옵션          | Azure 환경       |
   | Fastly     | Fastly CDN     | 실시간 캐시 무효화, 개발자 친화적               | 미디어, 트래픽 큰 서비스 |
   | StackPath  | StackPath CDN  | 비교적 저렴, 간단한 CDN 구성                | 중소 규모 서비스      |
   | Bunny.net  | Bunny CDN      | 저렴한 비용, 빠른 설정                     | 개인·스타트업        |

## 2. 정적 웹사이트 생성
### 1. S3 bucket 생성
  - 기존 버킷과 생성 차이 없음

### 2. 정적웹사이트 설정
  - 해당 버킷에 접근하여 속성탭
    - ![내 그림](assets/img/infra/aws_inflean/6정적웹사이트생성1.png "이미지")
  - 해당 관련 편집 진행 / 기본 페이지 등
    - ![내 그림](assets/img/infra/aws_inflean/6정적웹사이트생성2.png "이미지")
  - 완료하면 정적웹사이트 경로 생성 
    - ![내 그림](assets/img/infra/aws_inflean/6정적웹사이트생성3.png "이미지")

## 3. CloudFront
### 1. 생성
  - CludeFront 생성
    - ![내 그림](assets/img/infra/aws_inflean/6클라우드프론트 생성.png "이미지")
  - Origin Domian - 기존에 생성해둔 S3가 자동으로 찾아서 선택지에 생성됨
    - ![내 그림](assets/img/infra/aws_inflean/6클라우드프론트 생성2.png "이미지")
  - 뷰어 프로토콜 정책 - http로 접근이 오면 Https로 리다이렉트처리
    - ![내 그림](assets/img/infra/aws_inflean/6클라우드프론트 생성3.png "이미지")
  - 커넥티브에서 어디어디에 CDN서버를 둘지 결정함, 많을수록 비용증가
    - ![내 그림](assets/img/infra/aws_inflean/6클라우드프론트 생성4.png "이미지")

### 2. 정보 - 접근 url
  - ![내 그림](assets/img/infra/aws_inflean/6클라우드프론트정보.png "이미지")

## 4. CloudFront - Cloud53 / HTTPS적용
  - AWS는 ACM을 통해서 인증서를 받고 적용함.
  - 버지니아 북부 리전을 선택해서 인증서를 발급 받아야함(규정)
  - 기존 작업과 동일함. 크게 차이없음.
  - 굳이 AWS에서 진행안해도 동일함.
