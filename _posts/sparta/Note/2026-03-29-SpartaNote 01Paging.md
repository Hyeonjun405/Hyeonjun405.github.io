---
title: 01 Paging
date: 2026-03-29 10:00:00 +09:00
categories: [Sparta, SpartaNote]
tags: [ Java, Spring Framework, JPA ]
---

## 1. Note
 - Pageable = 요청 / Page<T> = 응답 / QueryDSL = 쿼리 도구
 - Pageable과 Paging은 그냥 warp에 가깝구나
 - 조회할때 레파지토리에서 설정해서 리턴만해 주면 그대로 나옴!

## 2. Paging & Pageable
### 1. Paging 
- 데이터를 N개 나눠보는 것이 페이징
- 예시
  ```
  전체 100개 데이터를 10개씩 나눠서 보여주는 것
   1페이지: 1~10번
   2페이지: 11~20번
   ...
  ```
  
### 2. Pageable
- 몇 페이지, 몇 개씩 줘" 라는 요청 정보
- 흐름
  ```
    // 클라이언트가 이렇게 요청
    GET /members?page=0&size=10&sort=username,asc
    
    // Spring이 자동으로 Pageable로 변환해줌
    Pageable pageable = PageRequest.of(0, 10, Sort.by("username"));
    
    pageable.getPageNumber()  // 0  (몇 번째 페이지)
    pageable.getPageSize()    // 10 (몇 개씩)
    pageable.getOffset()      // 0  (몇 번째부터, SQL offset)
  ```
- 데이터구분
  - 스프링에서는 내부적으로 알아서 처리함. Request에 정보가 더 있어도 무시 
  - Request에서 필요한 정보(쿼리파라미터나 바디 둘다 무관)
     ```
      // 클라이언트가 넘겨주는 것
      pageable.getPageNumber()  // 몇 번째 페이지
      pageable.getPageSize()    // 몇 개씩
      pageable.getSort()        // 정렬
      pageable.getOffset()      // (page * size) 자동 계산
     ```
    - Response에 담겨있는 정보
     ```
      page.getContent()         // 실제 데이터
      page.getTotalElements()   // 전체 데이터 수
      page.getTotalPages()      // 전체 페이지 수
      page.getNumber()          // 현재 페이지
      page.getSize()            // 페이지 크기
      page.isFirst()            // 첫 페이지 여부
      page.isLast()             // 마지막 페이지 여부
      page.hasNext()            // 다음 페이지 여부
     ```
  
### 3. Page<T>
 - 요청한 데이터 + 페이지 메타정보를 담은 응답
 - 소스
  ```
    Page<Member> page = ...;

    page.getContent()       // 실제 데이터 List
    page.getTotalElements() // 전체 데이터 수 (100개)
    page.getTotalPages()    // 전체 페이지 수 (10페이지)
    page.getNumber()        // 현재 페이지 (0)
    page.isLast()           // 마지막 페이지 여부    
  ``` 
 
### 4. QueryDSL & Paging
 ```
  // Pageable을 받아서 QueryDSL 쿼리에 적용하는 것뿐!
  .offset(pageable.getOffset())   // Pageable에서 꺼내서
  .limit(pageable.getPageSize())  // QueryDSL에 넣어주는 것
 ```

## 3. Spring - Paging
### 1. Cotnroller
 - 컨트롤러 내부에서 처리
 ```
    @GetMapping("/members")
    public MemberPageResponse getMembers(
            @RequestParam String username,
            Pageable pageable          // Spring이 자동으로 변환해줌
    ) {
        Page<Member> page = memberService.searchPage(username, pageable);
        return new MemberPageResponse(page); 
    }
 ```
 - Response
   - Page<T> 타입으로 그대로 리턴할 경우 모든 Paging 데이터가 날라감.
   - 별도 ResponseDTO에 생성자로 page<T> 타입을 받고, 원하는것만 필드에 담으면 필요한것만!
    ```
      @Getter
      public class MemberPageResponse {
      
          // Page에서 필요한 것만 꺼내서
          private List<MemberDto> content;  // 실제 데이터
          private long totalElements;       // 전체 데이터 수
          private int totalPages;           // 전체 페이지 수
          private int pageNumber;           // 현재 페이지
          private boolean isLast;           // 마지막 페이지 여부
      
          public MemberPageResponse(Page<Member> page) {
              this.content       = page.getContent()
                                       .stream()
                                       .map(MemberDto::new)  // Entity → DTO 변환
                                       .collect(Collectors.toList());
              this.totalElements = page.getTotalElements();
              this.totalPages    = page.getTotalPages();
              this.pageNumber    = page.getNumber();
              this.isLast        = page.isLast();
          }
      }
    ```
 
### 2. Repository
 ```
  @Repository
  public class MemberRepository {
  
      private final JPAQueryFactory queryFactory;
  
      public Page<Member> searchPage(String username, Pageable pageable) {
  
          // 컨텐츠 쿼리
          List<Member> content = queryFactory
              .selectFrom(member)
              .where(member.username.eq(username))
              .offset(pageable.getOffset()) // 시작위치
              .limit(pageable.getPageSize()) // 가져올 개수
              .fetch();
  
          // 카운트 쿼리
          JPAQuery<Long> countQuery = queryFactory
              .select(member.count())
              .from(member)
              .where(member.username.eq(username));
  
          //PageableExecutionUtils는 의존성추가하면 생김.
          return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchOne);
      }
  } 
 ```

## 4. Paging 분해
### 1. 특정 페이지의 특정 인덱스 데이터 꺼내기
  ```
  Page<Member> page = memberService.searchPage(pageable);
  
  // 1. content 리스트로 꺼내기
  List<Member> content = page.getContent();
  
  // 2. 원하는 인덱스 접근
  // 인덱스는 현재 페이지 기준
  Member first  = content.get(0);  // 첫 번째
  Member second = content.get(1);  // 두 번째
  Member last   = content.get(content.size() - 1); // 마지막
  ```

### 2. 페이징 정보 + 추가 데이터 함께 보내기
  ```
  @Getter
  public class MemberPageResponse {
  
      // 페이징 정보
      private List<MemberDto> content;
      private long totalElements;
      private int totalPages;
      private int pageNumber;
      private boolean isLast;
  
      // 👇 추가로 함께 보낼 데이터
      private MemberSummary summary;      // 특정 타입
      private long totalOrderCount;       // 추가 필드
  
      public MemberPageResponse(Page<Member> page, MemberSummary summary, long totalOrderCount) {
          // Page 풀어서 담기
          this.content       = page.getContent()
                                   .stream()
                                   .map(MemberDto::new)
                                   .collect(Collectors.toList());
          this.totalElements = page.getTotalElements();
          this.totalPages    = page.getTotalPages();
          this.pageNumber    = page.getNumber();
          this.isLast        = page.isLast();
  
          // 추가 데이터 담기
          // 별도 편의성 메소드로 진행함.
          this.summary        = summary;
          this.totalOrderCount = totalOrderCount;
      }
  }
  ```
