---
title: 03 AWS ELB
date: 2025-12-20 10:00:00 +09:00
categories: [infra, awsInflearn]
tags: [ infra, awsInflearn ]
---

## 1. ELB
### 1. ELB(Elastic Load Balancer)
 - 트래픽(부하)를 적절하게 분배해주는 장치를 로드밸런서(Load Balancer)
 - 서버를 2대 이상 가용할 때 ELB를 필수적으로 도입하게 됨.
 - AWS의 ELB에는 부가 기능으로 SSL/TLS(HTTPS)를 제공함.

### 2. SSL/TLS
 - SSL/TLS는 HTTP를 HTTPS로 바꿔주는 인증서
 - SSL (Secure Sockets Layer)
   - 넷스케이프가 만든 초기 암호화 프로토콜
   - SSL 2.0 / 3.0 존재
   - 보안 취약점 때문에 완전히 폐기
 - TLS (Transport Layer Security)
   - SSL을 계승한 표준 프로토콜
   - IETF 표준
   - 현재 사용 중인 유일한 방식
 - SSL은 폐기된 옛 기술이고, TLS는 그 후속이자 현재 표준

### 3. http - domain - SSL/TLS
 - 클라이언트가 도메인으로 접근하면 DNS를 통해 서버의 IP를 확인하고,
 - 해당 IP로 접속을 시도하면 서버는 TLS 인증서를 전달
 - 클라이언트는 인증서를 검증한 뒤 대칭키를 생성하고,
 - 이후부터는 암호화된 통신을 수행
  
### 4. ELB를 사용
 - < 클라이언트 -> EC2 > 로 접근하던 방식에서
 - < 클라이언트 - ELB - EC2 > 로 접근하도록 변경됨.
 - 만약 EC2가 여러개 라면
 - < 클라이언트 - ELB - EC2(1), EC2(2), EC2(3) > 로 ELB가 EC2를 기준에따라 적정하게 분배함.

## 2. 로드밸런서 생성
### 1. EC2내에 로드밸런서에 접근
  - ![내 그림](assets/img/infra/aws_inflean/3로드밸런서생성.png "이미지")

### 2.로드밸런서 유형
   - ![내 그림](assets/img/infra/aws_inflean/3로드밸런서유형.png "이미지")
     - Application Load Balancer : URL, 호스트, 헤더 기준으로 트래픽을 분기하는 웹·API용
     - Network Load Balancer : 초고성능·고정 IP로 대량 트래픽을 빠르게 전달하는 네트워크용
     - Gateway Load Balancer : 보안·네트워크 장비(방화벽, IDS/IPS 등)로 투명하게 전달·확장하기 위한 게이트웨이형

### 3. 네트워크 매핑
  - ![내 그림](assets/img/infra/aws_inflean/3네트워크매핑.png "이미지")
    - VPC : 로드밸런서가 속하는 논리적인 가상 네트워크 범위로, 통신 가능한 IP 대역과 네트워크 경계를 정의
    - 가용 영역 및 서브넷 : 로드밸런서를 배치할 실제 위치, 여러 가용 영역을 선택해 장애 발생 시에도 트래픽을 분산·유지

### 4. 보안그룹
 - 보안그룹 별도 생성 필요
   - ![내 그림](assets/img/infra/aws_inflean/3보안그룹생성.png "이미지")
  
 - 보안그룹의 인바운드,아웃바운드
   - ![내 그림](assets/img/infra/aws_inflean/3보안그룹의 인바운드,아웃바운드.png "이미지")
   - 사용자가 ELB로 접근하기 전에 무너가 결정하는게 인바운드
   - ELB가 나가는게 아웃바운드
  
### 5. 리스너/라우터 및 대상 그룹 생성
  - HTTP:80으로 접근할때, 어디로 연결(대상그룹/URL/고정응답) 할 것인가 에 대한 설정
    - ![내 그림](assets/img/infra/aws_inflean/3리스너및라우터.png "이미지")

  - 포트 80번으로 타겟으로 보냄.
    - ![내 그림](assets/img/infra/aws_inflean/3대상그룹생성.png "이미지")
    
  - 헬스 체크
    - ![내 그림](assets/img/infra/aws_inflean/3헬스체크.png "이미지")
    - 주기적으로(기본 30초) 대상 그룹에 속해있는 각각의 EC2 인스턴스에 요청함
    - 만약 문제가 있을 경우 정해진 상태코드가 안오게 되고,
    - 안오면 그쪽으로는 로드밸런싱 진행하지 않음.

### 6. 로드밸런서가 완료되면 DNS주소 확인가능함.
  - ![내 그림](assets/img/infra/aws_inflean/3ELBDNS설정됨.png "이미지")


## 3. SSL/TLS 적용
### 1. Certificate Manager
  - AWS에서 제공하는 SSL/TLS 인증서를 손쉽게 프로비저닝, 관리, 배포 및 갱신 서비스

### 2. 로드밸런서에 인증서 추가
  - 기존에 만들어놓은 로드밸런서에 리스너추가
    - ![내 그림](assets/img/infra/aws_inflean/3ELBDNS설정됨.png "이미지")

  - https 리스너추가 후 인증서 추가
    - ![내 그림](assets/img/infra/aws_inflean/3인증서추가.png "이미지")

  - 기존의 http 접근은, https로 리다이렉트처리가 필요함.
    - ![내 그림](assets/img/infra/aws_inflean/3인증서추가.png "이미지")

## 4. note
### 1. note
 - HTTPS는 ELB가 아니라 서버에서도 가능함.
 - 비용 절약 측면에서 서버에서 진행하기도 하는데, 개발자가 작업해야 하는게 많아짐.
 - ACME 클라이언트로 인증서를 화인 갱신등의 작업을 함.

### 2. ACME 클라이언트

| 구분         | 이름              | 특징                            | 주 사용 환경              |
| ---------- | --------------- | ----------------------------- | -------------------- |
| 표준         | certbot         | 가장 널리 쓰이는 공식 클라이언트, 문서·예제 풍부  | 일반 서버, Nginx, Apache |
| 경량         | acme.sh         | 쉘 스크립트 기반, 매우 가볍고 의존성 적음      | 경량 서버, 컨테이너          |
| Go 기반      | lego            | Go 언어 기반, 다양한 DNS 플러그인 지원     | Docker, Kubernetes   |
| 경량 배치      | dehydrated      | 단순 구조, 스크립트 자동화에 적합           | 배치 서버, 크론            |
| Kubernetes | cert-manager    | K8s 네이티브, Ingress와 자동 연동      | Kubernetes           |
| 클라우드 관리형   | AWS ACM         | AWS에서 완전 자동 관리, 파일 직접 다운로드 불가 | ALB, NLB, CloudFront |
| 클라우드 관리형   | Cloudflare SSL  | 프록시 사용 시 자동 HTTPS             | Cloudflare 사용 환경     |
| 상용 CA      | DigiCert ACME   | 상용 OV/EV 인증서 자동화              | 기업 환경                |
| 상용 CA      | GlobalSign ACME | 기업용 PKI 연계                    | 기업·금융                |
 
 

