---
title: 07 Spring AI 이미지
date: 2026-03-26 10:00:00 +09:00
categories: [Sparta, SpartaWIL03]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - 이미지를 분석하거나 파악하는 용도에 가까운듯
   - 사용하는 방법이나 요령은 조금더 찾아봐야할 듯 하고
   - 예시로 보여준 것은 이미지 여러개를 던지고 이미지에 텍스트 출력
   - 이것도 뭔가 잘만하면 분석쪽에서 활용할 수 있을지도 ?


## 2. 멀티모달 (Multimodal)
### 1. 멀티모달
 - 텍스트, 이미지, 오디오, 비디오 등 서로 다른 형태의 데이터를 동시에 입력받아 관계를 이해하고 추론하는 AI 기능
 - 패턴비교
   - 전통적인 AI (Single-modal): 텍스트 입력 : 텍스트 출력 (텍스트 전용 모델)
   - 멀티모달 (Native Multimodal): 입력: 텍스트 + 이미지 + 오디오 + 비디오 + 코드 등 복합 입력
 - 지원하는 모달리티
   - 텍스트 (Text): 방대한 문서 분석 및 대화
   - 이미지 (Image): 고해상도 사진 및 그래픽 분석
   - 오디오 (Audio): 녹음 파일, 음악, 음성 대화 직접 이해
   - 비디오 (Video): 최대 수 시간 분량의 영상 흐름 및 특정 장면 파악

### 2. 주요멀티모달 AI 비교

| 모델                              | 입력 (Input)           | 출력 (Output)  | 특징            | 실무 포인트        |
| ------------------------------- | -------------------- | ------------ | ------------- | ------------- |
| OpenAI (GPT-4o 계열)              | 텍스트, 이미지, 음성, 영상     | 텍스트, 이미지, 음성 | 진짜 “올인원 멀티모달” | 가장 범용적        |
| Google (Gemini)                 | 텍스트, 이미지, 음성, 영상, 문서 | 텍스트, 이미지     | 문서 이해 강함      | PDF, 코드 분석 강점 |
| Anthropic (Claude)              | 텍스트, 이미지, 문서         | 텍스트          | 긴 컨텍스트 최강     | 대용량 분석        |
| Meta (LLaMA 계열)                 | 텍스트 (확장 시 이미지)       | 텍스트          | 오픈소스          | 커스터마이징        |
| Stability AI (Stable Diffusion) | 텍스트, 이미지             | 이미지          | 이미지 생성 특화     | 디자인, 생성       |


## 3. 모달 구현
### 1. Request
 ```
  @Getter
  @FieldDefaults(level = AccessLevel.PRIVATE)
  public class ImageAnalysisRequest {
    String message;
  
    //멀티파트파일 타입
    MultipartFile image;
  
  }
 ```

### 2. Response
 ```
 public class ImageAnalysisResponse {

  String analysis;

  String imageType; // 이미지 타입

  Long imageSize; // 사이즈 타입

  TokenUsage tokenUsage;

  @Getter
  @Builder
  @FieldDefaults(level = AccessLevel.PRIVATE)
  public static class TokenUsage {

    Integer promptTokens;

    Integer completionTokens;

    Integer totalTokens;
  }
 ```

### 3. Service(☆)
 ```
  public ImageAnalysisResponse analyzeImage(String message, MultipartFile image) throws IOException {

    String contentType = image.getContentType();
    if (contentType == null) {
      contentType = "image/jpeg";
    }

    try {
      String finalContentType = contentType;
      
      // repsponse 받을때
      // 유저에서 media로 추가해야함 (☆☆☆☆)
      // 그외에 발송하는 패턴은 전부 동일함
      // 기존에 text로 말송하던 것에 추가로 media로 추가하는 것!
      var response = chatClient.prompt()
          .user(u -> u
              .text(message)
              .media(MimeTypeUtils.parseMimeType(finalContentType), image.getResource())
          )
          .call()
          .chatResponse();

      String analysis = response.getResult().getOutput().getText();
    ~~~~~~~~~~~~
     
 ```

### 4. Controller
 ```
  // Mapping에 추가 인자로 consumes 추가 필요!
  @PostMapping(value = "/analyze", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
  public ApiResponse<ImageAnalysisResponse> analyzeImage(
      @RequestParam String message,
      @RequestParam MultipartFile image) throws IOException {
    return ApiResponse.ok(visionChatService.analyzeImage(message, image));
  }
 ```
