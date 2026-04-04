---
title: 02 VectorDB
date: 2026-04-04 10:00:00 +09:00
categories: [Sparta, SpartaNote]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - 벡터 디비의 흐름!
   - VectorDB에는 Context형태로 저장하는건 맞고
   - metaData로 필터해서 embedding 데이터를 끄집어내서
   - 나온 chunck들을 모은뒤에
   - LLM에 한번더 보내서 그 서류들을 바탕으로 답변하는 패턴을 취함.
 - LLM은 저정할때와 가공할때만 사용되고
 - 나머지는 VectorDB에서 사용
 - Vector_Store는 공통으로 써도 무관함
   - metaData로 내가 원하는 데이터를 찾을 수 있기 때문에 모아서 해도 되기는함!
 - ☆ VectorDB ☆
   - SpartaNote02 - Vector DB < 기본 개념과 흐름 >
   - SpartaWIL04 - 02pgvector_기본 패턴 < 사용법 / 기본 패턴 파악 >
   - SpartaWIL04 - 03pgvector_실무적인 접근 < 실무 패턴 >

## 1. VectorDB 사용 전체 흐름
### 1. 원본 문서 (PDF, DB 데이터, API 응답 등) 저장
 - 텍스트 추출 (필요에 따라서 AP에서 선별해서 넣기도 함)
 - chunk 단위로 분할 (데이터를 한번에 넣으면 추후에 통으로 읽어야하므로 청크화)
 - 임베딩 생성 ( 데이터 임베딩화 )
 - DB 저장 (vector + metadata)

   ```
   CREATE TABLE vector_documents (
     id UUID PRIMARY KEY, -- UUID
     content TEXT, - 원본 문서
     embedding VECTOR(1536), -- 임베딩화 된 데이터
     metadata JSONB, -- JSON 데이터
     created_at TIMESTAMP -- 생성일
   );
   ```
  
### 2. 검색 (Retrieval)
 - 사용자 질문 → 임베딩 생성 
 - pgvector cosine similarity 검색
 - topK 결과 가져오기
 - metadata 기반 필터링

### 3. 응답 생성 (RAG)
 - 검색 결과 + 질문 → LLM에 전달
 - 최종 답변 생성

## 2. PgVector 관련 문법
### 1. Where 관련
  ``` 
  WHERE metadata->>'documentId' = '123'
  -- "->>" 로 JSON 값을 "문자열"로 꺼냄
  
  WHERE (metadata->>'chunkIndex')::int > 5
  -- 값 비교(타입 변경 필요)       
  ```

### 2. 벡터 검색 + WHERE + ORDER
  ```
  SELECT *
  FROM vector_documents
  WHERE metadata->>'category' = 'backend'
  ORDER BY embedding <-> :queryVector 
  LIMIT 5;
  
  
  ORDER BY embedding <->
  -- 유사도 정렬       
   
  :queryVector
  -- 사용자 입력값
  -- Service에서 " float[] queryVector = embeddingClient.embed("스프링 트랜잭션이 뭐야?"); " 요런값
  
  LIMIT
  -- topK
  ```

## 3. metadata (필수 작업)
### 1. JSON형태로 데이터에 관한 정보를 저장함.
  
   ```
   {
     "documentId": "123", 
     "fileName": "spring-guide.pdf",
     "page": 5,
     "chunkIndex": 2,
     "category": "backend",
     "author": "admin",
     "createdAt": "2026-04-04"
   }
   ```

### 2. 특정 metaData 기준 조회
   ```
   SELECT *
   FROM vector_documents
   WHERE metadata->>'category' = 'backend'
   ORDER BY embedding <-> :queryVector
   LIMIT 5;
   
   -- WHERE metadata->>'team' = 'A' => teamA의 데이터를 조회한다
   -- WHERE metadata->>'documentId' = '123' => 도큐먼트Id관리하고 있다면 서류 123번 조회  
   ```

## 4. Return 값
### 1. select * from ~
  ```
  id: 550e8400-e29b-41d4-a716-446655440000
  content: "스프링 트랜잭션은 데이터의 일관성을 보장하기 위한..."
  embedding: [0.123, -0.532, 0.923, ...]
  metadata: {"documentId":"123","chunkIndex":0,"category":"backend"}
  created_at: 2026-04-04 10:00:00
  ```
   
### 2. DTO 결과값
### 1. DTO
```
class VectorDocument {
    UUID id;
    String content;
    float[] embedding;
    Map<String, Object> metadata;
}
```

### 2. 리턴 방식
```
 // select * from 한 값 5개가 VectorDocument에 쌓임
 List<VectorDocument> results = ~~;
 
 // 실무적으로 어차피 비슷한내용 합쳐서 LLM에서 한번더 가공해야하니
 // context로 한번에 받게됨
 // 그냥 사용해도 문제는 없음!
 String context = results.stream()
    .map(r -> r.getContent())
    .collect(Collectors.joining("\n"));
 ```
