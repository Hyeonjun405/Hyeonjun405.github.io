---
title: 04 Advisor
date: 2026-04-02 10:00:00 +09:00
categories: [Sparta, SpartaWIL04]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - AOP/인터셉터와 같이 Spring AI를 사용할때 앞뒤로 넣는 작업
   - 이러면 굳이 노가다할 필요가 없었네...

## 2. Advisor
### 1. 개념
 - Advisor는 LLM 호출 전후를 가로채서 요청과 응답을 가공하는 구조
 - 프롬프트에 공통 지침을 추가하거나, 응답을 후 처리하는 역할 - 인터셉터나 AOP 같은
 - 여러 개를 체인처럼 연결해서 순서대로 실행할 수 있음 - 체인
 - RAG처럼 문서를 검색해 붙이는 것도 Advisor로 구현됨
 - LLM 로직과 부가 기능을 분리해주는 확장 포인트
 
### 2. 주요 용도
 - 공통 system prompt 넣기
 - RAG 붙이기 (문서 검색)
 - 토큰 로그 찍기
 - 응답 후처리 (파싱, 필터링 등)

### 3. 주요 흐름
- 사용자 질문
  - 사용자가 call()을 호출하면, 질문과 함께 Prompt 객체가 생성
  - Advisor 체인을 타기 위한 준비 상태가 됨

- Advisor 1 (Before) / 1번
  - 가장 바깥쪽 Advisor가 먼저 실행되며, 공통 정책을 주입
  - 예: "한국어로 답변해라", "회사 정책을 준수해라" 같은 system 메시지 추가

- Advisor 2 (Before) / 2번
   - 다음 Advisor에서 동적으로 컨텍스트를 확장
   - 예: 벡터 DB에서 유사 문서를 조회 → Prompt에 추가 → RAG 수행

- LLM 호출
   - 모든 Advisor의 전처리가 끝난 최종 Prompt가 LLM에 전달
   - 모델이 실제로 답변을 생성

- Advisor 2 (After) / 2번
   - LLM 응답을 받은 직후, 내부 처리용 로직 수행
   - 예: 토큰 사용량 로깅, 응답 포맷 검증, JSON 파싱 등

- Advisor 1 (After) / 1번
   - 가장 바깥쪽으로 돌아오면서 최종적인 후처리 수행
   - 예: 응답 저장, 감사 로그 기록, 추가 가공

- 최종 응답:
  - 모든 Advisor 체인을 통과한 결과가 사용자에게 반환


## 3. Advisor 구현 방법
### 1. 챗클라이언트
```
@Bean
public ChatClient chatClient(ChatModel chatModel, RagAdvisor ragAdvisor) {
    return ChatClient.builder(chatModel)
        // 여기에 추가하는 경우 전역으로 사용됨
        // 만약에 각기 서비스단에서 하면 그 서비스단에만 적용
        // 여기에서는 셋팅만 되고 실행X, 서비스단에서 call하면 그 전후로 실행  
        .defaultAdvisors(ragAdvisor) 
        .build();
}
```

### 2. Advisor 구현
- 패턴
  ```
  @Component
  public class RagAdvisor extends BaseAdvisor {
  
      private final VectorStore vectorStore;
  
      public RagAdvisor(VectorStore vectorStore) {
          this.vectorStore = vectorStore;
      }
  
      @Override
      protected void before(CallContext context) {
  
          // 1. 사용자 질문 추출
          String question = context.getPrompt().getUserMessage();
  
          // 2. 공통 정책 추가 (CAG 느낌)
          context.getPrompt().addSystemMessage("모든 답변은 한국어로 작성해줘.");
  
          // 3. RAG - 유사 문서 검색
          List<Document> docs = vectorStore.similaritySearch(question);
  
          String ragContext = docs.stream()
              .map(Document::getText)
              .collect(Collectors.joining("\n"));
  
          // 4. 프롬프트에 컨텍스트 추가
          context.getPrompt().addSystemMessage("다음 문서를 참고해서 답변해:\n" + ragContext);  
      }
  
      @Override
      protected void after(CallContext context, ChatResponse response) {
  
          // 5. 응답 후처리 (로깅)
          String answer = response.getResult().getOutput().getContent();
          System.out.println("응답 결과: " + answer);
      }
    
    // 값이 같을 경우 호출 순서로 작동함.  
    @Override
    public int getOrder() {
      return 0;
    }
  }
  ```
  
