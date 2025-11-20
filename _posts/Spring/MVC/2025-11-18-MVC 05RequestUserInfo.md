---
title: 05 Request User Info
date: 2025-11-18 10:00:00 +09:00
categories: [00Spring Framework, SpringMVC]
tags: [ Java, Spring Framework, SpringMVC ]
---

## 1. Request
### 1. Request에서 획득 가능한 정보

| 구분        | 내용                                             | 스프링에서 접근 방식                                  |
| --------- | ---------------------------------------------- | -------------------------------------------- |
| 요청 라인     | HTTP 메소드, URL, 프로토콜 버전                         | HttpServletRequest, @GetMapping 등            |
| 헤더        | User-Agent, Content-Type, Accept 등             | @RequestHeader, HttpServletRequest           |
| 쿼리 파라미터   | ?key=value 형태                                  | @RequestParam, DTO 바인딩, Map                  |
| 바디        | JSON, form-data, x-www-form-urlencoded 등 요청 본문 | @RequestBody, @ModelAttribute, MultipartFile |
| 쿠키        | 클라이언트 저장값                                      | @CookieValue, HttpServletRequest             |
| 세션        | JSESSIONID 기반 세션 정보                            | HttpSession                                  |
| 경로 변수     | /users/{id} 형태                                 | @PathVariable                                |
| Locale 정보 | 브라우저 언어 설정                                     | Locale, LocaleResolver                       |
| 인증 정보     | 로그인 사용자, 권한                                    | Authentication, Principal                    |
| 기타 메타데이터  | Remote IP, 포트, HTTPS 여부 등                      | HttpServletRequest                           |

### 2. 특정 대상 구분

| 구분              | 설명                                                     | 활용 예시                           |
| --------------- | ------------------------------------------------------ | ------------------------------- |
| IP 주소           | 클라이언트의 공인 IP. 프록시/로드밸런서 환경에서는 `X-Forwarded-For` 헤더를 확인 | 해외 접속 여부, 특정 기관 접근 제한, 위치 기반 통계 |
| User-Agent      | 브라우저/OS/디바이스 정보                                        | 모바일/PC 구분, 특정 브라우저 버전 사용자 필터링   |
| Accept-Language | 브라우저 언어 설정                                             | 지역/언어 추정, 다국어 페이지 제공            |
| Referer         | 이전 방문 URL                                              | 유입 경로 분석, 특정 사이트에서 온 요청 필터링     |
| 요청 시간 & 타임존 추정  | 요청 시각과 UTC 대비 시간                                       | 지역 시간대 추정, 해외 접근 패턴 분석          |
| TLS/SSL 정보      | Client 인증서, SNI(Server Name Indication)                | 기업 내부 사용자 인증, 특정 도메인 접근 판단      |
| 헤더 패턴           | Custom header, Proxy 헤더 등                              | 특정 앱/서비스 구분, 모바일 앱 요청 식별        |

## 2. 특정 타겟 구분
### 1. 지역 기반
 - IP 주소 확인: HttpServletRequest.getRemoteAddr()
 - 프록시 환경: X-Forwarded-For, X-Real-IP
 - Accept-Language 헤더: 브라우저 언어 기반 대략 추정
 - 요청 시간 + 타임존: 특정 국가/지역 시간대 기준 패턴 발견
 - 예시 활용: 해외 접속 차단, 특정 국가 접속 통계, 지역 맞춤 콘텐츠 제공

### 2. 사용자/개인 기반
 - IP 패턴: 사내망, 특정 기관 IP 등
 - User-Agent: 특정 디바이스/앱 이용자 구분
 - Referer/Custom Header: 특정 클라이언트 앱 식별
 - 쿠키/토큰: 세션 없이 클라이언트별 추적 가능(임시 ID 부여 등)
 - 예시 활용: 봇 차단, 특정 사용자 집단 추적, A/B 테스트 대상 구분

## 3. 접근 제한
### 1. IP로 구분
#### 1. IP 추출
  ```
  String ip = request.getHeader("X-Forwarded-For");
  if (ip == null || ip.isEmpty()) {
      ip = request.getRemoteAddr(); // 서버가 실제 연결된 클라이언트의 IP 주소
  } else {
      ip = ip.split(",")[0].trim(); // 첫 번째 IP가 실제 클라이언트
  }
  ```
  - getRemoteAddr은 프록시, 로드밸런서, CDN등을 거치면 중간 서버(IP)가 나올 수있음.
  - 따라서 `ip.split(",")[0].trim()`을 통해서 최초 아이피를 알아내야함

