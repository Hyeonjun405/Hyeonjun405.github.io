---
title: 06 MultipartFile
date: 2025-12-05 10:00:00 +09:00
categories: [00Spring Framework, SpringMVC]
tags: [ Java, Spring Framework, SpringMVC ]
---

## 1. MultipartFile
### 1. MultipartFile
 - MultipartFile은 스프링 프레임워크가 제공하는 인터페이스
 - HTTP multipart/form-data 요청으로 업로드된 파일을 스프링이 편하게 다루도록 감싼 추상화 객체
 - package : `org.springframework.web.multipart.MultipartFile`

### 2. 주요메서드
  ```
  String getName();                // 폼 필드 이름
  String getOriginalFilename();    // 클라이언트가 보낸 원래 파일명
  String getContentType();         // Content-Type 헤더 (ex: image/png)
  boolean isEmpty();               // 파일이 비어있는지
  long getSize();                  // 바이트 단위 크기
  byte[] getBytes() throws IOException;
  InputStream getInputStream() throws IOException;
  void transferTo(File dest) throws IOException, IllegalStateException;
  ```

### 3. MultipartFile이 생성되는 흐름 
 - 클라이언트가 multipart/form-data로 요청 전송
 - DispatcherServlet → MultipartResolver가 요청 파싱
 - MultipartResolver가 MultipartFile 구현체(예: StandardMultipartFile, CommonsMultipartFile)를 생성
 - 컨트롤러 파라미터로 주입

## 2. 예시
### 1. 단일 파일 
  ```
  @PostMapping("/upload")
  public ResponseEntity<String> upload(@RequestParam("file") MultipartFile file) throws IOException {
      if (file.isEmpty()) {
          return ResponseEntity.badRequest().body("empty");
      }
  
      String original = file.getOriginalFilename();
      long size = file.getSize();
      String contentType = file.getContentType();
  
      Path target = Paths.get("/data/uploads", UUID.randomUUID() + "-" + sanitize(original));
      file.transferTo(target.toFile());
  
      return ResponseEntity.ok("saved");
  }
  ```

### 2. 다중파일
  ```
  @PostMapping("/multi")
  public ResponseEntity<?> multi(@RequestParam("files") MultipartFile[] files) {
      for (MultipartFile file : files) {
          // 각 파일 검증 및 저장
      }
      return ResponseEntity.ok("ok");
  }
  ```


### 3. 검증
 ```
  // 확장자 검사
  String originalName = file.getOriginalFilename(); //getName은 폼필드이름이 나옴
  
  String ext = originalName.substring(originalName.lastIndexOf('.') + 1).toLowerCase();
  
  List<String> allowedExt = List.of("jpg", "jpeg", "png", "pdf");

  if (!allowedExt.contains(ext)) {
      throw new IllegalArgumentException("허용되지 않은 확장자");
  }
  
  // Mime 타입검증
  String mimeType = file.getContentType();

  List<String> allowedMime = List.of("image/jpeg", "image/png", "application/pdf" );

  if (!allowedMime.contains(mimeType)) {
      throw new IllegalArgumentException("허용되지 않은 MIME 타입");
  }
  
  // 파일 사이즈 검증
  long max = 5L * 1024 * 1024; // 5MB
  
  if (file.getSize() > max) {
     throw new MyFileTooLargeException();
  }
 ```
  - MIME 타입
    - 브라우저나 서버가 “이 파일이 어떤 종류인지” 구분하기 위해 붙여주는 공식적인 콘텐츠 형식 이름 
    - 요청헤더에 `Content-Type: image/png` 요렇게 들어옴.
    - virus.jpg.exe → 확장자를 jpg로 위장 가능하지만 MIME 타입은 내부 구조 기반으로 매핑돼서 위장하기 훨씬 어려움.
    - 종류
      ```
       image/jpeg      → jpg, jpeg 이미지
       image/png       → png 이미지
       application/pdf → pdf 문서
       text/plain      → txt 파일
       application/zip → zip 압축파일
      ```
    

## 3.note
### 1. 컨트롤러는 단일인데, 파일은 다중이면
- 파일 내역
 ```
  Content-Disposition: form-data; name="file"; filename="a.png"
  Content-Disposition: form-data; name="file"; filename="b.png"
  Content-Disposition: form-data; name="file"; filename="c.png"
 ```

- 컨트롤러
 ```
 @PostMapping("/upload")
  public String upload(@RequestParam("file") MultipartFile file) {
      System.out.println(file.getOriginalFilename());
      return "ok";
  } 
 ```

- 업로드 파일
  - 업로드 대상
    - 파라미터가 MultipartFile이면 → 가장 첫 번째 part 한 개만 넣음
    - 파라미터가 MultipartFile[] 또는 List<MultipartFile>이면 → 모두 넣음
  - 따라서 가장처음 a.png만 들어옴
