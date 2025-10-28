---
title: 01 테이블 생성
date: 2025-10-25 10:00:00 +09:00
categories: [01asianaClone, asianaData]
tags: [ asianaClone, Data ]
---

## 1. 주요기능
 - 항공편 검색 및 예약
 - 항공편 예약 확인
 - 항공편 출도착 조회
 - 이벤트 조회 및 쿠폰 발급

## 2. Note  
 - 기준
   - 1차 작업은 기본 기능이 목표로 구성하여, 4개 기능이 들어갈 수 있는 테이블 위주로 구성
   - 항공편을 중심으로 예약 및 비행기편이 관리하므로 가장 최소한의 기준이 되는 테이블을 생성 
   - 기준 테이블의 PK값을 중심으로 예약과 항공편 상태를 관리 필요.
 
 - 항공편 예약
   - 비행기 내 좌석이 뭐뭐있는지에 따라 가격이 바뀌므로 구분은 필요
   - 하지만, 예약시 연령대별 자리는 불필요함 좌석의 합계만 맞으면 문제없음. 

 - 추가 개발건
   - 향후 쿠폰, 유저 멤버관리, 기본정보 자동저장 등은 추후 구현(2025.10.25.)
 
## 3. 테이블 구성
### 1. 테이블 종류

| 구분    | 테이블명                  | 기능            | 설명                                                    | 비고                                 |
|-------| --------------------- | ------------- | ----------------------------------------------------- | ---------------------------------- |
| 1차    | flights              | 항공편 기본 정보 관리  | 출발지, 도착지, 출도착 시간 등 항공편의 기본 정보를 저장                     | 모든 예약·좌석·운항상태의 기준 테이블              |
| 1차    | flight_status       | 항공편 운항 상태 관리  | 특정 항공편의 운항 상태(예정, 지연, 취소 등)와 지연 시간, 최종 업데이트 시각을 관리    | flights 테이블과 1:1 관계 (flight_id 기준) |
| 1차     | airplane_seats      | 항공편별 좌석 정보 관리 | 항공편별 좌석 등급(이코노미, 비즈니스 등)과 총 좌석 수를 저장                  | flights 테이블과 1:N 관계                |
| 1차      | users               | 사용자 정보 관리     | 회원의 로그인 정보, 연락처, 이메일 등 사용자 계정을 관리                     | reservations 테이블과 1:N 관계           |
| 1차      | reservations        | 항공편 예약 정보 관리  | 유저가 항공편을 예약할 때 생성되는 예약 내역을 관리 (승객정보, 좌석등급, 예약상태 등 포함) | flights, users 모두 참조 (N:1 관계)      |
| 2차 | events        | 이벤트 정보 관리     | 프로모션이나 할인 이벤트 정보를 관리하는 테이블                            | 쿠폰과 1:N 관계 예상                      |
| 2차 | coupons       | 쿠폰 정보 관리      | 쿠폰 코드, 할인율, 만료일 등의 정보를 저장                             | user_coupons 테이블과 1:N 관계           |
| 2차 | user_coupons  | 사용자별 쿠폰 발급 관리 | 어떤 사용자가 어떤 쿠폰을 발급받았는지를 관리                             | users, coupons 참조 (N:1 관계)         |

### 2. 테이블 관계

| 기능 단계  | 주요 테이블 간 관계                             | 설명                    |
| ------ | --------------------------------------- | --------------------- |
| 항공편 검색 | flights ↔ airplane_seats                | 항공편별 좌석 등급/수량 확인      |
| 예약 등록  | flights ↔ reservations ↔ users          | 유저가 특정 항공편 예약 생성      |
| 예약 확인  | users ↔ reservations ↔ flights          | 내 예약 조회, 상태 확인        |
| 출도착 조회 | flights ↔ flight_status                 | 항공편 운항상태(지연, 취소 등) 확인 |
| 쿠폰/이벤트 | users ↔ user_coupons ↔ coupons ↔ events | 유저별 이벤트 참여 및 할인 처리    |

## 4. SQL
### 1. flights (비행기 노선 정보)

  ``` 
  CREATE TABLE flights.flights (
      flight_id BIGSERIAL PRIMARY KEY,        -- 항공편 ID
      flight_number VARCHAR(20) NOT NULL,     -- 항공편 번호
      departureAirport VARCHAR(50) NOT NULL,  -- 출발지
      arrivalAirport VARCHAR(50) NOT NULL,    -- 도착지
      departure_time TIMESTAMP NOT NULL,      -- 출발 시간
      arrival_time TIMESTAMP NOT NULL         -- 도착 시간
  );
  ```

### 2. flight_status (항공편 운항 상태)
  ```
  CREATE TABLE flights.flight_status (
      flight_id BIGINT PRIMARY KEY,             -- 항공편 ID
      status VARCHAR(20) NOT NULL,              -- 운항 상태 (scheduled, delayed, cancelled 등)
      delay_time INT DEFAULT 0,                 -- 지연 시간 (분)
      last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- 최종 업데이트 시간
      CONSTRAINT fk_status_flight FOREIGN KEY (flight_id)
          REFERENCES flights.flights (flight_id)
          ON DELETE CASCADE
  );
  ```

### 3. airplane_seats (항공편별 좌석 정보)
  ```
  CREATE TABLE flights.airplane_seats (
      seat_id BIGSERIAL PRIMARY KEY,            -- 좌석 정보 ID
      flight_id BIGINT NOT NULL,                -- 항공편 ID
      class_type VARCHAR(20) NOT NULL,          -- 좌석 등급 (이코노미, 비즈니스 등)
      total_seats INT NOT NULL,                 -- 총 좌석 수
      CONSTRAINT fk_seat_flight FOREIGN KEY (flight_id)
          REFERENCES flights.flights (flight_id)
          ON DELETE CASCADE
  );
  ```

### 4. users (유저 테이블)
  ```
  CREATE TABLE flights.users (
      user_id BIGSERIAL PRIMARY KEY,            -- 유저 ID
      username VARCHAR(50) NOT NULL,            -- 사용자 이름/아이디
      password VARCHAR(255) NOT NULL,           -- 비밀번호
      email VARCHAR(100),                       -- 이메일
      phone VARCHAR(20)                         -- 전화번호
  );
  ```

### 5. reservations (항공편별 예약 정보)
  ```
  CREATE TABLE flights.reservations (
      reservation_id BIGSERIAL PRIMARY KEY,     -- 예약 ID
      flight_id BIGINT NOT NULL,                -- 항공편 ID
      user_id BIGINT NOT NULL,                  -- 유저 ID
      seat_class VARCHAR(20) NOT NULL,          -- 좌석 등급
      passenger_name VARCHAR(50) NOT NULL,      -- 승객 이름
      passenger_type VARCHAR(10) NOT NULL,      -- 승객 유형 (성인/소아/유아)
      status VARCHAR(20) NOT NULL,              -- 예약 상태 (예약완료/취소 등)
      CONSTRAINT fk_res_flight FOREIGN KEY (flight_id)
          REFERENCES flights.flights (flight_id)
          ON DELETE CASCADE,
      CONSTRAINT fk_res_user FOREIGN KEY (user_id)
          REFERENCES flights.users (user_id)
          ON DELETE CASCADE
  );
  ```

## 5. ERD
 ![내 그림](assets/img/AsianaClone/tableERD.png "이미지")
