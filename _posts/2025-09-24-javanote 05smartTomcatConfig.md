---
title: 05 인텔리제이 - 스마트톰캣 설정
date: 2025-09-24 10:00:00 +09:00
categories: [Java, JavaNote]
tags: [Java]
---

## 1. 인텔리제이 - servelt Project
 - 프로젝트 생성할때 아키텍처 설정 org.apache.maven.archetypes:maven-archetype-webapp
 - 메이븐 프로젝트로 간단하게 생성/스마트톰캣 자동 설정은 가능함.
 - 이렇게 안하면 매우 복잡해져서 이클립스를 사용하거나 스프링 레거시쓰는게 편해짐.
 - 스프링부트를 사용하면 부트 자체에 톰캣이 내장되어 있어서 별도로 스마트 톰캣 설정이 필요하지 않음.(☆)

## 2. Configuration
### 1. Tomcat server
 - 로컬에 설치된 톰캣 실행파일 경로 (CATALINA_HOME) 지정.
 - SmartTomcat은 여기 있는 톰캣 바이너리를 기반으로 실행함. 
 - memo
   - 웹상에서 실제 tomcat을 받고, 그에 대한 디렉토리 경로를 설정함.
   - lib, bin 디렉토리등 최상위 경로를 설정

### 2. Catalina base
 - 실제 실행 시 사용할 작업 디렉토리 (CATALINA_BASE).
 - 로그, conf, temp, work, webapps 등 실행용 리소스들이 복사되어 들어감.
 - 보통 SmartTomcat이 ~/.SmartTomcat/<프로젝트명>/<톰캣버전>에 생성함.
 - memo
   - 여기에 톰캣이 실행될때 필요한 파일이 들어감.
   - 프로젝트 별로 분리하므로 별도로 공간 셋팅해둘것

### 3. Deployment directory
 - 배포할 웹 애플리케이션 루트 디렉토리 지정.
 - 예: Maven은 src/main/webapp, Gradle은 src/main/webapp, 일반 JSP 프로젝트도 동일.
 - SmartTomcat이 이 경로를 CATALINA_BASE/webapps/<contextPath>로 매핑해서 실행.
 - memo
   - 스프링 레거시의 경우 project/~/src/main/webapp
   - 디플로이 파일이 있는 곳을 설정함.

### 4. Use classpath of module
 - 어떤 IntelliJ 모듈의 classpath를 사용해서 실행할지 선택.
 - 즉, target/classes (Maven) 또는 build/classes/java/main (Gradle) 같이 컴파일된 .class 파일과 라이브러리들을 포함해서 서버에 올려줌.
 - memo
   - 해당 프로젝트가 web프로젝트면 설정되어있으면 자동으로 프로젝트 설정됨.

### 5. Context path
 - 웹 애플리케이션의 접근 경로.
 - http://localhost:8080/<contextPath> 형식으로 접근.
 - 빈칸("")으로 두면 루트(/)로 접근 가능.
 - memo
   - 앞에 localhost는 자동으로 붙으므로 내부에서 구분할 것만 설정하면됨.

### 6. Server port
 - 실제 톰캣이 동작하는 포트 (HTTP 요청 처리 포트).
 - 보통 8080 사용. 여러 서버를 띄울 때는 충돌 안 나게 다른 포트 지정.

### 7. Admin port (Shutdown port)
 - 톰캣 프로세스를 제어하는 내부 포트.
 - catalina.bat stop 같은 명령이 이 포트를 통해 동작.
 - 일반적으로 8005, 하지만 SmartTomcat은 종종 -1로 비활성화하기도 함.

### 8. VM options
 - JVM 실행 시 전달할 옵션들.
 - 예: -Xmx1024m -Dspring.profiles.active=dev
 - 힙 메모리 크기, GC 옵션, 시스템 프로퍼티 같은 걸 넣음.

### 9. Environment variables
 - 실행 환경에서 필요한 OS 환경변수 지정.
 - 예: JAVA_OPTS, DB_HOST, DB_USER 같은 값.

### 10. Extra JVM classpath
 - Use classpath of module 외에 추가로 클래스패스를 지정할 때 사용.
 - 외부 라이브러리나 특정 폴더를 실행 시 classpath에 포함시키고 싶을 때 씀.
