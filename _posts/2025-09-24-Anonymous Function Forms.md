---
title: 익명함수
date: 2025-09-24 10:00:00 +09:00
categories: [Java, JavaNote]
tags: [Java]
---

## 1. 익명함수
### 1. 익명 함수란
 - 이름 없이 정의되는 일회성 함수
 - 주로 인터페이스나 클래스의 구현체로 사용
 - 한 번만 사용되는 동작을 간결하게 구현

### 2. 자바에서 익명 함수 등장 배경 
 - 자바는 함수 자체를 독립적으로 정의하는 구조가 없음 (함수 = 클래스 메서드에 종속)
 - 하지만 특정 행동(동작)을 전달해야 하는 상황이 많음
 - 스레드 실행, 이벤트 처리, 컬렉션 처리 등

### 3. 예시
```
  List<String> fruits = Arrays.asList("Banana", "Apple", "Cherry");

        // Collections.sort의 두번째 메소드는 정렬기준
        // 익명 클래스로 Comparator 구현
        Collections.sort(fruits, new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return s2.length() - s1.length(); // 길이 기준 내림차순
            }
        });

        System.out.println(fruits); // [Banana, Cherry, Apple]
```

## 2. 람다
### 1. 람다
 - 익명 함수(이름 없는 함수)를 표현하는 자바 문법
 - Java 8부터 등장, 함수형 인터페이스를 구현할 때 주로 사용
 - 코드를 간결하게 하고, 함수형 프로그래밍 스타일 지원

### 2. 람다 문법
```
 (파라미터) -> { 실행 코드 }
 // 파라미터: 입력값
 // ->: 화살표, “입력 → 실행”
 // 실행 코드: 단일식이면 {} 생략 가능, 결과값 반환 가능
```

### 3. 예시
```
 Collections.sort(fruits, (s1, s2) -> s2.length() - s1.length());
 // Collections.sort(fruits, ...) fruits 리스트를 정렬, 
 // (s1, s2) -> s2.length() - s1.length() 익명 함수 형태로 정렬 기준을 정의 
```

## 3. 메서드참조
### 1. 메서드 참조
 - 이미 존재하는 메서드를 그대로 참조해서 함수형 인터페이스의 구현으로 사용하는 문법
 - 람다식을 더 간결하게 쓰기 위해 등장

### 2. 기본 문법
```
    클래스이름::메서드이름
```

### 3. 예시
```
Collections.sort(fruits, Comparator.comparingInt(String::length).reversed());
```
 - Collections.sort(fruits, ...)
   - fruits 리스트를 정렬
   - 첫 번째 파라미터: 정렬할 리스트 자체
 - Comparator.comparingInt(String::length)
   - String::length → 문자열 길이를 반환하는 메서드 참조
   - comparingInt(...) → 이 반환값(int)을 기준으로 Comparator 생성
   - “문자열 길이 기준으로 오름차순 정렬하는 Comparator” 생성
 - .reversed()
   - 기본적으로 comparingInt는 오름차순
   - .reversed()를 붙이면 내림차순 정렬

## 4. 익명 클래스 vs 람다식 vs 메서드 참조
### 1. 비교

| 구분     | 문법                    | 장점                   | 단점           | 예제 사용 시점            |
| ------ | --------------------- | -------------------- | ------------ | ------------------- |
| 익명 클래스 | `new 인터페이스(){...}`    | 자바 7 이전에도 사용 가능, 명시적 | 길고 반복 시 불편   | 한 번만 쓰거나 람다/참조 불가 시 |
| 람다식    | `(파라미터) -> { 실행 코드 }` | 코드 간결, 직관적           | 복잡하면 가독성 ↓   | 단순한 동작, 함수형 스타일     |
| 메서드 참조 | `클래스::메서드`            | 가장 간결, 재사용 용이        | 커스텀 로직 적용 어렵 | 이미 존재하는 메서드를 바로 활용  |
