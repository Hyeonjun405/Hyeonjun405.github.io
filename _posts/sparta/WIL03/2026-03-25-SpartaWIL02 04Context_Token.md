---
title: 04 Context & Token
date: 2026-03-25 10:00:00 +09:00
categories: [Sparta, SpartaWIL03]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - Context
   - JAVA로 JVM의 메모리를 고려했다면
   - AI의 Context의 메모리를 고려해야함.
   - 안그러면 돈이 많이 나감.
 - 토큰
   - 주식의 증권이라고 보면 편할것 같음.
   - 특정 단위로 토큰이 존재하고, 이 토큰의 가격은 AI운영주최에 따라서 달라짐
   - 그래서 토큰 단위로 비용을 산정하고, 실제 금액은 그 시기의 정책에 따라 금액을 표기.
   - 예를 들어서
     - GPT에서 월 100만원 사용해서 확인하고자 할때
     - 토큰을 100개정도 사용했다고 하면
     - 제미나이나 기타 AI에서 100개에 얼마정도 나오는지 비교하면 됨.
 - 관심이나 집중도가 많이 바뀐듯함.
   - 기존에는 메모리의 효율성과 안전한 공간 등등이였는데
   - 데이터센터가 등장하면서 굳이 직접 인프라 환경을 구축하지않고
   - 외부 센터를 이용하면서 비용을 감축하는 방법을 채택하는 듯.

## 2. Context & Context Window
### 1. Context(컨텍스트)
  - 모델이 응답을 생성할 때 사용하는 전체 입력 정보
  - 이 입력은 단일 문장이 아니라 순서를 가진 토큰 시퀀스로 구성
  - 모델은 이 Context를 조건으로 다음 토큰의 확률을 계산
  - Context는 내부에 저장되는 상태가 아니라 매 요청마다 외부에서 전달되는 값
  - 따라서 어떤 Context를 주느냐에 따라 모델의 출력 결과가 직접적으로 달라짐

### 2. Context Window 
  - 모델이 한 번의 추론에서 처리할 수 있는 최대 입력 길이
  - 이 길이는 토큰 단위로 정의되며 모델 구조에 의해 결정
  - 모델은 이 범위 내에서만 입력을 받아 attention 연산을 수행할 수 있음
  - 입력이 Context Window를 초과하면 일부가 잘리거나 처리되지 않음
  - 따라서 Context Window는 모델이 사용할 수 있는 입력의 물리적 한계를 의미

### 3. Context의 3가지 핵심 역할
#### 1. 대화의 연속성 유지 (Conversation History)
 - 이전 대화 내용을 Context에 포함하여 전달받기 때문에 "내 이름이 뭐였지?" 같은 대명사나 생략된 질문을 이해함
 - Spring AI에서 ChatMemory / MessageWindowMemory 등 자동으로 넣어주는 기능도 있음
   - 자동으로 넣으면 통제가 X
   - 여러 전략을 통해 직접 넣는 방법을 채택하는 경우가 많음.
 - 직접 추가하는 패턴
 ```
  List<Message> conversationHistory = List.of(
      new UserMessage("안녕, 내 이름은 유진호라고 해."),    // 100 토큰
      new AssistantMessage("안녕하세요 유진호님!"),       // 50 토큰
      new UserMessage("내가 좋아하는 음식은 TACO야"),      // 80 토큰
      new AssistantMessage("TACO를 좋아하시는군요!"),     // 60 토큰
      new UserMessage("내 이름이 뭐였지?")                // 50 토큰
  );
  // 총 Context 사용량: 340 토큰 -> AI는 "유진호님입니다"라고 답변 가능
 ``` 

#### 2. 외부 지식 참조 (RAG 패턴)
 - RAG 패턴(라그패턴)
   - 외부 데이터를 검색해서, 그걸 기반으로 답을 생성하는 방식
   - 회사 내규 같은 텍스트 등은 인터넷에서 접근이 어려우니
   - 그런 데이터를 통으로 미리 알려주고
   - AI가 답변할때 그것을 바탕으로 대답할 수 있도록 함.
 - AI가 학습하지 않은 최신 정보나 특정 문서를 Context에 넣어주면, AI는 이를 '참고 자료'로 활용해 답변
 - 태그나 뭔가 표기를 통해서 문서와 질문을 잘구분해주는게 중요함
 ```
  @GetMapping("/ask")
  public String askWithDocument() {
      String document = "Spring AI는 Spring 생태계의 AI 통합 프레임워크입니다. " +
                        "주요 기능으로는 ChatClient, Vector Database 통합 등이 있습니다."; // 참고할 문서
      String userQuestion = "Spring AI의 핵심 기능은?";
  
      // 1. 프롬프트 템플릿 생성 (문서와 질문을 결합)
      String combinedPrompt = String.format(
          "아래 제공된 [문서] 내용을 바탕으로 질문에 답하세요.\n\n" +
          "[문서]\n%s\n\n" +
          "[질문]\n%s", 
          document, userQuestion
      );
  
      // 2. AI에게 전송
      return chatClient.prompt()
              .user(combinedPrompt)
              .call()
              .content();
  }
 ```
