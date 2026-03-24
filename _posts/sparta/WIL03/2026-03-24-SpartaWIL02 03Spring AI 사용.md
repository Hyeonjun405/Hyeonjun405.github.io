---
title: 03 Spring AI 사용
date: 2026-03-24 10:00:00 +09:00
categories: [Sparta, SpartaWIL03]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - 메시지를 보내는 방식
   - 실제 웹 또는 AP에서 문자로 보내는 방식은 동일
   - 백엔드에서 하는것은 이것을 다듬어서 어떻게 최소한의 토큰으로 프롬프트를 전달하는가가 중요
   - 내용이 길어질수록 토큰이 비싸지고, 비용이 많이 발생함.
 - 프롬프트 엔지니어링에서 내용을 메소드로 분류를 하는 느낌

## 2. Spring 설정
### 1. build.gralde
  ```
  # 버전은 지속적으로 바뀌는 상황이라 확인은 필요.
  # dependencyManagement에 bom등록과 의존성 필요 
  dependencyManagement {
      imports {
          mavenBom "org.springframework.ai:spring-ai-bom:1.0.0-M6"
      }
  }
  
  dependencies {
      // Google AI Gemini 스타터 추가
      // spring AI 1.1 이상에서는 OpenAI뿐만 아니라 Gemini, Claude 등 다른 LLM도 동일한 API 구조로 연결 가능
      implementation 'org.springframework.ai:spring-ai-openai-spring-boot-starter'
  }
  ```

### 2. application.yaml
 ```
 spring:
  ai:
    openai:
      # 1. API 키: Google AI Studio에서 발급받은 키를 입력합니다.
      # ${GOOGLE_AI_GEMINI_API_KEY}
      # 키값은 AP에서 
      api-key: GOOGLE_AI_GEMINI_API_KEY 
      chat:
        # 2. Base URL: Google Gemini가 제공하는 OpenAI 호환 API 주소입니다.
        base-url: "https://generativelanguage.googleapis.com/v1beta/openai/"
        options:
          # 모델명: 현재 가장 최신/경량 모델인 gemini-2.0-flash-lite 등을 지정합니다.
          model: "gemini-2.5-flash-lite"
          # 온전성(Temperature): 0.0은 가장 보수적이고 사실적인 답변을 생성합니다.
          # (분석, 요약, 데이터 추출에 최적화된 설정)
          temperature: 0.7
          # 최대 토큰 수: AI가 생성할 답변의 최대 길이를 제한합니다.
          max-tokens: 4096
        # 4. 엔드포인트 경로: 대화형 API를 호출하기 위한 표준 경로입니다.
        completions-path: "/chat/completions"
 ```

### 3. 기본 메시지 형태
 ```
  @GetMapping("/type1")
  public String generateMarketing() {  
    return chatClient.prompt()   // 프롬프트 생성 시작
        .user("메시지")          // 사용자 입력 설정
        .call()                  // 실제 LLM 호출
        .content();              // 결과 텍스트 추출
  } 
 ```


## 3. 메시지 구분
### 1. System / User / Assistant

| 구분    | System         | User        | Assistant      |
| ----- | -------------- | ----------- |----------------|
| 역할    | AI의 행동/규칙 정의   | 사용자 요청      | AI의 응답을 기록     |
| 의미    | “어떻게 답할지” 결정   | “무엇을 물어보는지” | “이전에 어떻게 답했는지” |
| 우선순위  | 가장 높음          | 중간          | 낮음             |
| 사용 목적 | 톤, 스타일, 정책 설정  | 질문, 명령      | 대화 흐름 유지       |
| 사용 시점 | 보통 최초 1회 또는 고정 | 매 요청마다      | 히스토리 포함 시      |
| 영향 범위 | 전체 응답에 지속 영향   | 해당 요청       | 다음 응답에 간접 영향   |
| 없을 경우 | 기본 모델 성향 사용    | 요청 자체가 없음   | 단일 응답 구조       |


### 2. 메소드 구분
#### 1. 편의성
 ```
  chatClient.prompt()
    .system("너는 백엔드 전문가야")
    .user("JPA 설명해줘")
 ```

#### 2. 편의성메소드의 실제 모습
 ```
  chatClient.prompt()
    .messages(
        new SystemMessage("너는 백엔드 전문가야"),
        new UserMessage("JPA 설명해줘")
    )
 ```

## 4. AI 메시지 전송 관련 설정(System / User)
### 1. message 템플릿
 ```
  @GetMapping("/marketing")
  public String generateMarketing(
      @RequestParam(value = "productName") String productName,
      @RequestParam(value = "features") String features) {

    // 메시지 템플릿을 주게되면
    String template = """
        제품명 {productName}의 마케팅 문구를 작성하세요.
        주요 특징: {features}
        조건: 감성적이고 100자 이내로 작성할 것.
        """;

    return chatClient.prompt()
        .user(u -> u.text(template) // 프롬프트에 규칙 또는 정해진 메시지 형태를 줄 수 있음.
            .param("productName", productName)
            .param("features", features))
        .call()
        .content();
  }
 ```

### 2. 역할 제공
 ```
   @GetMapping("/translate")
  public String translate(
      @RequestParam(value = "text") String text,
      @RequestParam(value = "targetLanguage", defaultValue = "영어") String targetLanguage) {

    return chatClient.prompt()
        // 1. AI의 페르소나 설정 (System Message)
        .system("당신은 전문 번역가입니다. 주어진 텍스트를 문맥에 맞게 자연스럽게 번역해주세요.")

        // 2. 동적 파라미터 주입 (Prompt Template)
        .user(u -> u.text("다음 텍스트를 {lang}로 번역해주세요: {text}")
            .param("lang", targetLanguage)
            .param("text", text))
        .call()
        .content();
  } 
 ```

