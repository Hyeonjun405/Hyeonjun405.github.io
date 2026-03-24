---
title: 02 Spring AI 개념
date: 2026-03-24 10:00:00 +09:00
categories: [Sparta, SpartaWIL03]
tags: [ Java, Spring Framework ]
---

## 1. Note
- 스프링이 어느정도 익숙하면 굉장히 편함
  - 기존 라이브러리 추가 와 빈을 사용하는 형태랑 굉장히 비슷하게 사용
  - 그래서 의존성에서 필요한 빈들만 알면 쉽게 접근을 할 수 있을 것 같음.
  - 원래 운영중이던 서비스에서 필요한 부분만 추가로 기능만 달아서 사용도 가능함.
  - 역할분리가 굉장히 잘 되어 있다면 매우 좋을 듯!
- 근데 이걸 어떻게 써먹어야하지?

## 2. Spring AI
### 1. Spring AI
- 스프링 기반 애플리케이션에서 AI 기능을 쉽게 사용할 수 있도록 만든 라이브러리
- 여러 AI 모델을 공통 인터페이스로 추상화해 일관된 방식으로 호출가능함.
- 프롬프트 작성, 응답 처리, 외부 데이터 연계(RAG) 같은 기능을 기본 제공해 개발 부담을 줄임
- 스프링의 DI, 설정 방식과 자연스럽게 통합되어 기존 구조를 그대로 활용할 수 있음
- 결과적으로 AI 기능을 하나의 스프링 컴포넌트처럼 다루게 해주는 역할

### 2. Spring AI 주요 특징
- 모델 추상화 (Model Abstraction)
  - 각 AI 제조사마다 제각각인 API 호출 방식을 통일된 인터페이스로 제공
  - Benefit: 코드 한 줄 수정으로 모델을 교체할 수 있습니다.
  - 예시: 테스트 때는 비용이 저렴한 `GPT-3.5`를 쓰다가, 배포 시 `GPT-4o`나 `Claude`로 바꿔도 비즈니스 로직은 변하지 않습니다.
   
- Spring 생태계 완벽 통합
  - Spring 프레임워크의 핵심 기능을 AI 개발에 그대로 적용
  - 의존성 주입 (DI): `ChatClient` 등을 빈(Bean)으로 등록해 어디서든 주입받아 사용
  - AOP 활용: AI 호출 전후의 로깅, 보안, 트랜잭션 처리를 선언적으로 관리
  - Spring Boot Starter: 복잡한 라이브러리 설정 없이 의존성 추가만으로 바로 시작
 