#### 3. 예시를 통한 학습 (Few-Shot Learning)
 - 질문을 던지기 전에 "질문-답변"의 예시를 몇 가지 Context에 넣어주면, 
 - AI는 그 패턴을 복사하여 훨씬 더 정확한 형식으로 답변
 ```
   public String askWithFewShot() {
      // 1. AI에게 학습시킬 '예시(Shot)'들을 구성
      String fewShotExamples = """
          질문: '사과'를 영어로 번역하고 짧은 예문을 만들어줘.
          답변: Apple (I like to eat a fresh apple in the morning.)
          
          질문: '바나나'를 영어로 번역하고 짧은 예문을 만들어줘.
          답변: Banana (Monkey is eating a yellow banana.)
          """;
  
      // 2. 실제 질문
      String userQuestion = "'수박'을 영어로 번역하고 짧은 예문을 만들어줘.";
  
      // 3. 예시 + 질문을 합쳐서 전달 (AI는 앞선 패턴을 그대로 복제함)
      String combinedPrompt = fewShotExamples + "\n질문: " + userQuestion + "\n답변:";
  
      return chatClient.prompt()
              .user(combinedPrompt)
              .call()
              .content();
      // 예상 답변: Watermelon (We enjoyed a sweet watermelon at the beach.)
   }
 ```

## 3. 토큰
### 1. 토큰의 종류
#### 1. 비교

  | 구분                | Prompt Tokens                   | Generation Tokens | Total Tokens |
  | ----------------- | ------------------------------- | ----------------- | ------------ |
  | 의미                | 입력 토큰                           | 출력 토큰             | 전체 토큰        |
  | 구성                | system + user + assistant + 데이터 | 모델이 생성한 응답        | 입력 + 출력      |
  | 역할                | 모델의 “조건”                        | 모델의 “결과”          | 전체 사용량       |
  | 제어 방법             | messages 구성                     | maxTokens로 제한     | 둘의 합         |
  | 비용 영향             | 있음                              | 있음                | 최종 비용        |
  | context window 영향 | 포함됨                             | 포함됨               | 반드시 제한 내     |

#### 2. Prompt Tokens (입력 토큰)
 - 모델에 입력으로 전달되는 모든 토큰
 - `System + User + Assistant + RAG 데이터`
 - 많을수록 문맥 이해가 좋아지지만 비용이 증가함

#### 3. Generation Tokens (출력 토큰)
 - 모델이 새로 생성한 출력 토큰
 - `.maxTokens()`로 제한 가능
 - 길면 비용이 증가, 짧으면 응답 끊김

#### 4. Total Tokens (전체 토큰)
 - `Total = Prompt Tokens + Generation Tokens`
 - 실제 과금 기준
 - Context Window 제한 기준 

### 2. 토큰의 최적화
#### 1. System Message 최적화
 - System Message는 모든 API 호출마다 포함되므로, 단 몇 줄을 줄이는 것만으로도 누적 절감 효과가 엄청남
 
 ```
  String systemMessage = """
    당신은 매우 친절하고 상냥하며 사용자를 배려하는 AI 어시스턴트입니다.
    사용자의 질문에 항상 정중하고 예의바르게 답변해야 하며,
    사용자가 이해하기 쉽도록 명확하고 간결하게 설명해주세요.
    또한 사용자의 감정을 고려하여 공감하는 태도를 보여주세요.
    전문적이면서도 친근한 톤을 유지하며,
    필요한 경우 예시를 들어 설명해주세요.
    """;
  // 요약
  String systemMessage = """
    친절한 AI 어시스턴트. 명확하고 간결하게 답변.
    """;
 ```

