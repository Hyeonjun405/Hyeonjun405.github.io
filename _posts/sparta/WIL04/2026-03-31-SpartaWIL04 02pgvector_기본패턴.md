---
title: 02 pgvector 기본 패턴
date: 2026-03-31 10:00:00 +09:00
categories: [Sparta, SpartaWIL04]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - 필요한 정보를 벡터DB에 넣은 뒤에 사용하는 방식
 - 기존에는 프롬프트를 사용해서 LLM이 가지고 잇는 데이터를 가지고 답변을 생성했음.
 - RAG 방식은 RAG데이터를 바탕으로 검사하고 그것을 중심으로 답변하게됨.
 - 만약 특정한 서류기준으로 조회를 하고자 한다면
   - 서류를 조회할때 특정 UUID값 기준으로 조회하고
   - 그게 아니면 전체 서류에서 조회하면 됨
 - ☆ VectorDB ☆
   - SpartaNote02 - Vector DB < 기본 개념과 흐름 >
   - SpartaWIL04 - 02pgvector_기본 패턴 < 사용법 / 기본 패턴 파악 >
   - SpartaWIL04 - 03pgvector_실무적인 접근 < 실무 패턴 >

## 2. SpringAI + pgVector
### 1. RAG 방식(Retrieval-Augmented Generation)
 - LLM이 모르는 내용(우리 회사 문서, 최신 자료 등)을 검색해서 끌어와 답변하게 만드는 방식
 - 사용할 데이터를 정해주고, LLM이 그것을 바탕으로 응답을 생성하도록 유도하는 것.

### 2. pgVector이점
 - PostgreSQL 안에서 바로 벡터 검색 가능 (별도 DB 필요 없음)
 - SQL로 유사도 검색 가능 → 개발 난이도 낮음
 - 일반 데이터 + 임베딩을 한 번에 관리 (트랜잭션 보장)
 - 필터링 + 유사도 검색을 동시에 처리 가능
 - 비용 효율 좋음 (추가 인프라 없음)

## 3. RAG용 테이블 생성
### 1. 커스터마이징
 - 가능한 영역
    - 테이블명 변경
    - 컬럼 추가 (예: category, created_at 등)
    - metadata 구조 확장
 - 비권장 영역
    - embedding 컬럼 제거/변경
    - content 구조 완전 변경
      - 벡터 타입 변경 (pgvector 기준 깨짐)

### 2. vector_documents 생성
  ```
    -- 업로드된 파일의 원본 정보를 통째로 보관하는 테이블
    -- 원본 데이터를 VectorDB에 분할해서 보관함.
    -- 그러다보니 VectorDB에서는 쪼개진 상태라 정확히 파악이 어려워서 관리용으로 Vector_documents
    -- 삭제 할 경우에 Vector_dcouments에서 전체 쪼개진 파일 확인 가능 해짐
    CREATE TABLE vector_documents
    (
        id           UUID PRIMARY KEY      DEFAULT gen_random_uuid(), -- 문서 고유 식별자 (자동 생성)
        file_name    VARCHAR(255) NOT NULL,                           -- 업로드된 파일명 (예: manual.pdf)
        content      TEXT         NOT NULL,                           -- 파일 전체 텍스트 내용
        content_type VARCHAR(50)  NOT NULL,                           -- 파일 형식 (PDF, TXT 등)
        metadata     TEXT,                                            -- 부가 정보 (저자, 페이지 수 등 자유 형식)
        chunk_count  INTEGER      NOT NULL DEFAULT 0,                 -- 이 문서가 몇 개의 청크로 분할됐는지 카운트
        created_at   TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP, -- 문서 최초 등록 시각
        updated_at   TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP  -- 문서 최종 수정 시각
    );
  ```