#### 2. IP와 국가 매핑
 - GeoIP 데이터베이스 사용
   - MaxMind GeoLite2(무료) 또는 GeoIP2(유료) 사용
   - Java 라이브러리 geoip2로 IP 조회 가능

    ```
      File database = new File("/path/to/GeoLite2-Country.mmdb");
      DatabaseReader dbReader = new DatabaseReader.Builder(database).build();
      
      InetAddress ipAddress = InetAddress.getByName(ip);
      CountryResponse response = dbReader.country(ipAddress);
      
      String countryCode = response.getCountry().getIsoCode(); // 예: KR, US, JP
    ```
   
 - IP 범위/정규식 기반 화이트/블랙리스트
   - 특정 국가 IP 범위를 미리 수집해서 범위 비교
   - CIDR 형식 또는 단순 시작/끝 IP 비교

 - 서드파티 API 사용
   - IP 정보 제공 서비스: ipinfo.io, ipstack, ipapi 등
   - 요청 보내면 JSON으로 국가, 도시, ISP 정보 반환


### 2. Accept-Language
 - 헤더값
   ```
   Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
   ```
    - ko-KR : 가장 선호하는 언어(한국어, 대한민국)
    - ko;q=0.9 : 두 번째 선호 언어
    - en-US;q=0.8, en;q=0.7 : 순차적으로 선호도 낮은 언어

 - Request 확인
  ```
  String language = request.getHeader("Accept-Language");
  ```
 - 특징
   - 사용자가 임의로 바꿀 수 있음 → 완전히 신뢰할 수는 없음
   - IP 기반 추정과 같이 사용하면 정확도 높일 수 있음


### 3. User-Agent
 - 헤더 예시
  ```
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36
  ```
   - Mozilla/5.0 → 전통적으로 호환성을 위해 포함
   - (Windows NT 10.0; Win64; x64) → 운영체제 정보
   - AppleWebKit/537.36 → 렌더링 엔진
   - Chrome/119.0.0.0 → 브라우저 이름과 버전
   - afari/537.36 → 브라우저 엔진 정보

- 패턴

  | 구분           | 패턴 예시                                         | 특징                                        | 실제 사용 여부  |
  | ------------ | --------------------------------------------- | ----------------------------------------- | --------- |
  | 빈 UA         | (빈 문자열), null                                 | 가장 기본적인 봇 패턴. 사람이 쓰는 브라우저는 UA가 거의 항상 존재   | 매우 자주 사용  |
  | 명백한 봇 이름     | bot, crawl, spider, scrapper, scanner         | 구글/네이버 등 크롤러는 정식 이름 포함. 비인가 봇들도 이런 단어를 넣음 | 매우 자주 사용  |
  | 자동화 스크립트     | python-requests, okhttp, curl, wget           | 스크립트 기반 호출에서 자주 쓰는 라이브러리 UA               | 매우 자주 사용  |
  | 라이브러리 기본 UA  | Java/1.8.0, Apache-HttpClient, Go-http-client | 기본값을 변경하지 않은 경우 봇 확률 높음                   | 자주 사용     |
  | 헤드리스 브라우저    | HeadlessChrome, PhantomJS                     | UI 없는 크롤링 환경                              | 많이 사용     |
  | Selenium/자동화 | Selenium, webdriver, Puppeteer, Playwright    | 브라우저 자동화 도구 사용 시 포함되는 문자열                 | 많이 사용     |
  | 모바일 위장 실패    | Mozilla/5.0 (Linux; Android) … 하지만 기종명이 없음    | 실제 모바일 브라우저는 기종/빌드 번호가 매우 상세함             | 사용        |
  | 비정상적 길이      | 너무 짧거나 지나치게 긴 UA                              | 짧으면 자동화, 길면 조작 흔적                         | 사용        |
  | 조작 흔적        | Mozilla/5.0 (compatible) 정도만 적힌 단순 UA         | 자동 생성된 UA 패턴                              | 사용        |
  | 공격 도구 패턴     | sqlmap, nikto, dirbuster                      | 보안 툴들이 기본적으로 넣는 UA                        | 매우 자주 차단됨 |

 - 특징
   - 쉽게 변조 가능 → 단독으로 신뢰할 수 없음
   - 보안이나 차단 목적이면 IP, 세션, 인증 정보와 함께 사용해야 함