#### 2. 대화 히스토리 관리 전략 3가지
 - 슬라이딩 윈도우 (Sliding Window) / 메시지를 줄임.
   ```
    //질문과 답변 대화 내용이 들어있기 때문에 과거 메시지로 판단이됨.
    @Service
    public class SlidingWindowChatService {
        // 최대 유지할 메시지 수 (예: 최근 10개 질문/답변 쌍 = 20개 메시지)
        private static final int MAX_HISTORY_MESSAGES = 20; 
    
        public ChatResponse chat(String question, String conversationId) {
            List<Message> fullHistory = loadHistoryFromDB(conversationId);
    
            // 1. 최근 N개 메시지만 슬라이싱
            List<Message> recentHistory = getRecentMessages(fullHistory, MAX_HISTORY_MESSAGES);
    
            // 2. AI 호출 (최근 문맥만 포함)
            var response = chatClient.prompt()
                .messages(recentHistory)
                .user(question)
                .call()
                .chatResponse();
    
            return processResponse(response, fullHistory);
        }
    
        private List<Message> getRecentMessages(List<Message> history, int max) {
            if (history == null || history.size() <= max) return history;
            return history.subList(history.size() - max, history.size());
        }
    } 
   ```
 - 요약 기반 압축 (Summarization)
   ```
    @Service
    public class SummarizedChatService {
    private static final int SUMMARY_THRESHOLD = 30; // 30개가 넘어가면 압축 시작

    public ChatResponse chat(String question, String conversationId) {
        List<Message> history = loadHistoryFromDB(conversationId);

        if (history.size() > SUMMARY_THRESHOLD) {
            // 1. 오래된 대화(0~20번째)를 요약하여 교체
            // 메시지 발송전 요약 
            history = summarizeOldHistory(history);
        }

        return chatClient.prompt()
            .messages(history)
            .user(question)
            .call()
            .chatResponse();
    }

    private List<Message> summarizeOldHistory(List<Message> history) {
        List<Message> toSummarize = history.subList(0, 20);
        List<Message> keepAsIs = history.subList(20, history.size());

        // AI에게 요약 요청 (내부 프롬프트 활용)
        // 여기서 추가 토큰 발생함.
        String summary = chatClient.prompt()
            .user("다음 대화 내역을 200자 이내로 핵심만 요약해: " + convertToText(toSummarize))
            .call().content();

        List<Message> optimized = new ArrayList<>();
        optimized.add(new SystemMessage("이전 대화 요약: " + summary));
        optimized.addAll(keepAsIs);
        return optimized;
    }
   }
   ```
 - 중요도 기반필터링
   ```
    // 필요한 문구가 들어간 대화만 뽑아서 저장함
    // 주문과 관련된 대화면, <상품, 주소, 수량, 결제> 등등
    // 맥락이 끊어져서 이상해질 수 있음. 
    @Service
    public class SmartHistoryService {
    public List<Message> filterHistory(List<Message> history) {
    List<Message> important = history.stream()
    .filter(this::isImportant) // 핵심 로직
    .collect(Collectors.toList());
    
            // 문맥 유지를 위해 최소한의 최근 메시지(예: 10개)는 강제로 합침
            if (important.size() < 10) {
                important.addAll(getRecentMessages(history, 10));
            }
            return important.stream().distinct().toList();
        }
    
        private boolean isImportant(Message msg) {
            String content = msg.getText().toLowerCase();
            return content.contains("결제") || content.contains("주소") || content.length() > 100;
        }
    }  
   ```

#### 3. maxTokens 동적 설정 전략
 - 모든 질문에 큰 maxTokens를 할당하면 시스템 자원이 낭비되고 비용 예측이 어려워짐
 - 질문의 성격에 따라 필요한 만큼만 스마트하게 할당하는 것이 핵심
 - 예시 패턴
 
  | 질문 유형 | 의도 파악 키워드       | 권장 maxTokens | 예상 실제 사용 |
  | --- |-----------------| --- | --- |
  | 단답형 | 맞아, 인가요, 언제, 어디 | 100 | 50 ~ 100 |
  | 설명 요청 | 설명, 알려줘, 뭐야, 무엇 | 500 | 300 ~ 500 |
  | 코드 생성 | 코드, 구현, 작성, 프로그램 | 2,000 | 1,000 ~ 2,000 |
  | 긴 글 작성 | 에세이, 보고서, 써줘    | 3,000 | 2,000 ~ 3,000 |
  | 기본값 | 기타 일반 질문        | 1,000 | 500 ~ 1,000 |