### 3. vector_store 생성
  ```
  -- 검색대상 벡터 테이블
  CREATE TABLE vector_store
  (
      id         UUID PRIMARY KEY   DEFAULT gen_random_uuid(),  -- 테이블 내 고유 식별자
      content    TEXT      NOT NULL,                            -- 잘린 텍스트
      metadata   JSONB,                                         -- 청크정보(JSON) / 서류명,페이지,청크정보
      embedding  vector(3072),                                  -- 벡터
      created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
  );
  ```

### 4. 기타 설정
  ```
  -- [Step 3] 검색 성능 최적화 인덱스
  CREATE INDEX idx_documents_filename ON vector_documents (file_name);
  
  -- 메타데이터 필터링 가속화
  CREATE INDEX idx_vector_store_metadata ON vector_store USING gin (metadata);
  ```

## 5. Spring 의존성 및 셋팅
### 1. 의존성
  ```
  dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:2023.0.2"
        mavenBom "org.springframework.ai:spring-ai-bom:1.0.0"
      }
  }
  
  dependencies {
      implementation 'org.springframework.ai:spring-ai-starter-model-openai'
      implementation 'org.springframework.ai:spring-ai-starter-vector-store-pgvector'
  }
  ```

### 2. 프로퍼티스
 ```
   autoconfigure:
    exclude:
      # Spring Boot 자동설정을 꺼서, pgvector 설정을 내가 직접 하겠다는 의미
      - org.springframework.ai.vectorstore.pgvector.autoconfigure.PgVectorStoreAutoConfiguration
 
   ai:
    openai:
      # API 키: Google AI Studio에서 발급받은 키를 입력
      # ${GOOGLE_AI_GEMINI_API_KEY}
      api-key: GOOGLE_AI_GEMINI_API_KEY
      # 임베딩 모델 설정(데이터의 벡터화)
      embedding: 
        base-url: https://generativelanguage.googleapis.com/v1beta/openai/
        options:
          model: gemini-embedding-001
      chat:
        # Base URL: Google Gemini가 제공하는 OpenAI 호환 API 주소
        base-url: "https://generativelanguage.googleapis.com/v1beta/openai/"
        options:
          # 모델명: 현재 가장 최신/경량 모델인 gemini-2.0-flash-lite 등을 지정합니다.
          model: "gemini-2.5-flash-lite"
          # 온전성(Temperature): 0.0은 가장 보수적이고 사실적인 답변을 생성합니다.
          # (분석, 요약, 데이터 추출에 최적화된 설정)
          temperature: 0.7

        # 엔드포인트 경로: 대화형 API를 호출하기 위한 표준 경로입니다.
        completions-path: "/chat/completions"

    # pgvector 설정 
    vectorstore:
      pgvector:
        initialize-schema: false          # 직접 SQL로 테이블을 만들었으므로 false 권장
        dimensions: 3072                  # 임베딩 모델 최적화 차원
        distance-type: COSINE_DISTANCE    # 유사도 계산 방식 (cosine, l2, inner_product)
        index-type: hnsw                  # 고속 검색을 위한 인덱스 방식
 ```