- 선언적 설정 (Declarative Configuration)
  - Java 코드로 API 키나 파라미터를 일일이 설정할 필요가 없음.
  - `application.yml` 파일에서 모든 것을 관리

  ```
   spring:
     ai:
       openai:
         # 1. API 키: Google AI Studio에서 발급받은 키를 입력합니다.
         # ${GOOGLE_AI_GEMINI_API_KEY}
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
  
- 확장성 및 벡터 데이터베이스 지원
  - Vector Store 지원: Pinecone, Redis, PostgreSQL 등 다양한 벡터 DB를 표준화된 인터페이스로 지원
  - RAG(검색 증강 생성) 구현: 외부 지식을 AI에게 전달하는 복잡한 파이프라인을 쉽게 구축할 수 있도록 설계
   
  | 구분 | 도입 전 (직접 연동) | 도입 후 (Spring AI) |
  | --- | --- | --- |
  | 코드 가독성 | HTTP 클라이언트로 복잡한 JSON 처리 | `ChatClient` 인터페이스 호출 |
  | 모델 교체 | API 명세가 달라 대대적인 코드 수정 필요 | 설정값(Model Name) 변경만으로 완료 |
  | 데이터 연동 | 벡터 DB 연동 로직 직접 구현 | 표준 인터페이스(`VectorStore`) 사용 |
  | 유지 보수 | 각 제조사 SDK 업데이트마다 대응 필요 | Spring AI 업데이트만으로 최신 기능 유지 |


## 3. Spring + AI이점
### 1. 보안 및 자산 보호
- API Key를 서버 내부(환경 변수, Secrets Manager)에 숨김
- 사용자에게는 절대 노출되지 않음
    
   ```
   @Service
   public class ChatService {
        
     @Value("${spring.ai.openai.api-key}")
     private String apiKey; // 서버 외부로 절대 유출되지 않음
    	
   }
   ```
   
### 2. 정교한 비용 및 사용량 관리
- 사용자별 일일 호출 횟수를 제한하여 예기치 못한 비용 지출을 방지
    
   ```
   public String chatWithLimit(String userId, String message) {
       if (usageService.getTodayUsage(userId) >= 10) {
           throw new UsageLimitException("오늘 사용량을 모두 소진했습니다.");
       }
       return chatClient.prompt().user(message).call().content();
   }
   ```

### 3. 개인화된 비즈니스 로직
- DB에 저장된 사용자 프로필, 구매 이력 등을 프롬프트에 결합하여 "나만을 위한 답변"을 생성
- 예: "이 고객은 30대 남성이고 최근 등산화를 구매했어. 이 정보를 바탕으로 상품을 추천해줘."
    
   ```
   @Service
   public class ProductRecommendationService {
        
       @Autowired
       private UserRepository userRepository;
        
       @Autowired
       private ProductRepository productRepository;
        
       public String recommendProduct(Long userId, String query) {
           // 1. DB에서 사용자 정보 조회
           User user = userRepository.findById(userId)
                   .orElseThrow(() -> new UserNotFoundException());
            
           // 2. 사용자 구매 이력 조회
           List<Product> purchaseHistory = productRepository
                   .findByUserId(userId);
            
           // 3. 개인화된 프롬프트 생성
           String prompt = String.format("""
               다음 사용자에게 상품을 추천해주세요:
               - 연령대: %s
               - 관심사: %s
               - 구매 이력: %s
               - 질문: %s
               """, 
               user.getAgeGroup(),
               user.getInterests(),
               purchaseHistory.stream()
                   .map(Product::getName)
                   .collect(Collectors.joining(", ")),
               query
           );
            
           // 4. AI 호출
           return chatClient.prompt()
                   .user(prompt)
                   .call()
                   .content();
       }
   }
   ```

### 4. 응답 품질 관리 및 모니터링
- 사용자가 대충 질문해도 백엔드에서 '전문가 페르소나'를 입혀 높은 품질의 답변을 유도
- AI가 답한 내용에 부적절한 표현이 있는지 검증한 뒤 사용자에게 전달가능

   ```
   @Service
   public class ManagedChatService {
        
       public String chat(String message) {
           // 1. 입력 검증
           if (containsInappropriateContent(message)) {
               return "부적절한 내용이 포함되어 있습니다.";
           }
            
           // 2. 프롬프트 템플릿 적용 (일관된 응답 품질)
           String enhancedPrompt = """
               당신은 전문적이고 친절한 고객 지원 AI입니다.
               다음 규칙을 따르세요:
               1. 존댓말 사용
               2. 3문장 이내로 답변
               3. 확실하지 않으면 "정확한 답변을 드리기 어렵습니다"라고 답변
                
               사용자 질문: %s
               """.formatted(message);
            
           try {
               // 3. AI 호출
               String response = chatClient.prompt()
                       .user(enhancedPrompt)
                       .call()
                       .content();
                
               // 4. 로깅 (모니터링)
               log.info("AI 호출 - 입력 토큰: {}, 출력 토큰: {}, 비용: ${}",
                       inputTokens, outputTokens, cost);
                
               return response;
                
           } catch (Exception e) {
               // 5. 에러 처리
               log.error("AI 호출 실패", e);
               return "죄송합니다. 일시적인 오류가 발생했습니다.";
           }
       }
   }
   ```

### 5. 성능 최적화 (캐싱)
- 자주 묻는 질문은 AI를 호출하지 않고 캐시(Cache)에서 즉시 반환하여 비용 절감과 속도 향상을 동시에 잡음

 ```
 @Service
 public class CachedChatService {
     
     private final Cache<String, String> cache = 
         Caffeine.newBuilder()
             .maximumSize(1000)
             .expireAfterWrite(1, TimeUnit.HOURS)
             .build();
     
     public String chat(String message) {
         // 동일한 질문은 캐시에서 반환 (비용 절감)
         return cache.get(message, key -> {
             return chatClient.prompt()
                     .user(key)
                     .call()
                     .content();
         });
     }
 }
 ```
