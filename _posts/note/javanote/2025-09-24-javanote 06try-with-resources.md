---
title: 06 try-with-resources 패턴
date: 2025-09-24 10:00:00 +09:00
categories: [note, JavaNote]
tags: [Java]
---

### 1. try-with-resources
 - 자원을 자동으로 닫아주는 try문
 - 파일, 스트림, 소켓, DB 커넥션처럼 사용을 다 한 뒤에 close() 를 호출해서 반납해야 하는 객체들을 처리할 때 쓰는 구문 
 - 기본문법
   ```
   try (자원 선언) {
    // 자원 사용
    } catch (...) {
    // 예외 처리
    }
   ```

### 2. 특징
 - try() 괄호 안에 AutoCloseable 또는 Closeable 인터페이스를 구현한 객체만 넣을 수 있음
 - try 블록이 끝나기만 하면 무조건 close() 가 자동 호출됨
 - 예외가 터지든 말든 finally 안 쓰고도 안전하게 자원 반납 가능함

### 3. 예시
- 단일
  ```
   try (BufferedReader br = new BufferedReader(new FileReader("a.txt"))) {
      String line = br.readLine();
   } catch (IOException e) {
     e.printStackTrace();
   }
    /* 종료후
      if (br != null) {
        br.close();    // 자동 호출
      }
    */
   ```
- 여러종류 동시에 사용할 경우
   ```
   try (
     BufferedReader br  = new BufferedReader(new FileReader("input.txt")); // 동시에 2개
     BufferedWriter bw  = new BufferedWriter(new FileWriter("output.txt")) // 동시에 2개
   ) {
      String line;
       while ((line = br.readLine()) != null) {
       bw.write(line);
       bw.newLine();
      }
   } catch (IOException e) {
     e.printStackTrace();
   }
   // 마지막 선언부터 닫히기 시작함.
   ```

### 4. close가 없는경우
```
 // AUtoCloseable 인터페이스를 구현한 클래스를 직접 만듬. (
 class MyResource implements AutoCloseable {

   public void doSomething() { // 작업, Override X
     System.out.println("작업 수행"); 
   }

   @Override
   public void close() { // 닫기
     System.out.println("자원 닫음");
   }
 }

 public class Main {
   public static void main(String[] args) {
   try (MyResource r = new MyResource()) {  //클래스 사용하고,
     r.doSomething(); // 로직처리하면
   } 
  }
 }
```