### 3. VectorStoreConfig
 ```
  @Setter
  @Configuration
  @ConfigurationProperties(prefix = "spring.ai.vectorstore.pgvector")
  public class VectorStoreConfig {
  
    private int dimensions;
 
    @Bean
    public VectorStore vectorStore(DataSource dataSource, EmbeddingModel embeddingModel) {
  
      JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
  
      // 직접만든 벡터스토어를 쓰기 위해서 JdbcTemplate를 파라미터로 제공함.
      return PgVectorStore.builder(jdbcTemplate, embeddingModel)
          // 임베딩 벡터의 차원 수를 설정
          // 프로퍼티스에서 가져옴
          .dimensions(dimensions)
  
          // 벡터 간의 유사도를 계산하는 방식을 설정
          .distanceType(PgVectorStore.PgDistanceType.COSINE_DISTANCE)
  
          // 검색 속도를 높이기 위한 인덱스 알고리즘을 설정
          // HNSW는 대규모 데이터셋에서 빠르고 정확한 근사 최근접 이웃 검색을 지원
          .indexType(PgVectorStore.PgIndexType.HNSW)
  
          // 애플리케이션 시작 시 자동으로 테이블 스키마를 생성할지 여부를 결정
          // 직접 SQL로 테이블을 관리하므로 false로 설정하여 기존 구조를 유지
          .initializeSchema(false)
  
          // 기존에 존재하는 벡터 테이블을 삭제하고 새로 만들지 설정
          // 데이터 유실 방지를 위해 false로 설정하는 것이 안전
          .removeExistingVectorStoreTable(false)
  
          // 시작 시 DB 테이블의 컬럼 구성이나 차원이 설정값과 일치하는지 검증
          // 차원 설정과 실제 DB의 vector 타입 일치 여부를 체크
          .vectorTableValidationsEnabled(true)
  
          // 데이터가 저장될 PostgreSQL의 스키마 이름을 지정
          // 별도의 커스텀 스키마를 쓰지 않는다면 기본값인 "public"을 사용
          .schemaName("public")
  
          // 벡터 데이터가 실제로 저장될 테이블의 이름을 지정
          // 여기서는 직접 생성한 "vector_store" 테이블과 연결
          .vectorTableName("vector_store")
          .build();
    } 
  }
 ```

## 6. Spring - 기본 샘플(데이터 조회)
### 1. 문서 업로드 및 벡터화
```
  @Transactional
  public DocumentUploadResponse uploadDocument(MultipartFile file) throws IOException {
    String filename = file.getOriginalFilename();
    String contentType = file.getContentType();
    String content = new String(file.getBytes(), StandardCharsets.UTF_8);

    /* 원본 엔티티 생성 (ID 선발급)
     *  - 애플리케이션에서 UUID를 미리 생성하여 사용
     *  - 청크 단위 업로드 시 동일한 document_id를 사용하기 위해 필요
     *  - DB 생성 전략을 사용할 경우 save 이후에만 ID 확인 가능하므로 부적합
     */
    VectorDocument vectorDocument = VectorDocument.builder()
        .id(UUID.randomUUID())
        .fileName(filename)
        .content(content)
        .contentType(contentType)
        .build();

    // DB에 저장하여 확정된 ID 획득
    VectorDocument savedDocument = vectorDocumentRepository.save(vectorDocument);

    /* private 함수
     * - 확정된 ID를 전달하여 문서 분할 및 메타데이터 설정
     * - 문서를 한번에 넣으면 크기 때문에 분할함
     * - 여기서 메타데이터 생성함.
     */
    List<Document> chunks = createChunks(content, savedDocument);

    /* 청크 개수 업데이트 및 Vector Store 저장
     *  - 실제로 쓰이는 값은 X
     *  - 개발자 또는 운영자가 관리 하기 위한 용도
     *  - 청크로 쪼개고 하나씩 저장하면서 성공 개수를 직접 세는 느낌.(해당 로직에는 X)
     */
    vectorDocument.setChunkCount(chunks.size());
    
    // 주입받은 벡터스토어
    vectorStore.add(chunks); 

    return DocumentUploadResponse.builder()
        .documentId(savedDocument.getId().toString())
        .filename(savedDocument.getFileName())
        .chunkCount(savedDocument.getChunkCount())
        .build();
  }
  
  // 2. 문서 분할 로직 (추상화)
  private List<Document> createChunks(String content, VectorDocument entity) {
    /* new TokenTextSplitter(
     *    500,   // defaultChunkSize: 한 청크당 약 500 토큰(단어 조각)씩 자름
     *    100,   // minChunkSizeChars: 최소 100자 이상은 되어야 의미가 있다고 판단
     *    5,     // minChunkLengthToEmbed: 너무 짧은 문장(예: "네.")은 임베딩 제외
     *    10000, // maxNumChunks: 한 파일당 생성될 수 있는 최대 조각 수
     *    true   // keepSeparator: 문단 구분자를 유지하여 가독성 보존
     *  )
     */
    TextSplitter splitter = new TokenTextSplitter(500, 100, 5, 10000, true);

    // 공통 메타데이터 생성
    Map<String, Object> metadata = Map.of(
        "document_id", entity.getId().toString(),
        "filename", entity.getFileName(),
        "source", "user_upload"
    );

    // Spring AI Document 객체 생성 후 분할
    Document rawVectorDocument = new Document(content, metadata);
    List<Document> splitChunks = splitter.split(rawVectorDocument);

    // 각 청크에 랜덤 UUID 부여 (충돌 방지 및 고유 식별자 확보)
    return splitChunks.stream()
        .map(chunk -> new Document(UUID.randomUUID().toString(), chunk.getText(),
            chunk.getMetadata()))
        .toList();
  }
```

