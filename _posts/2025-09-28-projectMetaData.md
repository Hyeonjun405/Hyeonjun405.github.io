---
title: Spring initializr
date: 2025-09-28 10:00:00 +09:00
categories: [Spring Framework, SpringNote]
tags: [ Spring Framework, Spring Note ]
---

### 1. Spring initializr
 - 스프링프로젝트 생성할때 사용
 - https://start.spring.io

### 2. 스프링부트 실행시 메타데이터 설정

| 항목           | 설명                                                                                                                                       | 이번 설정               |
| ------------ |------------------------------------------------------------------------------------------------------------------------------------------| ------------------- |
| Project      | 프로젝트 유형을 선택합니다. 프로젝트의 의존성을 관리하기 위한 도구가 결정됩니다.                                                                                            | Gradle - Groovy     |
| Language     | 프로그래밍 언어를 선택합니다.                                                                                                                         | Java                |
| Spring Boot  | 스프링 부트 버전을 선택합니다. <br> SNAPSHOT은 개발 중인 최신 상태를 나타내지만 자주 업데이트되어 안정적이지 않습니다.<br> Mx는 임의의 숫자는 비교적 안정적이지만 아직 정식 출시 전 버전입니다.<br> 따라서 보통은 안정 버전을 선택합니다. | 3.3.3 ※1            |
| Group        | 일반적으로 조직명이나 회사명이 들어가며, 자바의 패키지명으로도 사용됩니다.                                                                                                | com.example         |
| Artifact     | 프로젝트명입니다. 이 이름이 그대로 JAR 파일이나 WAR 파일의 이름이 됩니다.                                                                                            | LombokSample        |
| Name         | 프로젝트의 표시명입니다. 일반적으로 Artifact와 동일하게 설정합니다.                                                                                                | LombokSample        |
| Description  | 프로젝트 설명입니다.                                                                                                                              | 선택 사항               |
| Package name | 자바 패키지명입니다. 보통 Group+Artifact를 통해 자동으로 생성됩니다.                                                                                            | com.example.demo ※2 |
| Packaging    | Jar: 자바 독립형 애플리케이션을 생성합니다.<br> War: 웹 애플리케이션을 생성합니다.<br> ※ 외부 웹 서버에 배포할 때 선택합니다.                                                         | Jar                 |
| Java         | 사용할 자바 버전을 선택합니다. 보통은 안정 버전을 선택합니다.                                                                                                      | 21                  |
