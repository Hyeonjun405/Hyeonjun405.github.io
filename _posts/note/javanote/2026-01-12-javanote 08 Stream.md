---
title: 08 Stream
date: 2026-01-12 10:00:00 +09:00
categories: [note, JavaNote]
tags: [Java]
---

## 1. Stream
### 1. Stream
 - 스트림(Stream)은 데이터의 집합이 아니라 데이터 처리 파이프라인
 - 컬렉션은 데이터를 가지고, 스트림은 데이터를 처리하고, 결과를 저장하지 않음.

### 2. 특징
 - 데이터 저장 안 함
   - 스트림은 요소를 담지 않음.
   - 항상 원본 데이터 소스에서 흘려보냄.

 - 단방향
   - 한 번 지나간 요소는 다시 사용이 불가함.
   - 즉 스트림은 재사용 불가.

 - 지연 실행 (Lazy)
   - filter, map을 아무리 붙여도 바로 실행되지 않음
   - 최종 연산이 있어야만 실행

 - 선언형
   - 반복문, 인덱스, break 같은 제어 흐름이 없음
   - 조건”, “변환”, “수집”만 선언함.

### 3. 기본구조
  ```
  users.stream()          // 데이터 소스
       .filter(User::isActive)   // 중간 연산
       .map(User::getName)       // 중간 연산
       .toList();                // 최종 연산
  ```

### 4. 중간연산
- 중간연산 (Intermediate Operation)

| 구분       | 내용                                     |
| -------- | -------------------------------------- |
| 대표 메서드   | filter, map, flatMap, distinct, sorted |
| 반환값      | Stream                                 |
| 체이닝      | 가능 (연속 연결 가능)                          |
| 실행 시점    | 즉시 실행되지 않음                             |
| 역할       | 데이터 흐름을 가공, 변형                         |
| 특징       | 지연 실행(lazy), 파이프라인 구성 요소               |
| SOLID 관점 | 단일 책임 원칙에 부합 (각 연산은 하나의 역할만 수행)        |

- 최종 연산 (Terminal Operation)

| 구분       | 내용                                      |
| -------- | --------------------------------------- |
| 대표 메서드   | forEach, collect, reduce, toList, count |
| 반환값      | 컬렉션, 값, void 등                          |
| 체이닝      | 불가능 (스트림 종료)                            |
| 실행 시점    | 이 시점에 전체 스트림 실행                         |
| 역할       | 최종 결과 생성                                |
| 특징       | 스트림을 소모하며 재사용 불가                        |
| SOLID 관점 | 결과 생성 책임을 명확히 분리                        |


## 2. Stream 기본형
### 1. BookClass
  ```
  Class Book{
  
  private String title;
  prviate int price;
  prviate String publisher;
  prviate boolean isEbook;
  
    getter/setter
  }
  ```

### 2. For문으로 
  ```
    List<Book> books = new ArrayList<>();
    books.add(new Book("java", 5500, "별", false));
    books.add(new Book("xml", 2500, "달", true));
    books.add(new Book("CSS", 1555, "별", true));
    
    for(Book book : books){
      if(book.isEbook()){ // ebook인 경우
         System.out.println(book); // book을 println
      }
    }
  ```

### 3. Stream형 
  ```
  // 책 종류 println
  book.steam() // 스트림을 사용할 준비
      .filter(book -> book.isEbook()) // filter는 true/false 검증, 즉 book.isEbook()이 T/F를 검증함.
      .forEach(book -> System.out.println(book)); //필터된 book들 대상으로 println함.
  ```

### 4. 비슷한 추가 유형
  ```
  //가격 평균
  Double averageEbook
    = book.steam() // 스트림을 사용할 준비
        .filter(book -> book.isEbook()) // filter는 true/false 검증, 즉 book.isEbook()이 T/F를 검증함.
        .mapToInt(book -> book.getprice()) // 책의 가격을 스트림 형태로 
        .average() // 평균계산 
        .getAsDouble(); //Stream -> Double 형변환
  ```

