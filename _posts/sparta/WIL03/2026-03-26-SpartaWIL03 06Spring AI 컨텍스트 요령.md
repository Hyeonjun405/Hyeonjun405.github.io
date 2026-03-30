---
title: 06 Spring AI 컨텍스트 관리 요령
date: 2026-03-26 10:00:00 +09:00
categories: [Sparta, SpartaWIL03]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - LLM 컨텍스트 관리라는 것은 LLM을 위해 만들어진 전략이 X
   - 메모리를 어떻게 관리하는가에 대한 전략 O
   - 이 메모리전략으로 다른 시스템이나 모듈에서 패턴에 맞게 조정하면 사용이 가능함.
   - 컴퓨터 사이언스의 기본 개념이라고 보면 될듯.

## 2. 기존 이력을 관리하며 응답하는 기본 패턴
### 1. 기존의 대화를 기억한다
- assistantMessage 패턴을 이용해서 구분해서 넣어주거나 히스토리처럼 이전의 대화를 같이 던져누는 방법
  - 히스토리를 넣으면, 로직상에서 기존의 대화를 넣었는지 안넣었는지 구분이 잘안됨.
  - 이럴때는 assistantMessage를 사용하는게 유리하기는 함.
- 메시지를 구성하는 작업이
  - 복잡하게 잡혀있다면 assistantMessage로 분리해서 관리를 하는게 좋아보이고
  - 단순 대화라면 History처럼 뭔가 DB에 저장했다가 한번에 때리는것도 좋아 보임!
- 기억을 주기만 하면 어쨋든 결과는 같음.

### 2. 소스패턴
```
 // 메모리에 저장, DB나 어딘가에 별도 저장 필요
 private final Map<String, List<Message>> conversations = new ConcurrentHashMap<>();

 // conversationId값은 AP와 AI내에서 소통하는 것
 // 저 값은 사용자 - AP 간에는 별도 인증키등으로 하는게 필요 
 public ContextChatResponse chatWithHistory(String question, String conversationId) {

    // ID가 없으면 새로 생성 / < ID값으로 대화 구분함. >
    if (conversationId == null || conversationId.isBlank()) {
      conversationId = UUID.randomUUID().toString();
    }

    // 기존 대화 이력을 가져오거나 새로 생성
    // getOrDefault 첫번째 인자 넣었을때 없으면 default값 넣음(두번째 인자) 
    List<Message> history = conversations.getOrDefault(conversationId, new ArrayList<>());

    // 사용자 질문 추가 후에 List에 내용을 추가함.
    UserMessage userMessage = new UserMessage(question);
    history.add(userMessage);

    try {
      // AI 호출: 지금까지의 history 전체를 메시지로 전달
      ChatResponse response = chatClient.prompt()
          .messages(history)
          .call()
          .chatResponse();

      //실제 응답을 가져옴
      String assistantResponse = response.getResult().getOutput().getText();

      // AI 답변을 히스토리에 추가
      AssistantMessage assistantMessage = new AssistantMessage(assistantResponse);
      history.add(assistantMessage);

      // 업데이트된 히스토리 저장
      // 기존 히스토리 덮음
      conversations.put(conversationId, history);

      // 토큰 사용량 정보 추출 및 DTO 변환
      var usage = response.getMetadata().getUsage();
      ContextChatResponse.TokenUsage tokenUsage = ContextChatResponse.TokenUsage.builder()
          .promptTokens(usage.getPromptTokens().intValue())
          .completionTokens(usage.getCompletionTokens().intValue())
          .totalTokens(usage.getTotalTokens().intValue())
          .build();

      return ContextChatResponse.builder()
          .message(assistantResponse)
          .conversationId(conversationId)
          .timestamp(LocalDateTime.now())
          .tokenUsage(tokenUsage)
          .build();

    } catch (Exception e) {
      log.error("AI 호출 중 오류 발생: {}", e.getMessage());
      throw new DomainException(DomainExceptionCode.AI_RESPONSE_ERROR);
    }
  }
```

