---
title: 05 FunctionCalling
date: 2026-04-03 10:00:00 +09:00
categories: [Sparta, SpartaWIL04]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - Tools이 많을수록 토큰이 증가하고 LLM이 선택을 실수할 확률이 높아짐
   - 따라서 별도로 필요한 것만 잘 분류를 해서
   - 프롬프트 보낼때 사용가능성 있는 것만 체크해서 보내는게 중요할듯 하고
   - description에 명확히 명시해줘서
   - LLM이 잘 선택할수 있도록 셋팅해주는 것이 중요    
 - Tool에 사용된 JPA는 고민필요
   - Read만 가능하게 조정필요 @transactional(readOnly = true)
   - 어떻게 연관관계 가져갈지 고민 필요함.
 - 체력관리! 체력관리!

## 2. FunctionCalling
### 1. FunctionCalling
 - LLM이 사용자의 의도를 분석해 적절한 외부 함수를 호출하고,
 - 결과를 받아 후속 처리까지 자동으로 수행할 수 있게 해주는 기능

### 2. 특징
- 정교한 함수 선택
  - 사용자의 질문이나 요청을 분석하여, 수많은 함수 중 가장 적합한 것을 정확하게 선택
  - 이를 통해 모델이 단순 추측이 아니라 의도에 맞는 기능을 호출
- 정형 파라미터 추출
  - 자연어로 전달된 정보에서 함수 실행에 필요한 값들을 JSON 스키마 형태로 추출
  - 예를 들어 날짜, 위치, 수치 등 다양한 인자를 정확하게 매핑할 수 있음
- 멀티모달 연동
  - 텍스트뿐 아니라 이미지, 영상, 오디오 등 다양한 데이터 형태를 이해함
  - 이를 기반으로 적절한 함수를 호출할 수 있음
  - 예를 들어 사진 분석 후 관련 API를 자동 호출하는 경우가 있음
- 병렬 및 연쇄 호출
  - 하나의 요청에서 여러 함수를 동시에 호출하거나,
  - 특정 함수 실행 결과를 다음 함수 호출에 연계하는 식으로 복잡한 작업을 처리할 수 있음
  - 이를 통해 효율적인 워크플로우 구현이 가능
- 엔터프라이즈 통합
  - Google Cloud(Vertex AI) 같은 클라우드 환경에서 인증, 권한, 보안 설정이 강화된 상태로 함수 호출을 지원
  - 조직 내부 도구와 안전하게 연동 가능하여 실무 적용에 적합함

## 3. 사용
### 1. Tool 정의
- 소스 정의
```
public class FunctionTools {

  @Tool(description = "특정 도시의 현재 날씨 정보를 조회합니다")
  public WeatherResponse getWeather(WeatherRequest request) {
    log.info("날씨 조회: {}", request.getCity());

    Random random = new Random();
    int temperature = 10 + random.nextInt(20);
    String[] conditions = {"맑음", "흐림", "비", "눈"};
    String condition = conditions[random.nextInt(conditions.length)];

    return WeatherResponse.builder()
        .city(request.getCity())
        .temperature(temperature)
        .condition(condition)
        .timestamp(LocalDateTime.now())
        .build();
  }
  ~~~
}
```

- Memo
  - 설명 작성
    - LLM이 사용자의 의도를 파악해 올바른 함수를 선택하는 정확도가 높아짐
    - 다른 개발자가 코드를 볼 때 함수 역할을 쉽게 이해할 수 있음
  - 설명 미작성 
    - 함수 호출 자체는 정상적으로 동작
    - 다만 LLM이 함수 선택을 하는 과정에서 일부 혼동이 생길 수 있음

### 2. Service
```
@RequiredArgsConstructor
public class FunctionCallingService {

  private final ChatClient.Builder clientBuilder;
  private final FunctionTools functionTools; // 주입
  
  /**
   * 기본 Function Calling - 모든 도구 사용 가능
   */
  public String chat(String userMessage) {
    log.info("[Chat] User Message: {}", userMessage);
    try {
      return clientBuilder.build()
          .prompt()
          .user(userMessage)
          .tools(functionTools) // 이렇게 사용
          .call()
          .content();
    } catch (Exception e) {
      log.error(e.getMessage(), e);
      throw new DomainException(DomainExceptionCode.AI_RESPONSE_ERROR);
    }
  }
```