## 3. Stream 정렬/중복제거
### 1. 정렬
  ```
  class Book implements Comparable<BOOK>{
  
  private String title;
  prviate int price;
  
  getter/setter
  ~~~
  
  @Override
  public int compareTo(Book book){ // 정렬을 위한 필수 오버라이딩메소드
    return this.tilte.compareTo(book.title);
  }
  ```
  ```
   List<Book> books = new ArrayList<>();
   books.add(new Book("java", 5500));
   books.add(new Book("xml", 2500));
   books.add(new Book("CSS", 1555));
   
   books.steam()
        .sotred() //는 클래스에 반드시 Comparable<T> 구현해야함. 아니면 수동으로 구현필요.
        .forEach(book -> System.out.println(book)); 
   // 이때 books는 정렬된게 아니라, 정렬된것처럼 println하는걸 구현한것.
   // 별도로 정렬하는 스트림 필요함.
  ```

### 3. 중복제거
  ```
  class Book{
  
  private String title;
  prviate int price;
  
  getter/setter
  ~~~
  
  @Override 
  public int hasCode(){ // 비교를 위한 필수 오버라이딩 메소드
    return super.hashCode(title, price);
  }
  
  @Override
  public boolean equals(Object obj){ //비교를 위한 필수 오버라이딩 메소드
    if(!(obj instanceof Book)) return false // 북이 아니면 false
    
    // 북타입이라면,
    Book book = (Book) Obj;
    return Object.equals(title, book.title)
          && price = book.pice;
  }
  ```
  ```
   List<Book> books = new ArrayList<>();
   books.add(new Book("java", 5500));
   books.add(new Book("xml", 2500));
   books.add(new Book("java", 5500));
   
   book.steam()
       .distinct() // 오버라이딩한 hashCode와 equals를 활용하여 검사 
       .forEach(book -> System.out.println(book)); 
  ```
## 4. Stream 수집/병렬처리
### 1. 수집
  ```
  class Book{
  
  private String title;
  prviate String publisher;
  
  getter/setter
  ~~~
  ```
  ```
   List<Book> books = new ArrayList<>();
   books.add(new Book("java", "태양"));
   books.add(new Book("HTML", "달빛"));
   books.add(new Book("DB", "태양"));
   
   Map<String, List<Book>> result = 
      book.steam()
         .collect( // 컬렉트는 여기 안에 사용함.
            Collectors.groupingBy(Book::getPublisher) //책 클래스의 getPublisher       
       ) 
   /* 맵 생성 과정
     - Book클래스의 getPublisher해서 나온 값이 result의 Key값이 됨,
     - 기존에 있으면 List에 추가, 없으면 신규 String으로 등록   
   */
   
   
   result.forEach((publisher, listByPublisher) -> {
        System.out.println(publisher); //분류된 키값 달빛/ 태양
        listByPublisher.forEach(System.out::println) // 
      });
  ```


### 2. 카운팅
  ```
  class Book{
  
  private String title;
  prviate String publisher;
  
  getter/setter
  ~~~
  ```
  ```
   List<Book> books = new ArrayList<>();
   books.add(new Book("java", "태양"));
   books.add(new Book("HTML", "달빛"));
   books.add(new Book("DB", "태양"));
   

  Map<String, Long> result = 
    book.steam()
       .collect( 
          Collectors.groupingBy(
            Book::getPublisher, -- 분류에 맞는
            Collectors.counting() -- 숫자를 카운팅함.
            )       
     ) 
   
   result.forEach((publisher, count) -> {
     System.out.println(publisher + " : " + count); 
   });
  ```

### 3. 병렬처리
  ```
  class Book{
  
  private String title;
  prviate String publisher;
  
  getter/setter
  ~~~
  ```
  ```
   List<Book> books = new ArrayList<>();
   books.add(new Book("java", "태양"));
   books.add(new Book("HTML", "달빛"));
   books.add(new Book("DB", "태양"));

   books.parallelStream()
        .~ 동일하게 진행
  ```

