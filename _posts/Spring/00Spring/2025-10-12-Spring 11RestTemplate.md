---
title: 11 RestTemplate
date: 2025-10-12 10:00:00 +09:00
categories: [00Spring Framework, Spring]
tags: [ Java, Spring Framework, SpringBoot, RestTemplate, WebClient ]
---

## 1. RestTemplate 
### 1. RestTmeplate
 - Spring Framework에서 제공하는 HTTP 클라이언트
 - 스프링 애플리케이션에서 다른 REST API 서버로 HTTP 요청을 보내고 응답을 받기 위해 사용하는 도구
 - 특징

| 특징              | 설명                                                                           |
| --------------- | ---------------------------------------------------------------------------- |
| 동기 방식           | 요청을 보내면 서버 응답을 받을 때까지 기다림                                                    |
| 다양한 HTTP 메서드 지원 | GET, POST, PUT, DELETE 등                                                     |
| 메시지 변환 지원       | JSON ↔ Java 객체 변환 자동 처리 (Jackson 라이브러리 필요)                                   |
| 예외 처리           | HTTP 상태 코드 기반 예외 발생 (`HttpClientErrorException`, `HttpServerErrorException`) |
| 간단한 REST 호출     | 한 줄로 요청과 응답 처리 가능                                                            |


### 2. RestTemplate & RestController
#### 1. RestController
   - @RestController는 스프링 MVC 컨트롤러 + @ResponseBody 결합형
   - 컨트롤러 메서드에서 반환하는 객체를 자동으로 JSON으로 변환해서 HTTP 응답으로 보내주는 역할을 함
   - Java
     ```
      @RestController
      public class MyController {
      
          @GetMapping("/hello")
          public String hello() {
              return "Hello";  // 클라이언트에 "Hello"라는 HTTP 응답으로 전송
          }
      }
     ```

#### 2. RestTempalte
   - 스프링 애플리케이션이 외부 API에 요청을 보낼 때 사용하는 것
   - HTTP 클라이언트 역할 / 외부 REST API 호출해서 데이터를 받아오는 용도
   - Java
     ```
     RestTemplate restTemplate = new RestTemplate();
     String response = restTemplate.getForObject("https://api.example.com/data", String.class);
     ```

#### 3. 비교

   | 항목         | RestController     | RestTemplate          |
   | ---------- | ------------------ | --------------------- |
   | 목적         | 클라이언트 요청 처리 → 응답   | 다른 서버/API 호출 → 응답 수신  |
   | 역할         | 서버에서 JSON 응답 자동 생성 | 클라이언트에서 JSON 응답 자동 파싱 |
   | HTTP 요청 주체 | 클라이언트 → 내 서버       | 내 서버 → 외부 서버          |
   | 처리 자동화     | 객체 → HTTP 응답 자동 변환 | HTTP 응답 → 객체 자동 변환    |


### 3. RestTemplate / WebClient / RestClient

  | 이름               | 역할                 | 동기/비동기       | 사용 목적                       | 등장 시점       |
  | ---------------- | ------------------ | ------------ | --------------------------- | ----------- |
  | RestTemplate | 서버 간 HTTP 요청 클라이언트 | 동기           | 간단한 외부 API 호출, 레거시          | Spring 3~5  |
  | WebClient    | 서버 간 HTTP 요청 클라이언트 | 비동기/Reactive | 대규모 비동기 호출, 리액티브            | Spring 5+   |
  | RestClient   | 서버 간 HTTP 요청 클라이언트 | 동기           | RestTemplate 대체, 단순 REST 호출 | Spring 6.1+ |

   - Spring 진영에서 공식적으로 RestTemplate는 더 이상 주요 업데이트 대상이 아님.
   - Spring 5 (2017년)부터 WebClient가 등장 -> 사용금지가 아닌 WebClient 사용권장 상태
     - RestTemplate는 요청을 보낼 때마다 스레드가 요청이 끝날 때까지 대기(Blocking)상태가 됨.
     - 따라서 요청이 많아질수록 느려지는 현상이 발생함.
   - WebClient는 비동기 방식을 지원하기 위해 만들어진 방식으로, 동기방식을 위한 RestClient 등장
     - 동기 : RestTemplate -> RestClient
     - 비동기 : WebClient (동기로 사용가능하지만 비권장)

