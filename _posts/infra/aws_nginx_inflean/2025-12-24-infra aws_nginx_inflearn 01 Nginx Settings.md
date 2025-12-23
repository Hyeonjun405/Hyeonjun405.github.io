---
title: 01 Nginx Setting
date: 2025-12-24 10:00:00 +09:00
categories: [infra, nginxInflearn]
tags: [ infra, nginxInflearn ]
---

## 1. Nginx
### 1. Nginx
 - 서버 앞에서 트래픽을 받아서 정리하고 넘겨주는 고성능 웹 서버이자 리버스 프록시
 - Nginx는 “애플리케이션 서버 앞단”에서 처리
 - 이벤트 기반 구조
 - 매우 적은 리소스로 많은 동시 연결 처리
 - 문지기 + 교통정리 + 방패 역할을 하는 서버

### 2. Nginx 역할
 - 정적 컨텐츠 제공
 - SSL 처리
 - 로드 밸런싱
 - 장애 대응
 - 캐싱
 - 보안 처리 (IP 차단, 요청 수 제한)

### 3. Apache vs Nginx
  
  | 구분         | Nginx            | Apache HTTP Server |
  | ---------- | ---------------- | ------------------ |
  | 기본 구조      | 이벤트 기반, 비동기      | 프로세스/스레드 기반        |
  | 동시 접속 처리   | 매우 강함 (적은 리소스)   | 접속 수 증가 시 리소스 증가   |
  | 메모리 사용     | 적음               | 상대적으로 큼            |
  | 정적 파일 처리   | 매우 빠름            | 보통                 |
  | 동적 콘텐츠 처리  | 리버스 프록시로 WAS에 위임 | 모듈 기반으로 직접 처리 가능   |
  | 설정 방식      | 단순, 명확           | 기능 많고 복잡           |
  | 리버스 프록시    | 매우 강력            | 가능하지만 상대적으로 약함     |
  | SSL/TLS 처리 | 고성능              | 가능하나 설정 복잡         |
  | .htaccess  | 지원 안 함           | 지원                 |
  | 대표 사용처     | 트래픽 입구, 프록시      | 전통적 웹 서버           |

### 4. Nginx와 SpringBoot
 - Nginx는 웹 서버 / 리버스 프록시, 트래픽 입구, HTTPS 처리, 정적 파일 담당
 - 스프링부트는 애플리케이션 서버, 비즈니스 로직, API 처리 담당
 - Nginx는 입구에서 최초 정리, 스프링부트는 실제 비즈니스 처리.
 - nginx가 80번으로 받아서, 스프링부트로 포워딩

## 2. Nginx 생성
### 1. Nginx - Linux
 ```
  # apt에서 설치 가능한 패키지 리스트(최신 패키지, 버전 등)를 최신화
  $ sudo apt update
  
  # nginx 설치에 필요한 라이브러리 설치
  $ sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
  
  # nginx 공식 패키지를 설치 / 공식 문서에서 제공 
  $ curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
      | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
  $ gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
  $ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
  http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
      | sudo tee /etc/apt/sources.list.d/nginx.list
  
  # nginx 설치
  $ sudo apt update
  $ sudo apt install nginx
 ```

### 2. 상태 확인 및 실행
 ```
  # nginx 상태 확인
  $ sudo systemctl status nginx
  
  # nginx 시작
  $ sudo systemctl start nginx
  
  # nginx 상태 확인
  $ sudo systemctl status nginx
 ```

### 3. 로그검증
 ```
  # Nginx 로그 파일 저장 장소
  $ cd /var/log/nginx
  
  # 파일의 마지막 10줄을 출력
  $ tail access.log
  $ tail error.log
 
 ```

## 3. Nginx 기본문법
### 1. Nginx의 설정 파일 위치
 - `/etc/nginx/nginx.conf`
   - Nginx에서 가장 근본이 되는 설정 파일(루트 설정 파일)
   - 전역적으로 설정되어야 하는 내용(워커 프로세스 개수, 로그 저장 위치 등)이 포함되어 있다.
 
 - `/etc/nginx/conf.d/default.conf`
   - 기본 웹 서버(Web Server) 설정 파일
   - 설정 
   ```
     server {
        listen       80;
        server_name  localhost; # localhost로 들어오는 접근은

        # 로케이션 설정
        location / {
            root   /usr/share/nginx/html;
            index  index.html;
        }
   
        # 부트처럼 별도 WAS가 있는 경우 (프록시를 사용해야함)
        location / {
           proxy_pass http://127.0.0.1:8080;
        }

        error_page   500 502 503 504  /50x.html;
    
        location = /50x.html {
           root   /usr/share/nginx/html;
        }
     }
   ```

### 2. 수정 관련 명령어
 ```
  # Nginx 설정 파일 중 문법 에러가 있는 지 체크
  $ sudo nginx -t
  
  # Nginx의 설정 파일이 바뀐 경우 아래 명령어를 입력해줘야 설정 파일이 반영된다.
  $ sudo nginx -s reload
 ```


