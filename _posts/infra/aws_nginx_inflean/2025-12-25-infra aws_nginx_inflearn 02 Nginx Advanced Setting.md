---
title: 02 Nginx Advanced Settings
date: 2025-12-25 10:00:00 +09:00
categories: [infra, nginxInflearn]
tags: [ infra, nginxInflearn ]
---

## 1. ( 1 Nginx * N domain )
### 1. default.conf
  ```
  #서버1
  server {
        listen 80;
        server_name jscode.p-e.kr; # 이 경로의 접근은 A localtion

        location / { # A location
                root /usr/share/nginx/nginx-frontend-react/dist;
                index index.html;
        }
  }
  #서버2
  server {
          listen 80;
          server_name admin.jscode.p-e.kr;  # 이 경로의 접근은 B localtion
  
          location / { # B location
                  root /usr/share/nginx/nginx-frontend-next/out;
                  index index.html;
          }
  }
  ```

### 2. Nginx 재반영
  ```
    # Nginx 설정 파일 중 문법 에러가 있는 지 체크
    $ sudo nginx -t
  
    # Nginx의 설정 파일이 바뀐 경우 아래 명령어를 입력해줘야 설정 파일이 반영된다.
    $ sudo nginx -s reload
  ```

## 2. HTTPS
### 1. HTTPS 인증서
 - HTTPS 인증서 = 서버의 신원을 증명하는 디지털 증명서
 - 사람으로 치면 “공인된 기관이 발급한 신분증”
 - 암호화와 신원확인을 하는 역할을 함.
 - 굳이 특정 인증서lib에 매몰될 필요는 없음.
 - 어디에서인가 발급하고 그걸 사용하기만 하면됨.

### 2. 인증서안에 있는 내용
 - 이 서버의 도메인(example.com)
 - 서버의 공개키(Public Key)
 - 발급자(CA: Certificate Authority)
 - 유효기간
 - 서명(Signature)

### 3. 인증서 생성 방법
  
  | 발급 방식              | 발급 주체 / 도구                                        | 발급 절차 요약                   | 특징                            | 적합한 사용처                 |
  | ------------------ | ------------------------------------------------- | -------------------------- | ----------------------------- | ----------------------- |
  | Let’s Encrypt (무료) | Let’s Encrypt + Certbot<br>(acme.sh, lego 등)      | 도메인 소유 검증 → 자동 발급          | 무료 / 90일 / 자동 갱신<br>사실상 표준    | 일반 웹서비스<br>직접 서버 운영     |
  | 클라우드 매니지드 인증서      | AWS ACM<br>GCP Managed Cert<br>Azure Managed Cert | 콘솔에서 생성 → LB/CDN 연결        | 서버에 인증서 파일 없음<br>갱신 완전 자동     | 클라우드 기반 서비스<br>로드밸런서 구조 |
  | 유료 CA 인증서          | DigiCert<br>GlobalSign<br>Sectigo                 | CSR 생성 → 구매 → 서버 설치        | 비용 발생<br>OV/EV 가능<br>신뢰 정책 엄격 | 금융, 공공<br>외부 감사 대상      |
  | 사내 CA 인증서          | 회사 내부 CA                                          | 내부 CA 발급 → 사내 PC에 루트 CA 배포 | 외부 브라우저 신뢰 ❌<br>내부 신뢰 ⭕       | 사내망 시스템<br>내부 API       |
  | Self-signed 인증서    | 자체 생성                                             | 개인키 생성 → 자체 서명             | 무료 / 즉시 사용<br>브라우저 경고 발생      | 로컬 개발<br>테스트 전용         |

### 4. 인증서의 검증
#### 1. CA(Certificate Authority) - 인증서를 발급하는 주체
 - 특정 도메인의 인증서 발급만 함.

#### 2. 사용자의 클라이언트
 - 신뢰 가능한 CA인가
    - 인증서를 서명한 CA가
    - 브라우저(또는 OS)에 내장된 신뢰 목록에 있는지
 - 인증서가 만료되지 않았는가
    - Not Before / Not After 기간 확인
 - 도메인이 일치하는가
    - 속한 도메인과 인증서의 CN / SAN 값이 일치하는지
 - 인증서 체인이 정상인가
    - 중간 인증서 누락 여부
    - 루트 CA까지 연결되는지
 - 서명이 위조되지 않았는가
    - CA 공개키로 서명 검증 

#### 3. 서버
 - 인증서 + 개인키를 제공
 - TLS 프로토콜 수행
 - 암호화된 통신 처리

### 5. Nginx + certBot 인증서 발급
#### 1. certbot
 - 원래는 Nginx의 설정파일에 직접 인증키와 관련된 내용을 추가해야하지만, 
 - certbot을 이용하면 자동으로 코드를 추가해줌. ( /etc/nginx/conf.d/default.conf )
  ```
    server {
      listen 443 ssl;
      server_name example.com;
  
      # 이부분이 추가됨.
      ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

      location / {
          proxy_pass http://localhost:8080;
      }
    }
  ```