- context.getPrompt()

  | 조작 예시                            | 용도           | 내용/설명                            |
  | -------------------------------- | ------------ | -------------------------------- |
  | `addSystemMessage("규칙 추가")`      | LLM 행동 지침 설정 | 시스템 메시지로 “모든 답변은 한국어로”처럼 규칙을 전달  |
  | `addUserMessage("질문")`           | 사용자 입력 추가    | 현재 사용자 질문을 prompt에 포함시키기 위해 사용   |
  | `addAssistantMessage("참고 답변")`   | 이전 답변 참고     | 이전 LLM 응답을 포함시켜 문맥 유지            |
  | `setInstructions(List<Message>)` | 프롬프트 전체 교체   | 과거 대화 + 현재 질문 등을 모두 합쳐 새 프롬프트 구성 |
  | `clearMessages()`                | 프롬프트 초기화     | 새 주제로 완전히 시작할 때 기존 메시지 제거        |
  | `getInstructions()`              | 현재 메시지 확인    | Advisor 로직에서 기존 메시지 읽고 조건 처리     |


### 3. 서비스단
```
@Service
public class ChatService {

    private final ChatClient chatClient;
    private final RagAdvisor ragAdvisor;

    public ChatService(ChatClient chatClient, RagAdvisor ragAdvisor) {
        this.chatClient = chatClient;
        this.ragAdvisor = ragAdvisor;
    }

    public String ask(String question) {
        return chatClient.prompt()
            .user(question) // 유저 질문을 담고
            .call() // 이거 다음부터 공통 advisor가 진행하고, 답변하면 after진행
            .content(); // 최종 컨텐츠 생성
    }
}
```

## 4. 내장 Advisor
### 1. Memo
 - 굉장히 빠르게 변화하고 있음(2026.04.01.)
 - 정말 실시간으로 업데이트되면서 바뀌고 있어서 픽스로 이해하기보다는
   - 상황에 따라서 지금 시점에서 필요했던 것들을 파악하고
   - 추후에 필요한게 잇는지 확인하고 있으면 쓰고 없으며 직접 구현 하는 방향이 낫을 것 같음
   - 어드바이저의 사용방법에 집중하는게 유리함.
 - 결과적으로 기존의 어드바이스 정보를 바탕으로 직접 만드는 상황이 많음.
  
### 2. 주요 내장 Advisor(2026.04.01.) 

| Advisor                  | 주요 역할        | 동작 시점          | 사용 목적            | 장점              | 단점         | 사용 추천 상황          |
| ------------------------ | ------------ | -------------- | ---------------- | --------------- | ---------- | ----------------- |
| QuestionAnswerAdvisor    | RAG (문서 검색)  | Before         | 외부 지식 기반 답변      | 구현이 매우 간단       | 검색 품질 의존   | 문서 기반 QA, 사내 지식봇  |
| MessageChatMemoryAdvisor | 대화 메모리 유지    | Before         | 멀티턴 대화 유지        | role 기반으로 구조 명확 | 토큰 증가 빠름   | 챗봇, 상담 시스템        |
| PromptChatMemoryAdvisor  | 메모리 (텍스트 기반) | Before         | 단순 히스토리 유지       | 구현 단순           | role 구조 깨짐 | 간단한 대화 기능         |
| SimpleLoggerAdvisor      | 요청/응답 로깅     | Before + After | 디버깅, 로그 확인       | 바로 확인 가능        | 운영환경 부적합   | 개발/테스트 환경         |
| SafeGuardAdvisor         | 유해 요청/응답 차단  | Before + After | 정책 위반 방지, 안전성 확보 | 보안/리스크 감소       | 과도한 차단 가능  | 외부 사용자 서비스, 운영 환경 |


### 3. SafeGuardAdvisor(기억해두면 좋은)
 - 유해하거나 위험한 요청/응답을 차단하는 안전 필터
 - 특정 String 등이 포함되면 **처리하거나 질문 자체를 막는 등의 작업을 함.
 - 작업
   - 질문이 위험하면 → LLM 호출 자체를 막음 (Before)
   - 응답이 위험하면 → 결과를 차단하거나 수정 (After)
   - 운영 환경에서 필수로 쓰는 보안용 Advisor
   