### 4. IP + Accept-Language + User-Agent
#### 1. 패턴 조합 기준
 
| 구분          | IP 패턴                 | Accept-Language 패턴      | User-Agent 패턴                              | 의미 / 판단 근거                  |
| ----------- | --------------------- | ----------------------- | ------------------------------------------ | --------------------------- |
| 국내 정상 사용자   | KR 또는 국내 대역           | ko-KR 우선, en-US 보조      | Chrome / Safari / SamsungBrowser 등 정상 브라우저 | 가장 정상적인 패턴                  |
| 해외 정상 사용자   | 해외 대역                 | en-US, ja-JP 등 해당 국가 언어 | 정상 브라우저                                    | 해외에서 실제 사용자 접근 가능성          |
| 해외 우회 접속 의심 | 해외 대역                 | ko-KR이 최우선              | 정상 브라우저                                    | VPN으로 한국 서비스 접속 시 자주 나오는 패턴 |
| 국내 VPN 의심   | 국내 IP로 보임             | 언어가 중동·러시아·남미 언어        | 정상 브라우저                                    | VPN으로 한국 IP를 우회한 경우 나타남     |
| 자동화 스크립트    | 무작위·비정상 대역 또는 클라우드 대역 | 거의 없음 또는 빈 값            | curl, python-requests, Java-httpclient 등   | 자동화 툴 요청 확률 높음              |
| 봇·크롤러       | 클라우드/IDC IP           | 없음                      | bot, crawl, spider 포함                      | 명백한 봇 패턴                    |
| 헤드리스 크롤링    | 클라우드·해외 대역            | en-US 등 기본값             | HeadlessChrome, PhantomJS 등                | 자동화 브라우저 기반 크롤링             |
| 조작 흔적       | 지역 일관성 없음             | zh-CN인데 한국 IP 등 불일치     | UA가 비정상 또는 단순                              | 스푸핑·봇 가능성 높음                |

#### 2. 패턴 예시

| 사용자 구분         | 항목              | 값                                              |
| -------------- | --------------- | ---------------------------------------------- |
| 국내 정상 사용자      | IP              | 한국 통신사 대역 (KT, SKT, LGU+)                      |
|                | Accept-Language | ko-KR,ko;q=0.9,en-US;q=0.8                     |
|                | User-Agent      | Chrome, Safari, SamsungBrowser 등 정상 브라우저       |
| 해외 정상 사용자      | IP              | 미국, 일본 등 해외 대역                                 |
|                | Accept-Language | en-US, ja-JP 등 해당 국가 언어                        |
|                | User-Agent      | 정상 브라우저                                        |
| 해외 VPN → 한국 우회 | IP              | AWS 서울, GCP 서울 등 클라우드 IP                       |
|                | Accept-Language | ko-KR                                          |
|                | User-Agent      | 정상 브라우저                                        |
| 국내 VPN → 해외 사용 | IP              | 한국 통신사 또는 한국 IDC IP                            |
|                | Accept-Language | ru-RU, ar-SA 등 한국 외 언어                         |
|                | User-Agent      | 정상 브라우저                                        |
| 자동화 스크립트       | IP              | 클라우드/IDC 대역 (AWS, Linode 등)                    |
|                | Accept-Language | 없음 또는 빈 값                                      |
|                | User-Agent      | python-requests, curl, okhttp, Java-httpclient |
| 비인가 봇          | IP              | 해외 IDC 또는 변칙 대역                                |
|                | Accept-Language | 없음                                             |
|                | User-Agent      | bot, crawl, spider 포함                          |
| 헤드리스 자동화       | IP              | 해외 클라우드 IP                                     |
|                | Accept-Language | en-US 기본값                                      |
|                | User-Agent      | HeadlessChrome, PhantomJS, Puppeteer           |