## 2. RestTemplate 사용
  ```
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
  ```
  ```
    @Autowired
    private RestTemplate restTemplate;

    // 1. GET 요청 (간단히 문자열로 받기)
    public String getStringData() {
        String url = "https://jsonplaceholder.typicode.com/posts/1";
        return restTemplate.getForObject(url, String.class); 
    }

    // 2. GET 요청 (객체로 매핑)
    public Post getPostData() {
        String url = "https://jsonplaceholder.typicode.com/posts/1";
        return restTemplate.getForObject(url, Post.class); //URL로 요청 후 응답을 Post.Class(DTO) 객체로 받음
    }

    // 3. POST 요청 (데이터 전송)
    public Post createPost() {
        String url = "https://jsonplaceholder.typicode.com/posts";
        Post newPost = new Post(1, "제목", "내용");
        return restTemplate.postForObject(url, newPost, Post.class); //newPost는 요청바디값
    }

    // 4. PUT 요청
    public void updatePost() {
        String url = "https://jsonplaceholder.typicode.com/posts/1";
        Post update = new Post(1, "수정된 제목", "수정된 내용");
        restTemplate.put(url, update); //put은 return값없음
    }

    // 5. DELETE 요청
    public void deletePost() {
        String url = "https://jsonplaceholder.typicode.com/posts/1";
        restTemplate.delete(url); //delete 는 return값없음
    }

    // 6. 헤더 추가 + 응답 전체 받기
    public ResponseEntity<String> getWithHeader() {
        String url = "https://jsonplaceholder.typicode.com/posts";
        HttpHeaders headers = new HttpHeaders();
        headers.add("Authorization", "Bearer my-token"); //헤더값 추가
        HttpEntity<String> entity = new HttpEntity<>(headers);

        // @URL : 지정한 URL에 HTTP 요청을 보냄
        // @HttpMethod.Get 요청 메서드는 GET
        // @entity: 요청 시 사용할 헤더 정보 포함
        // @String.class: 응답 본문을 String
        return restTemplate.exchange(url, HttpMethod.GET, entity, String.class);
        // 리턴은 ResponseEntity로 받음.
    }
  ```
## 3. WebClient 사용
### 1. 기본형
  ```
    ResponseEntity<String> getWithHeaderWebClient() {
      WebClient client = WebClient.create("https://jsonplaceholder.typicode.com");
  
      return client.get()                                       // ① GET 요청 지정
                   .uri("/posts")                               // ② URI 설정
                   .header("Authorization", "Bearer my-token")  // ③ 헤더 추가
                   .retrieve()                                  // ④ 요청 실행 및 응답 수신 단계 진입
                   .toEntity(String.class)                      // ⑤ ResponseEntity 형태로 변환
                   .block();                                    // ⑥ 비동기 → 동기 변환 (실제 결과 대기)
    }  
  ```
### 2. Java
  ```
    @Bean
    public WebClient webClient() {
        return WebClient.builder()
                .baseUrl("https://jsonplaceholder.typicode.com") // 기본 URL
                .build();
    }
  ```
  ```
    // 1. GET 요청 (문자열로 받기)
    public String getStringData() {
        return webClient.get()
                .uri("/posts/1")
                .retrieve()
                .bodyToMono(String.class)
                .block(); // block()으로 동기 변환
    }

    // 2. GET 요청 (객체로 매핑)
    public Post getPostData() {
        return webClient.get()
                .uri("/posts/1")
                .retrieve()
                .bodyToMono(Post.class)
                .block();
    }

    // 3. POST 요청 (객체 전송)
    public Post createPost() {
        Post newPost = new Post(1, "새 제목", "새 내용");

        return webClient.post()
                .uri("/posts")
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(newPost)
                .retrieve()
                .bodyToMono(Post.class)
                .block();
    }

    // 4. PUT 요청
    public void updatePost() {
        Post update = new Post(1, "수정된 제목", "수정된 내용");

        webClient.put()
                .uri("/posts/1")
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(update)
                .retrieve()
                .toBodilessEntity()
                .block();
    }

    // 5. DELETE 요청
    public void deletePost() {
        webClient.delete()
                .uri("/posts/1")
                .retrieve()
                .toBodilessEntity()
                .block();
    }

    // 6. 헤더 추가 + 응답 전체 받기
    public String getWithHeader() {
        return webClient.get()
                .uri("/posts")
                .header(HttpHeaders.AUTHORIZATION, "Bearer my-token")
                .retrieve()
                .bodyToMono(String.class)
                .block();
    }
  ```

