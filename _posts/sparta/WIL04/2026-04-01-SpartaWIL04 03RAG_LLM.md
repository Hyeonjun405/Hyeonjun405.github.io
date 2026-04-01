---
title: 03 RAG & LLM 
date: 2026-04-01 10:00:00 +09:00
categories: [Sparta, SpartaWIL04]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - 비교군을 잘 판단해서 파악 필요!
 - RAG & CAG (Context 데이터를 어떻게 전달 하는가?) 
   - RAG 데이터의 선별을 통해서 프롬프트에 전달
   - CAG 필요한 데이터를 무조건 Context에 전달해버림
 - RAG를 사용하는건
   - 어떤 질문을 하던 프롬프트는 어차피 공통적으로 들어감.
   - RAG를 쓴다는건 내부 데이터를 추가로 토큰을 사용해서 넘겨줘서 정확한 답변을 유도하는것
   - 쓰는 것과 안쓰는 것 비교하면 토큰을 사용해서 정확한 데이터를 찾아내는 것에 가까움
   - 안쓰는거랑 비교하면 X

## 2. 기본 흐름
### 1. RAG/CAG

| 구분        | RAG (Retrieval-Augmented Generation)        | CAG (Cache-Augmented Generation)       |
|-----------|---------------------------------------------|----------------------------------------|
| 핵심 개념(☆)  | 필요할 때마다 검색해서 가져옴<br> 벡터db등으로 데이터를 선별적으로 가져옴 | 미리 만들어둔 결과를 재사용<br> 고정적으로 데이터를 하드로 박음. |
| 데이터 획득 시점 | 매 요청마다 검색                                   | 사전에 생성/저장                              |
| 주요 기술     | Vector DB, Embedding, Similarity Search     | Cache, Key-Value 저장                    |
| 응답 속도     | 상대적으로 느림 (검색 + 생성)                          | 매우 빠름 (캐시 히트 시)                        |
| 비용        | 호출마다 비용 발생                                  | 캐시 활용 시 비용 절감                          |
| 정확도       | 최신 데이터 반영 가능                                | 캐시에 의존 (오래되면 틀릴 수 있음)                  |
| 사용 케이스    | 문서 검색, QA 시스템                               | FAQ, 반복 질문, 동일 질의                      |


### 2. RAG 흐름
#### 1. 문서저장

| 단계 | 이름           | 작업 내용 | 상세 설명                                         |
|----|--------------| --- |-----------------------------------------------|
| 1  | 문서 수집        | Data Loading | PDF, Word, 사내 Wiki 등 다양한 형태의 비정형 데이터를 불러옴     |
| 2  | 문서 분할        | Chunking  | 문서가 너무 길면 LLM이 읽기 힘들기 때문에, 의미 있는 단위(문단 등)로 쪼갬 |
| 3  | 임베딩 생성       | Embedding | 텍스트를 AI가 이해할 수 있는 다차원 수치(Vector)로 변환      |
| 4  | Vector DB 저장 | Storing | 변환된 벡터 데이터와 원문 조각을 검색 엔진(Vector DB)에 저장   |

#### 2. 질문 응답 (런타임)

| 단계 | 이름      | 작업 내용             | 상세 설명                                                                 |
| -- | ------- | ----------------- |-----------------------------------------------------------------------|
| 1  | 질문 임베딩  | Query Embedding   | 사용자의 질문을 임베딩 모델을 통해 벡터로 변환<br> 이 과정에서 문장의 의미가 숫자 공간으로 표현              |
| 2  | 유사도 검색  | Similarity Search | 질문 벡터와 Vector DB에 저장된 문서 벡터들을 비교하여 가장 의미적으로 유사한 문서 조각(Chunk)을 찾음      |
| 3  | 컨텍스트 증강 | Augmentation      | 검색된 문서 조각과 사용자의 질문을 결합하여 LLM에 전달할 프롬프트를 구성                            |
| 4  | 최종 생성   | Generation        | LLM이 전달받은 컨텍스트를 기반으로 답변을 생성 <br> 이때 모델은 자체 지식이 아닌, 주어진 문서를 근거로 응답 |


## 3. RAG와 Fine-tuning
### 1. 개념
 - RAG (Retrieval-Augmented Generation)
   - 검색 증강 생성 
   - 외부 데이터(문서, DB, 벡터DB 등)에서 필요한 정보를 검색해서 프롬프트에 넣고 답변
   - 모델 자체는 그대로, 지식만 외부에서 가져옴(학습X, 지식만 추가)
   - 핵심: “모르는 건 찾아서 답한다”
 - Fine-tuning
   - 모델 자체를 학습시켜서 행동/지식 자체를 바꿈
   - 특정 도메인, 스타일, 규칙을 모델에 내재화(학습해서 지식으로 만듬)
   - 핵심: “이미 알고 있는 상태로 만든다”
 - 비교
  
  | 구분                | RAG               | Fine-tuning    |
  | ----------------- | ----------------- | -------------- |
  | 지식 반영             | 외부에서 검색           | 모델 내부에 학습      |
  | 최신 데이터            | 매우 강함 (실시간 반영 가능) | 약함 (재학습 필요)    |
  | 비용                | 상대적으로 저렴          | 비쌈 (학습 비용 큼)   |
  | 속도                | 검색 + 생성이라 약간 느림   | 빠름             |
  | 정확도               | 검색 품질에 의존         | 학습 데이터 품질에 의존  |
  | 유지보수              | 쉬움 (데이터만 바꾸면 됨)   | 어려움 (재학습 필요)   |
  | 환각(Hallucination) | 낮음 (근거 기반)        | 높을 수 있음        |
  | 사용 사례             | 문서 QA, 사내 지식 검색   | 스타일, 규칙, 행동 학습 |