## 3. 메모리관리 layer 개념
### 1. Layer 1: Volatile Memory (휘발성 메모리)
- 특징
  - 요청 단위로만 유지 (세션 끝나면 사라짐)
  - 가장 빠름
  - LLM이 직접 참고하는 “현재 문맥”
- 저장소
  - 애플리케이션 메모리 (Heap)
  - 예: Java List, Map
  - 또는 프롬프트 내부 (Chat History)
- 역할
  - “지금 대화 흐름 유지”
  - LLM에게 바로 입력되는 컨텍스트
- 예시
 ```
  List<Message> chatHistory = new ArrayList<>();
  chatHistory.add(new Message("user", "상품 추천해줘"));
  chatHistory.add(new Message("assistant", "어떤 카테고리요?"));
 ```
   
### 2. Layer 2: Non-volatile Memory (비휘발성 메모리)
- 특징
  - 영구 저장 (DB)
  - 세션이 끝나도 유지
  - 속도는 상대적으로 느림
- 저장소
  - RDB (MySQL, PostgreSQL)
  - NoSQL (MongoDB, Redis 일부)
  - 파일 저장소
- 역할
  - “대화 기록 저장”
  - “사용자 상태 유지”
- 예시
 ```
  chatRepository.save(new ChatHistory(userId, "user", "상품 추천해줘"));
 ```

### 3. Layer 3: Semantic Memory (의미적 검색 메모리)
- 특징
  - 벡터 기반 저장
  - 의미로 검색 가능
  - AI 기능의 핵심 레이어
- 저장소
  - Vector DB : 의미(유사도) 기반으로 데이터를 찾는 데이터베이스
  - Pinecone : 서버 관리 없이 바로 쓰는 클라우드형 벡터 DB
  - Milvus : 대규모 데이터 처리에 강한 고성능 오픈소스 벡터 DB
  - Weaviate : 의미 검색 + 필터 기능이 강한 유연한 벡터 DB
- 역할
  - “비슷한 의미 찾기”
  - RAG (Retrieval-Augmented Generation) 핵심
- 예시
  - 저장
   ```
   String text = "사과는 과일이다";
   float[] embedding = embeddingModel.embed(text);
   vectorDB.save(embedding, text);
   ```
  - 검색
   ```
   String query = "과일 종류 알려줘";
   float[] queryVector = embeddingModel.embed(query);
   List<String> results = vectorDB.search(queryVector);
   ```

## 4. LLM 컨텍스트 관리 방법
### 1. 테이블 메모
#### 1. memo
- 채팅창 리스트 테이블을 두고 FK로 채팅 내용 테이블을 둠.
- 1개 채팅창에 여러 채팅내용이 누적되어 조회할 수 있도록 함.
- 또 여러 채팅창을 보관할 수 있게됨.
- 컨텍스트 스위칭(Context Switching) 개념<<

#### 2. 테이블 형태
- chat_conversations (채팅창 구분)
 ```
  CREATE TABLE chat_conversations
  (
    id         UUID PRIMARY KEY      DEFAULT gen_random_uuid(),
    user ~
    title ~     VARCHAR(255) NOT NULL,
    ~~~~~   
 ```
- chat_messages (채팅 내용)
 ```
  CREATE TABLE chat_messages
    id                BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    conversation_id   UUID        NOT NULL, (FK)
    role              VARCHAR(20) NOT NULL, -- USER, ASSISTANT, SYSTEM, SUMMARY
    status            VARCHAR(20) NOT NULL, -- ACTIVE, INACTIVE, DELETED
    message           TEXT        NOT NULL, //메시지를 저장
    ~~~~
 ```

### 2. 컨텍스트 스위칭(Context Switching)
#### 1. 개념
- 사용자가 여러 대화방이나 세션을 오갈 때, 각 대화의 맥락을 독립적으로 유지하고 필요 시 다시 불러오는 방식
- LLM은 상태를 자체적으로 기억하지 못하기 때문에, 매 요청마다 해당 대화의 이전 메시지를 함께 전달해야 함
- 이때 conversationId 같은 식별자를 기준으로 대화 이력을 조회하여 정확한 흐름을 복원함
- 결과적으로 사용자는 대화를 끊었다가 돌아와도 자연스럽게 이어짐

