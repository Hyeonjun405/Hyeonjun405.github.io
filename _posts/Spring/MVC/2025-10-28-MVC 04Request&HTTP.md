---
title: 04 Request&HTTP
date: 2025-10-28 10:00:00 +09:00
categories: [00Spring Framework, SpringMVC]
tags: [ Java, Spring Framework, SpringMVC ]
---

## 1. 요청 구분
### 1. Spring MVC 컨트롤러 파라미터 애노테이션 비교

| 애노테이션           | 데이터 위치             | HTTP 요청 형식               | 대상     | 특징 / 용도                             | 배열/리스트 처리                                               |
| --------------- | ------------------ | ------------------------ | ------ | ----------------------------------- | ------------------------------------------------------- |
| @RequestParam   | URL 쿼리 파라미터, 폼 데이터 | GET, POST                | 단일 필드  | 특정 파라미터 직접 매핑                       | key 반복 가능 (`roles=111&roles=222`)                       |
| @ModelAttribute | URL 쿼리 파라미터, 폼 데이터 | GET, POST                | DTO/객체 | 여러 파라미터를 DTO/객체로 바인딩, 폼 데이터를 객체로 받음 | 배열/리스트는 key 반복 또는 `roles[0]`, `roles[1]` 형태, JSON 바디 불가 |
| @RequestBody    | HTTP 바디            | POST, PUT, PATCH, DELETE | DTO/객체 | JSON, XML 등 바디 전체를 DTO로 매핑          | JSON 배열/중첩 객체 자유롭게 가능, 폼 방식과 호환 불가                      |


### 2. HTTP 요청 메서드 + Content-Type에 따른 바인딩

| HTTP 메서드    | Content-Type                      | 요청 위치       | Spring 애노테이션                    | 동작/설명                              |
| ----------- | --------------------------------- | ----------- | ------------------------------- | ---------------------------------- |
| GET         | 없음 / query string                 | URL 쿼리 파라미터 | @RequestParam / @ModelAttribute | URL 쿼리에서 값 읽어 DTO/필드 바인딩, 바디 없음    |
| GET         | application/x-www-form-urlencoded | URL 쿼리 파라미터 | @RequestParam / @ModelAttribute | URL 쿼리로 전달된 폼 데이터 바인딩, 바디는 무시      |
| POST        | application/x-www-form-urlencoded | 요청 바디       | @ModelAttribute / @RequestParam | 폼 데이터 바인딩, 배열/리스트 key 반복 방식 가능     |
| POST        | multipart/form-data               | 요청 바디       | @ModelAttribute + MultipartFile | 파일 업로드 + 폼 데이터 바인딩                 |
| POST        | application/json                  | 요청 바디       | @RequestBody                    | JSON 바디 → DTO 매핑, 배열/중첩 객체 자유롭게 가능 |
| PUT / PATCH | application/json                  | 요청 바디       | @RequestBody                    | JSON 바디 → DTO 매핑, 수정용              |
| PUT / PATCH | application/x-www-form-urlencoded | 요청 바디       | @ModelAttribute                 | 폼 데이터 바인딩 가능, 배열/리스트 key 반복        |
| DELETE      | application/json                  | 요청 바디       | @RequestBody                    | JSON 바디 → DTO 매핑, 삭제용              |
| DELETE      | x-www-form-urlencoded             | 요청 바디       | @ModelAttribute                 | 폼 데이터 바인딩 가능, 드물게 사용               |


## 2. Note
 - View 또는 외부에서 어떻게 어떤 방식으로 요청하는지에 따라서 조정이 필요함.
 - 요청 방식 
   - 폼방식으로 진행하게된다면 @ModelAttribute로 받는 방법
   - Json으로 받는다면 @RequestBody로 받기.

## 3. AJAX
### 1. JS
#### 1. Data값에 배열로 넘기는 경우

  ```
  $.ajax({
    url: '/save',
    data: {
      userNo: '123',
      roles: ['111', '222', '333']   // 이 형태면 key 반복으로 전송됨
    },
    traditional: true                 // 배열을 key 반복 방식으로 보내기 위한 설정 (중요)
  });
  ```
#### 2. 요청 자체를 Json으로 요청하는 방법

  ```
  $.ajax({
    url: '/save',
    type: 'POST',
    contentType: 'application/json',
    data: JSON.stringify({
      userNo: '123',
      roles: ['111', '222', '333']
    })
  });
  ```


### 2. Note
 - 타입별 값 방식   
   ```
    ['123','234'] → JSON 방식
    roles[0], roles[1] → 폼 방식
   ```
   
 - @ModelAttribute로 받을 경우 Json을 받지 못해서 traditional는 true 처리 필요함
 - 하지만 ResponseBody로 받을 경우 처리가 가능함.
 - 상황에 따라서 어떻게 처리할지 고민하고 소통이 필요함.

   
 