### 2. 사용 환경
- RAG
  - 데이터가 자주 바뀜 (공지사항, 정책, 매뉴얼)
  - 문서 기반 질의응답 (PDF, 위키, DB)
  - 근거가 중요한 경우 (출처 필요)
  - 예시
    - 사내 문서 검색 시스템
    - 고객센터 챗봇
    - 법률/의료 Q&A (근거 중요)
    
- Fine-tuning을 써야 하는가
  - 특정 말투/스타일 유지 (예: 상담원 톤)
  - 반복되는 패턴 학습
  - 규칙 기반 응답 (형식 고정)
  - 예시
    - 고객 응대 스타일 통일
    - 코드 생성 스타일 맞추기
    - 특정 포맷 JSON 응답 강제


## 4. RAG 기반 QA 시스템 - Controller
```
// 큰 특징 X
@PostMapping("/ask")
public ApiResponse<AnswerResponse> ask(@RequestBody QuestionRequest request) {
  return ApiResponse.ok( ragService.ask(request.getQuestion()));
}
```

## 5. RAG 기반 QA 시스템 - Service
### 1. Private
#### 1. 공통 프롬프트
```
private static final String RAG_PROMPT_TEMPLATE = """
  다음 문서들을 참고하여 질문에 답변해주세요.
  문서에 없는 내용은 답변하지 마세요.
  답변은 한국어로 작성해주세요.

  [참고 문서]
  %s

  [질문]
  %s

  [답변]
  """;
  
// String.format(RAG_PROMPT_TEMPLATE, combineDocuments(relevantDocs), question)) 패턴으로 사용가능
```

#### 2. 자동응답생성 
```
private String generateAnswer(String question, List<Document> docs) {
 return chatClient.prompt()
    .user(String.format(RAG_PROMPT_TEMPLATE, combineDocuments(docs), question))
    .call()
    .content();
} 
```

#### 3. 청크파일을 하나의 문자열로 합침
```
private String combineDocuments(List<Document> documents) {
return documents.stream()
    .map(doc -> String.format("[%s]: %s",
        doc.getMetadata().getOrDefault("filename", "Unknown"),
        doc.getText()))
    .collect(Collectors.joining("\n\n---\n\n"));
}
 ```
#### 4. 특정 도큐먼트 조회
```
public List<Document> searchDocumentsWithFilter(String query, String documentId, int topK) {
  return vectorStore.similaritySearch(
      SearchRequest.builder()
          .filterExpression("document_id == '" + documentId + "'")
          .query(query)
          .topK(topK)
          .build()
  );
}
```

### 2. public
#### 1. 기본 질의 형태
```
public AnswerResponse ask(String question) {
// 파라미터 - query / topK / threshold
List<Document> relevantDocs = searchDocuments(question, 5, 0.0); //private 

if (relevantDocs.isEmpty()) {
  throw new DomainException(DomainExceptionCode.NOT_FOUND_CONVERSATION);
}

return AnswerResponse.builder()
    .answer(generateAnswer(question, relevantDocs))
    .build();
} 
```

#### 2. 특정 도큐먼트 내 검색
```
public AnswerResponse askInDocument(String question, String documentId) {
  // 여기서 특정 도큐먼트를 선택함.
  List<Document> relevantDocs = searchDocumentsWithFilter(question, documentId, 3); //private 
  if (relevantDocs.isEmpty()) {
    throw new DomainException(DomainExceptionCode.NOT_FOUND_CONVERSATION);
  }

  // 시스템 추가로 인해 prviate 메소드 사용 X 
  String answer = chatClient.prompt()
      .system("당신은 전문 문서 기반 응답 시스템입니다. 제공된 문서 내용만 사용하세요.")
      .user(String.format(RAG_PROMPT_TEMPLATE, combineDocuments(relevantDocs), question)) //private 
      .call()
      .content();

  return AnswerResponse.builder()
      .answer(answer)
      .build();
} 
```

#### 3. 사용된 소스 Response
```
public RagResponse askWithSource(String question) {
  List<Document> docs = searchDocuments(question, 5, 0.7); //private 
  String answer = generateAnswer(question, docs); //private

  //docs에서 확인한 내용 response
  List<RagResponse.DocumentSource> sources = docs.stream()
      .map(doc -> RagResponse.DocumentSource.builder()
          .filename((String) doc.getMetadata().get("filename"))
          .documentId(doc.getId())
          .preview(doc.getText().substring(0, Math.min(doc.getText().length(), 100)))
          .build()
          )
      .toList();

  return RagResponse.builder()
      .answer(answer)
      .sources(sources)
      .build();
}
```

#### 4. Summary
```
public SearchSummaryResponse getSearchSummary(String query) {
  List<Document> docs = searchDocuments(query, 3, 0.7);
  String summary = chatClient.prompt()
      .user("다음 검색 결과들을 한 문장으로 요약해줘: " + combineDocuments(docs))
      .call()
      .content();

  return SearchSummaryResponse.builder()
      .query(query)
      .summary(summary)
      .build();
}
```