#### 2. 방법
 - 라이브러리 설치
   ```
    $ sudo snap install --classic certbot
    $ sudo ln -s /snap/bin/certbot /usr/bin/certbot
   ```
 
 - 인증서 적용
  ```
    $ sudo certbot --nginx -d <도메인 주소>
    # 예시
    $ sudo certbot --nginx -d jscode.p-e.kr
    $ sudo certbot --nginx -d admin.jscode.p-e.kr
  ```

## 3. conf 파일 분리하기
### 1. conf 파일 분리
 - 관리하는 시스템이 많아질 경우에는 conf 파일이 굉장히 많아지고 두꺼워짐
 - 따라서 이 파일들을 분리해서 사용한다면,
 - 가독성이 좋아지고 특정 사이트를 디컴 할때 빠르게 처리가 가능해짐 
### 2. 구조
#### 1. 예시구조
 ```
  /etc/nginx/conf.d/default.conf
  /etc/nginx/conf.d/websites/jscode.p-e.kr.conf;
  /etc/nginx/conf.d/websites/admin.jscode.p-e.kr.conf;
 ```

#### 2. 신규 conf
 ```
   #기존에 server 파일
   server {
    listen 443 ssl;
    server_name example.com;

    location / {
        proxy_pass http://localhost:8080;
    }
  }
 ```

#### 3. 기준 conf(/etc/nginx/conf.d/default.conf)
 ```
  #해당 파일들을 include 처리함.
  include conf.d/websites/jscode.p-e.kr.conf;
  include conf.d/websites/admin.jscode.p-e.kr.conf
 ```

## 4. 프록시
### 1. 프록시 서버
 - 중간에서 대신 통신해주는 대리인
 - 클라이언트와 서버가 직접 통신하지 않고, 그 사이에 끼어 요청과 응답을 중계
 - 요청을 대신 받아서 전달하고, 응답을 대신 받아서 돌려주는 중간 서버

### 2. 프록시의 역할
 - 보안 (외부 직접 접근 차단)
 - 트래픽 제어
 - 로드밸런싱
 - 캐싱
 - TLS 처리(HTTPS 종료)

### 3. 포워드프록시, 리버스 프록시
 - 포워드 프록시
   ```
    클라이언트 ─▶ 프록시 ─▶ 인터넷(서버)
   ```
   - 프록시는 클라이언트 쪽 네트워크에 있음
   - 서버는 프록시 존재를 모를 수도 있음
   - 보내려고 하는 요청을 관리 또는 보안 처리


 - 리버스 프록시
   ```
    인터넷(클라이언트) ─▶ 프록시 ─▶ 내부 서버
   ```
   - 프록시는 서버 앞단에 있음
   - 클라이언트는 실제 서버를 모름
   - 들어오는 요청을 관리 또는 보안 처리

### 4. 프록시 설정(nginx -> springBoot)
 ```
    # https 제한은 certbot으로 동일하게 하는게 편함
    server {
      listen 80;
      server_name example.com;
  
      location / {
          proxy_pass http://localhost:8080;
  
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
   }  
 ```

### 5. 리버스프록시로 Request 제한
#### 1. 방법
 - `/etc/nginx/conf.d/default.conf`파일에 접근할 수 있는 IP제한을 설정함.
 - 이때 conf파일을 분리해서 사용하는 경우에는 그 해당 페이지만 제한이 가능함.

#### 2. 제한
 ```
  # 서버블록 밖에서 작성 필요
  # limit_req_zone : 요청 수를 제한하기 위한 메모리 공간(zone)과 요청 속도(rate)를 정의
  # $binary_remote_addr : 요청 수를 제한하는 기준을 클라이언트의 IP로 설정
  # zone=mylimit:10m : 메모리 공간(zone)의 이름을 mylimit이라고 지정, 메모리 공간의 크기를 10MB 제한 (약 16만개의 IP 주소를 관리할 수 있음)
  # rate=3r/s : 1초에 최대 3개의 요청만 허용
  
  # 이위치에 해당내용은 선언이 필수.
  limit_req_zone $binary_remote_addr zone=mylimit:10m rate=3r/s;
  
  server {
          # limit_req_zone에서 정의한 mylimit이라는 조건을 이 server 블럭에 적용
          limit_req zone=mylimit;
          
          # 요청이 제한됐을 때 429(Too Many Requests) 상태 코드를 반환
          limit_req_status 429;
          
          server_name api.jscode.p-e.kr;
          
          location / {
                  proxy_pass http://localhost:8080;
          }
      listen 443 ssl; # managed by Certbot
      ssl_certificate /etc/letsencrypt/live/api.jscode.p-e.kr/fullchain.pem # managed by Certbot
      ssl_certificate_key /etc/letsencrypt/live/api.jscode.p-e.kr/privkey.pm; # managed by Certbot
      include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
      ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
  }
 ```