#### 2. 동작 흐름
- 테이블 내용
  - chat_conversations에서 특정의 유저 + 특정 제목을 가지고,
  - chat_message에서 해당 채팅창의 대화했던 채팅 내용 조회하고,
  - 메시지를 보낼때 이전 메시지를 통으로 조회해서 보내도록 함.

- 대화창 구분
 ```
  CREATE TABLE chat_conversations
  (
    id         UUID PRIMARY KEY      DEFAULT gen_random_uuid(),
    user ~
    title ~     VARCHAR(255) NOT NULL,
    ~~~~~   
 ```
- 대화 내용 구분
 ```
  CREATE TABLE chat_messages
    id                BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    conversation_id   UUID        NOT NULL, (FK)
    role              VARCHAR(20) NOT NULL, -- USER, ASSISTANT, SYSTEM, SUMMARY
    status            VARCHAR(20) NOT NULL, -- ACTIVE, INACTIVE, DELETED
    message           TEXT        NOT NULL, //메시지를 저장
    ~~~~
 ```
 
### 2. 프롬프트 최적화
#### 1. 개념

- 대화가 길어질수록 모든 이력을 프롬프트에 넣으면 토큰 초과, 비용 증가, 속도 저하 문제가 발생함
- 이를 해결하기 위해 이전 대화를 요약한 summary를 생성하고, 이후에는 요약 + 최신 메시지만 사용함
- 이 방식은 핵심 맥락은 유지하면서 불필요한 토큰을 줄이는 데 목적이 있음
- 결과적으로 응답 속도와 비용 효율을 동시에 개선할 수 있음.
- 그런데 요약을 요청을 해야해서 토큰이 사용되는 부분은 있음.

#### 2. 동작 흐름
- Service단에서 일정 시점에 요약 생성
  - 00건 이상 또는 00토큰 이상 등
  - 관리하기 편한 특정상황을 잡음.
- 이후에는 summary + 최근 대화만 사용

- 테이블
  ```
  CREATE TABLE chat_messages
    id                BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    conversation_id   UUID        NOT NULL, (FK)
    role              VARCHAR(20) NOT NULL, -- summary (☆)
    status            VARCHAR(20) NOT NULL, 
    message           TEXT        NOT NULL, //메시지를 저장
    ~~~~
  ```

- 포인트
  - 프롬프트에 메시지를 보낼때, SUMMARY는 시스템에 담아서 보내야함.
  ```
  return chatClient.prompt()
        .system( /* SUMMARY */) // 여기!
        .user( /* message */)
        .call()
        .content();
  ```

### 2. 멀티테넌시(Multi-tenancy)
#### 1. 개념
- 여러 사용자의 데이터를 하나의 시스템에서 처리하되, 서로의 데이터가 섞이지 않도록 분리하는 구조
- 보통 userId를 기준으로 데이터를 조회하여 논리적으로 격리함.
- 이 구조가 없으면 다른 사용자의 대화 내용이 잘못 노출될 위험이 있음
- 서비스 확장성과 보안을 위해 반드시 고려해야 하는 기본 설계 요소
 
#### 2. 동작 흐름
- chat_conversations을 조회할때 특정 user기준으로 조회하도록 함.
- 그럼 ID값을 기준으로 연속적인 대화를 가질 수 있음.
 ```
  CREATE TABLE chat_conversations
  (
  id         UUID PRIMARY KEY      DEFAULT gen_random_uuid(),
  user ~     --여기에서 유저를 구분함.
  title ~     VARCHAR(255) NOT NULL,
  ~~~~~
 ```
 ```
  SELECT *
  FROM chat_messages
  WHERE user_id = :userId -- 특정 userId가 조회 될 수 있도록!
  AND conversation_id = :conversationId
 ```