## 5. Assistant (이전 히스토리를 담기)
### 1. 대화 방식
 - AI내부에 기존의 대화를 기억하는 방법은 없고, 직접 대화를 넣어줘야만 기억을 할 수 있는 패턴이 됨
 - 대화의 흐름
  ```
   1회차 요청: `[System]` + `[User 1]` → 응답: `[Assistant 1]`
   2회차 요청: `[System]` + `[User 1]` + `[Assistant 1]` + `[User 2]` → 응답: `[Assistant 2]`
   3회차 요청: `[System]` + `[User 1]` + `[Assistant 1]` + `[User 2]` + `[Assistant 2]` + `[User 3]` → 응답: `[Assistant 3]`
  ```

### 2. 소스패턴
  ```
  public class ChatService {
  
      private final ChatModel chatModel;
  
      public void startConversation() {
          // 1. AI의 정체성 설정 (System)
          Message systemMessage = new SystemMessage("당신은 요리 전문가입니다.");
  
          // 2. 사용자의 첫 번째 질문 (User)
          Message userMessage1 = new UserMessage("김치찌개 맛있게 만드는 비법이 뭐야?");
  
          // 3. AI의 이전 응답 (Assistant) - 이 내용이 있어야 AI가 과거 답변을 기억함
          Message assistantMessage = new AssistantMessage("김치찌개의 비법은 충분히 볶은 김치와 쌀뜨물입니다.");
  
          // 4. 사용자의 두 번째 질문 (User) - 맥락이 필요한 질문
          Message userMessage2 = new UserMessage("그럼 된장찌개는?");
  
          // 5. 전체 메시지를 리스트로 묶어서 전송
          // AI는 assistantMessage를 보고 "아, 김치찌개 비법을 알려줬으니 이번엔 된장찌개 비법을 묻는구나"라고 이해합니다.
          List<Message> history = List.of(
              systemMessage, 
              userMessage1, 
              assistantMessage, 
              userMessage2
          );
  
          chatModel.call(new Prompt(history));
      }
  }
  ```

## 6. Entity 파싱
### 1. Response 파싱
 ```
 @Getter
 @NoArgsConstructor // JSON 역직렬화를 위해 기본 생성자 필수
 @FieldDefaults(level = AccessLevel.PRIVATE)
 public class ProductAnalysisResponse {

    @JsonProperty("sentiment")
    String sentiment;

    @JsonProperty("score")
    int score;

    @JsonProperty("summary")
    String summary;
 }
 ```

### 2. Controller
 ```
   @GetMapping("/analyze")
   public ProductAnalysisResponse analyzeReview(@RequestParam(value = "review") String review) {

    String promptText = """
        다음 제품 리뷰를 분석해주세요:
        
        리뷰 내용: {review}
        
        요구사항:
        1. sentiment는 positive, neutral, negative 중 하나로 응답하세요.
        2. score는 1점에서 10점 사이의 정수로 응답하세요.
        3. summary는 분석 내용을 한 문장으로 요약하세요.
        """;

    return chatClient.prompt()
        .user(u -> u.text(promptText).param("review", review))
        .call()
        .entity(ProductAnalysisResponse.class); // 객체로 자동 변환
  } 
 ```

### 3. 응답
 ```
 {
  "sentiment": "negative",
  "score": 1,
  "summary": "제품에서 벌레가 나와 섭취하지 못했다는 부정적인 리뷰입니다."
 }
 ```

## 7. 옵션방식
### 1. 옵션 패턴

```
  @GetMapping("/story")
  public String generateStory(@RequestParam(value = "topic") String topic) {
    return chatClient.prompt()
        .user("다음 주제로 창의적인 이야기를 작성해주세요: " + topic)
        .options(ChatOptions.builder()
            .temperature(0.9)  // 1.0에 가까울수록 창의적(랜덤성 증가)
            .maxTokens(500)    // 답변의 최대 길이 제한
            .build())
        .call()
        .content();
  }
```

### 2. 활용 가능한 메소드(주요)

| 파라미터             | 정의             | 영향 대상    | 값 범위       | 효과                    | 언제 사용       |
| ---------------- | -------------- | -------- | ---------- | --------------------- | ----------- |
| temperature      | 출력 랜덤성(창의성) 조절 | 단어 선택 확률 | 0.0 ~ 1.0+ | 낮으면 일관성 ↑ / 높으면 다양성 ↑ | 창의 vs 정확 조절 |
| topP             | 확률 상위 비율 제한    | 단어 후보군   | 0.0 ~ 1.0  | 낮으면 안전 / 높으면 다양       | 자연스러운 생성    |
| maxTokens        | 최대 출력 길이 제한    | 응답 길이    | 정수         | 길면 잘림 / 짧으면 빠름        | 비용/응답 길이 제어 |
| frequencyPenalty | 반복 단어 억제       | 동일 단어    | 0.0 ~ 2.0  | 반복 감소                 | 리스트/중복 방지   |
| presencePenalty  | 새로운 단어 유도      | 단어 다양성   | 0.0 ~ 2.0  | 새로운 내용 증가             | 창의적 생성      |
