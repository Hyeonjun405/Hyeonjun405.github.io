---
title: 09 Spring AI&Ollama
date: 2026-03-27 10:00:00 +09:00
categories: [Sparta, SpartaWIL03]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - 생각보다 사양이 많이 필요한 것 같음.
   - 지금 16GB에 GPU가 없는 상태에서 노트북에서 버티지를 못함.
   - 발열 줄여가면서 가볍게 사용하면 살짝 살짝 되기는 하는데
   - 애매한듯함.
 - Ollama 는 OS처럼 뭔가 실행을 도와주는 도구
   - 모델은 직접 선택해서 사용해야함.
   - 어떠한 것을 사용해야할까?


## 2. Ollama
### 1. Ollama
 - 로컬 환경에서 **대형 언어 모델(LLM)을 쉽게 실행할 수 있도록 도와주는 런타임 도구**
 - 별도의 복잡한 설정 없이 모델을 다운로드하고 CLI 또는 API 형태로 바로 사용할 수 있음
 - Spring AI 등과 연동하면 서버 애플리케이션에서 로컬 AI를 호출하는 구조를 간단하게 구성할 수 있음
 - 외부 API 없이도 동작하기 때문에 비용 절감과 데이터 보안 측면에서 유리

### 2. Model

| 모델             | 특징                   | 추천 용도          |
| -------------- | -------------------- | -------------- |
| Llama 3        | 성능·안정성 균형 좋음, 가장 범용적 | 기본 챗봇, 일반 서비스  |
| Mistral        | 가볍고 빠름, 효율 좋음        | 저사양 환경, 빠른 응답  |
| Mixtral        | 고성능, 리소스 많이 사용       | 고급 추론, 성능 우선   |
| DeepSeek       | 코드·추론 강점             | 개발 보조, 분석 작업   |
| Qwen           | 다국어 강점, 한국어 준수       | 글로벌 서비스, 한글 처리 |
| Code Llama     | 코드 특화                | 코드 생성/리팩토링     |
| DeepSeek Coder | 최신 코드 성능 우수          | 개발 생산성 향상      |
| Phi            | 초경량, 매우 빠름           | 테스트, 저사양 PC    |


## 3. Docker
### 1. 올리마 컨테이너 
  ```
  # Ollama 컨테이너 실행 (데이터 보존을 위한 볼륨 설정 및 포트 매핑)
  docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
  ```

### 2. 올리마 실행
 ```
  # qwen3:4b 모델 다운로드 및 대화형 인터페이스 실행
  # 실행할때 마다 사용함.
  docker exec -it ollama ollama run qwen2.5:3b

  # 다운로드 진행 상황 표시:
  # pulling manifest
  # pulling 8e7ca4ddb09c... 100% ▕████████████████▏ 1.9 GB
  # pulling f8ac8c4bdb71... 100% ▕████████████████▏  485 B
  # pulling cf59c5c5eacf... 100% ▕████████████████▏  11 KB
  # pulling 1f16d5c77cbe... 100% ▕████████████████▏   68 B
  # pulling e3038111a6ee... 100% ▕████████████████▏  561 B
  # verifying sha256 digest
  # writing manifest
  # success
 ```

### 3. 종료
 ```
 /bye
 ```

## 4. Ollama - Spring AI
### 1. 의존성
 ```
  # 기존에 설정한 AI가 있다면 주석필요
  implementation 'org.springframework.ai:spring-ai-ollama-spring-boot-starter'
 ```

### 2. properties
- 소스
 ```
  spring:
  application:
    name: lesson
  ai:
    ollama: # 추가
      # Ollama 서버 주소 (도커 포트 포워딩 11434 확인)
      base-url: http://localhost:11434
      chat:
        options:
          # 우리가 설치하고 테스트한 모델명
          model: qwen2.5:3b
          # 창의성 조절 (0.7은 일반적인 대화에 적합)
          temperature: 0.7
          
          # 3. 샘플링 제어 (Top-K, Top-P) - 방금 학습한 내용!
          # 확률 상위 40개 후보군으로 제한
          top-k: 40
          # 누적 확률 90% 이내의 단어만 선택
          top-p: 0.9
          
          # 4. 답변 길이 및 품질 제어
          # 답변의 최대 토큰(단어 조각) 수를 제한 (너무 짧으면 답변이 끊김)
          num-predict: 10000
          # 동일한 문구 반복을 방지하는 정도 (1.1 이상 권장)
          repeat-penalty: 1.1
 ```
- top_k/top_p
  - top_k
    - 모델이 다음 단어를 예측할 때, “확률이 높은 상위 K개 후보만 남기고 그 안에서 선택”하는 방식
    - 값이 작을수록 보수적이고 안정적인 결과
    - 값이 클수록 다양한 단어 선택 가능 (조금 더 창의적)
  - top_p
    - 확률이 높은 단어부터 누적해서 “합계가 p(%)가 될 때까지” 후보를 포함시키는 방식
    - 값이 낮을수록 더 안전하고 일관된 답변
    - 값이 높을수록 더 다양한 표현 가능
  - 선택 기준
  
  | 목적 | Top-K | Top-P | 기대 효과 | 추천 분야 |
  | --- | --- | --- | --- | --- |
  | 정확성 우선 | 10~20 | 0.7~0.8 | 일관되고 예측 가능한 답변 생성 | 기술 문서 요약, 코드 생성 |
  | 균형 잡힌 답변 | 40~50 | 0.9 | 정확성과 창의성의 적절한 조화 | 일반적인 챗봇 대화 |
  | 창의성 우선 | 80~100 | 0.95 | 다양하고 의외성 있는 답변 생성 | 소설 쓰기, 아이디어 브레인스토밍 |

### 3. 주입
- 주입
 ```
 private final ChatClient chatClient;
 ```

- 서비스단에서 기존에 사용하던 chatClient을 이용해서 그대로 사용가능함. 
