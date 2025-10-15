---
title: 02 프로젝트 노트
date: 2025-10-14 10:00:00 +09:00
categories: [01asianaClone, base]
tags: [ asianaClone ]
---

## 1. asiana-clone
### 1. asiana-clone
 - 아시아나 항공 예약 시스템을 기반으로 클론코딩한 웹 애플리케이션, Spring Boot와 여러 기술들을 활용한 백엔드 구조를 경험하고자 진행함.

### 2. 주요 기능
  - 항공편 검색 및 예약
  - 항공편 예약 확인
  - 항공편 출도착 조회
  - 이벤트 조회 및 쿠폰 발급

## 2. 패키지
### 1. Note 
 - 초기 화면 전체 구성을 확인을 위해 Common에서 get방식으로 전체 화면 체크함.
 - 모듈별로 데이터베이스에 접근하는 목적과 조건이 달라서, Common에서 하나씩 분리해나가면서 개발 진행.
 - 최종적으로 Common에 잔류하는 매핑들은 단순 화면 조회용 매핑. 
  
### 2. 패키지 구분

  | 패키지(모듈별 최상위) | 용도 | 하위 패키지     | class            | 설명                            |
  |--------------|--|------------|------------------|-------------------------------| 
  | common       | 공통 | controller | CommonController | - 단순 화면 조회용 컨트롤러              |
  |              |  | service    |                  |                               |
  | booking      | 항공권 예약 | controller |                  |                               |
  |              |  | service    |                  |                               |
  |              |  | dto        |                  |                               |
  |              |  | vo         |                  |                               |
  | reservation  | 예약 조회 | controller |                  |                               |
  |              |  | service    |                  |                               |
  |              |  | dto        |                  |                               |
  |              |  | vo     |                  |                               |
  | flight       | 출도착 조회 | controller | FlightController | - 기본 컨트롤러                     |
  |              |  | service    | FlightService    | - 출/도착지 리스트 <br> - 비행기 항공편 조회 |
  |              |  | dto        | FlightRequestDto | - 정보조회 DTO                    |
  |              |  | vo     | FlightStatus     | - 출도착 결과 데이터 VO               |
  | login        | 로그인/회원 | controller |                  |                               |
  |              |  | service    |                  |                               |
  |              |  | dto        |                  |                               |
  |              |  | vo     |                  |                               |
 
## 3. 매핑 
### 1. note
 - 1차 작업으로 화면 페이지 접근 완료
 - 단순 조회 화면인 출도착 작업부터 순차 작업 진행.
 
### 2. 매핑 리스트

 | 모듈          | url          | (HTTP)Method | 기능              | 설명                        |
 |-------------|--------------|--------------|-----------------|---------------------------|
 | Common      | /            | get          | 루트페이지           | - 기본 indexPage            |
 | booking     | /booking     | get          | 예약 페이지 기본 접근    | - 예약 페이지 접근               |
 | reservation | /reservation | get          | 예약 확인 페이지       | - 예약 확인 페이지 접근            |
 | flight      | /flight      | get          | 출도착 조회 페이지 접근   | - 출도착 페이지 접근              |
 | flight      | /flight/search     | Post         | 비행기 출도착 및 현황 조회 | - 비행기 출도착 정보 확인 후 하단부에 생성 |
 | Event       | /event       | get          | 이벤트 페이지 기본 접근   | - 이벤트 페이지 접근              |
 | login       | /login       | get          | 로그인창 접근         | - 로그인 창 접근                |
 
## 4. 데이터베이스
### 1. note
 
### 2. 디비 연결