### 2. 특정 문서 유사도 검색
```
public SimilaritySearchResponse similaritySearchByDocument(UUID documentId, String query,
      Integer topK) {
    // Vector Store에서 특정 문서 범위(documentID) 내에서, 입력 쿼리와 가장 가까운 chunk K개 가져오기
    List<Document> searchResults = vectorStore.similaritySearch(
        SearchRequest.builder()
          // 서류를 구분하는 작업을 제거하면 전체 범위에서 수행함.
            .filterExpression(new Filter.Expression(
                Filter.ExpressionType.EQ,
                new Filter.Key("document_id"),
                new Filter.Value(documentId.toString())
            ))
            .query(query)
            .topK(topK)
            .build()
    );

    /* 정리/포맷팅 
     * 벡터 검색 결과를 chunk(문자열) + metadata를 -> 객체(DTO) 형태로 전환
     */
    List<SimilaritySearchResponse.SearchResult> results = searchResults.stream()
        .map(doc -> SimilaritySearchResponse.SearchResult.builder()
            .id(doc.getId())
            .content(doc.getText())
            .metadata(doc.getMetadata())
            .build())
        .toList();

    // 정리된 DTO 리스트를 최종 응답 객체로 감싸서 반환
    return SimilaritySearchResponse.builder()
        .query(query)
        .resultCount(results.size())
        .results(results)
        .build();
  }
```

### 3. 특정 문서 제거
```
@Transactional
public void deleteDocument(UUID documentId) {
  VectorDocument entity = vectorDocumentRepository.findById(documentId)
      .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 문서입니다."));

  // DB 삭제 (Cascade 설정이 없다면 수동 삭제 혹은 외래키 정책 활용)
  vectorDocumentRepository.delete(entity);

  //  Vector Store 삭제
  // Spring AI의 Filter 기능을 활용해 해당 document_id를 가진 모든 청크 조회
  try {
    List<String> chunkIds = vectorStore.similaritySearch(
        SearchRequest.builder()
            .query("*")
            .filterExpression(new Filter.Expression(
                Filter.ExpressionType.EQ,
                new Filter.Key("document_id"),
                new Filter.Value(documentId.toString())
            ))
            .topK(10000)
            .build()
    ).stream().map(Document::getId).toList();

    /* 해당 청크 id들을 삭제함
     * - vectorStore.delete() 메서드는 컬렉션(리스트, 배열 등)을 입력받아 내부에서 한 번에 처리하도록 구현됨
     * - 별도로 for문 작업 불필요!
     */
    if (!chunkIds.isEmpty()) {
      vectorStore.delete(chunkIds);
      log.info("Vector Store 내 관련 청크 {}개 삭제 완료", chunkIds.size());
    }
  } catch (Exception e) {
    log.error("Vector Store 삭제 중 오류 발생 (DB는 삭제됨): {}", e.getMessage());
  }
}
```
