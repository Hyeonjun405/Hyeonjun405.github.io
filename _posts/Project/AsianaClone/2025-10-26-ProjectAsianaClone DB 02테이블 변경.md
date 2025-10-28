---
title: 02 테이블 변경
date: 2025-10-26 10:00:00 +09:00
categories: [01asianaClone, asianaData]
tags: [ asianaClone, Data ]
---

## 1. 주요기능
 - 항공편 검색 및 예약
 - 항공편 예약 확인
 - 항공편 출도착 조회
 - 이벤트 조회 및 쿠폰 발급

## 2. Note
 - 상황
   - View와 SpringBoot 기초 데이터를 만들어서 항공편 조회 기능을 구현을 완료함.
   - 항공편 테스트 케이스를 500건으로 만들고 다른 화면에서 작업을 진행하려고 했으나, 건수가 굉장히 많아지고 데이터자체가 불필요한게 많아서 리소스가 소모되는 것을 확인.
 - 변경
   - flights의 항공 ID별 좌석을 airplane_seats 테이블에 저장을 했으나, flights에서 aircrafts의 항공편 종류를 참조하는 것으로 변경함. 
   - 테이블 조회하는 첫 단계라 영향도가 작아서 그냥 테이블 구조 변경함.
 
## 3. 테이블 구성
### 1. 테이블 종류

| 변경 | 구분    | 테이블명                  | 기능            | 설명                                                    | 비고                                 |
|----|-------| --------------------- |---------------| ----------------------------------------------------- |------------------------------------|
| V  | 1차 | flights              | 항공편 기본 정보 관리  | 출발지, 도착지, 출도착 시간 등 항공편의 기본 정보를 저장                     | 모든 예약·좌석·운항상태의 기준 테이블              |
|    | 1차 | flight_status       | 항공편 운항 상태 관리  | 특정 항공편의 운항 상태(예정, 지연, 취소 등)와 지연 시간, 최종 업데이트 시각을 관리    | flights 테이블과 1:1 관계 (flight_id 기준) |
| V  | 1차 | aircrafts      | 항공기별 좌석 정보 관리 | 항공편별 좌석 등급(이코노미, 비즈니스 등)과 총 좌석 수를 저장                  | flights 테이블과 N:1 관계                |
|    | 1차 | users               | 사용자 정보 관리     | 회원의 로그인 정보, 연락처, 이메일 등 사용자 계정을 관리                     | reservations 테이블과 1:N 관계           |
|    | 1차 | reservations        | 항공편 예약 정보 관리  | 유저가 항공편을 예약할 때 생성되는 예약 내역을 관리 (승객정보, 좌석등급, 예약상태 등 포함) | flights, users 모두 참조 (N:1 관계)      |
|    | 2차 | events        | 이벤트 정보 관리     | 프로모션이나 할인 이벤트 정보를 관리하는 테이블                            | 쿠폰과 1:N 관계 예상                      |
|    | 2차 | coupons       | 쿠폰 정보 관리      | 쿠폰 코드, 할인율, 만료일 등의 정보를 저장                             | user_coupons 테이블과 1:N 관계           |
|    | 2차 | user_coupons  | 사용자별 쿠폰 발급 관리 | 어떤 사용자가 어떤 쿠폰을 발급받았는지를 관리                             | users, coupons 참조 (N:1 관계)         |

### 2. 테이블 관계

| 기능 단계  | 주요 테이블 간 관계                             | 설명                    |
| ------ | --------------------------------------- | --------------------- |
| 항공편 검색 | flights ↔ airplane_seats                | 항공편별 좌석 등급/수량 확인      |
| 예약 등록  | flights ↔ reservations ↔ users          | 유저가 특정 항공편 예약 생성      |
| 예약 확인  | users ↔ reservations ↔ flights          | 내 예약 조회, 상태 확인        |
| 출도착 조회 | flights ↔ flight_status                 | 항공편 운항상태(지연, 취소 등) 확인 |
| 쿠폰/이벤트 | users ↔ user_coupons ↔ coupons ↔ events | 유저별 이벤트 참여 및 할인 처리    |

## 4. 변경
#### 1. flights (비행기 노선 정보)

  ``` 
  CREATE TABLE flights.flights (
      flight_id BIGSERIAL PRIMARY KEY,
      flight_number VARCHAR(20) NOT NULL,
      aircraft_id BIGINT NOT NULL,
      departure_airport VARCHAR(50) NOT NULL,
      arrival_airport VARCHAR(50) NOT NULL,
      departure_time TIMESTAMP NOT NULL,
      arrival_time TIMESTAMP NOT NULL,
      CONSTRAINT fk_flight_aircraft FOREIGN KEY (aircraft_id)
          REFERENCES flights.aircrafts (aircraft_id)
          ON DELETE RESTRICT
  );
  ```

#### 2. aircrafts (항공기 종류 + 좌석 구성 포함)
  ```
  CREATE TABLE flights.aircrafts (
      aircraft_id BIGSERIAL PRIMARY KEY,   -- 항공기 ID
      aircraft_code VARCHAR(20) UNIQUE,    -- 항공기 코드 (예: A321)
      model_name VARCHAR(50) NOT NULL,     -- 모델명
      manufacturer VARCHAR(50),            -- 제조사
      economy_seats INT NOT NULL,          -- 이코노미 좌석 수
      business_seats INT DEFAULT 0,        -- 비즈니스 좌석 수
      first_seats INT DEFAULT 0,           -- 퍼스트 클래스 좌석 수
      total_capacity INT GENERATED ALWAYS AS 
          (economy_seats + business_seats + first_seats) STORED
  );
  ```

## 2. ERD
![내 그림](assets/img/AsianaClone/tableERD2.png "이미지")

