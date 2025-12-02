---
title: 10 JSCH
date: 2025-12-02 10:00:00 +09:00
categories: [note, JavaNote]
tags: [Java]
---

## 1. Memo(20215.12.02.)
 - 스프링 환경에서 RestAPI 사용중,
 - 배치가 스케줄러로 1회 호출하여 순차적으로 1작업 루틴(연결-작업-제거)씩 하는 상황이었으나,
 - 외부 요청에서 하는 작업으로 변경하면서 동시에 실행하는 상황이 생김.
 - 따라서 기존에 Component로 만들어서 주입하던 방식을 사용하면, 스레드에 불안전함!
 - 커넥션만 별도로 분리하고, 필요한 작업만 서비스단에서 처리할것!

## 2. JSCH
### 1. JSCH
 - 자바에서 SSH 기반 기능을 구현하기 위한 라이브러리
 - SSH 서버에 접속해서 명령을 실행하거나, SFTP로 파일을 업로드·다운로드하거나, 포트포워딩을 할 수 있게 만들어주는 도구

### 2. JSCH 특징
 - 자바로 구현된 순수 라이브러리라, 별도 플랫폼 의존성 없이 어디서든 사용 가능
 - SSH 프로토콜을 사용해 원격 서버와 안전하게 통신
 - SFTP 기능을 통해 파일 송수신 가능
 - SSH 터널링(포트포워딩)도 지원
 - 스프링과는 무관하며, 그냥 자바 프로젝트에서 가져다 쓰면 되는 외부 라이브러리

## 3. JSCH 기본형태
### 1. 기본 형태
  ```
    JSch jsch = new JSch();
    
    // SSH 로그인 정보 설정
    Session session = jsch.getSession("username", "host", 22); //유저이름, IP,포트
    session.setPassword("password"); //패스워드
    
    // 호스트 키 검증 생략(테스트용)
    session.setConfig("StrictHostKeyChecking", "no");
    
    // 세션 연결
    session.connect();
    
    // 채널 생성 및 작업
    ChannelExec channel = (ChannelExec) session.openChannel("exec");  
       
    // 종료
    channel.disconnect();
    session.disconnect();
  ```

### 2. 채널종류
 
  | 채널 종류           | 설명                 | 사용 예시                  | 사용 빈도 |
   | --------------- | ------------------ | ---------------------- | ----- |
   | exec            | 원격 서버에서 단일 명령 실행   | `ls -al`, `mkdir test` | 높음    |
   | shell           | 인터랙티브 쉘 연결         | 서버 터미널 접속, 스크립트 실행     | 중간    |
   | sftp            | SFTP를 통한 파일 업/다운로드 | 파일 업로드/다운로드            | 높음    |
   | subsystem       | SSH 서브시스템 호출       | SFTP, 기타 서버 서브시스템      | 낮음    |
   | direct-tcpip    | 로컬→원격 포트포워딩        | DB 접속 터널링, SSH 터널      | 중간    |
   | forwarded-tcpip | 원격→로컬 포트포워딩        | 원격 서비스 로컬 접근           | 낮음    |
   | x11             | X11 GUI 포워딩        | 리눅스 GUI 프로그램 실행        | 매우 낮음 |

#### 1. exec 채널
  ````
  // exec 채널 생성
  ChannelExec execChannel = (ChannelExec) session.openChannel("exec");
  execChannel.setCommand("ls -al");
  
  // 출력 스트림
  InputStream in = execChannel.getInputStream();
  execChannel.connect();
  
  // 결과 출력
  byte[] buffer = new byte[1024];
  while (true) {
      int read = in.read(buffer);
      if (read == -1) break;
      System.out.print(new String(buffer, 0, read));
  }
  
  // 종료
  execChannel.disconnect();
  session.disconnect();
  ````

#### 2. sftp 채널
  ```
  // SFTP 채널 생성
  ChannelSftp sftpChannel = (ChannelSftp) session.openChannel("sftp");
  sftpChannel.connect();
  
  // 파일 업로드
  sftpChannel.put("C:/local/file.txt", "/remote/path/file.txt");
  
  // 파일 다운로드
  sftpChannel.get("/remote/path/file.txt", "C:/local/file.txt");
  ```

#### 3. direct-tcpip 채널 (포트포워딩)
  ```
  // 로컬 포트포워딩 설정 (ChannelDirectTCPIP 객체 안 만듦)
  session.setPortForwardingL(3306, "10.0.0.5", 3306);
  
  // JDBC 연결 (터널 통해 원격 DB 접속)
  Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "user", "pass");
  ```

 - 세션에서 setPortForwardingL(3306, "10.0.0.5", 3306)을 설정하면,
 - 세션이 살아 있는 동안 OS 수준에서 로컬 포트 3306으로 들어오는 모든 트래픽이 SSH 세션을 통해 원격 10.0.0.5:3306으로 전달함

## 3.스프링에서 JSCH
### 1. 커넥션분리
  ```
    @Component
    public class JSchManager {
    
      private String username = "user";
      private String host = "host.com";
      private int port = 22;
      private String password = "password";
  
      public Session getSession() throws JSchException {
          JSch jsch = new JSch(); // 세션을 신규로 생성함.
          Session session = jsch.getSession(username, host, port);
          session.setPassword(password);
          session.setConfig("StrictHostKeyChecking", "no");
          session.connect();
          return session; // 돌려주는 세션 값이 신규 객체라 스레드에 안전함. 
      }
    }
  ```

### 2. 서비스단
  ```
    @Service
    public class RemoteService {
    
        private final JSchManager jschManager;
    
        public RemoteService(JSchManager jschManager) {
            this.jschManager = jschManager;
        }
    
        public String runCommand() {
            Session session = null;
            try {
                session = jschManager.getSession(); // 매번 새 Session 생성받기 때문에 안전.
                ChannelExec channel = (ChannelExec) session.openChannel("exec");
                channel.setCommand("ls -al");
                InputStream in = channel.getInputStream();
                channel.connect();
    
                // 결과 처리
                channel.disconnect();
                return new String(in.readAllBytes());
    
            } catch (Exception e) {
                e.printStackTrace();
                return null;
            } finally {
                if (session != null && session.isConnected()) {
                    session.disconnect(); 
                }
            }
        }
    }
  ```

## 4. 비슷한 라이브러리
### 1. Apache Mina SSHD
 - 아파치 재단에서 제공하는 SSH/SFTP 라이브러리
 - 클라이언트·서버 모두 구현 가능
 - 유지보수가 활발한 편
 - JSCH보다 구조가 명확하고 확장성이 좋다는 평가가 많음
 - 최근 스프링에서도 SSH 기능이 필요하면 Mina 기반 도구를 쓰는 경우도 있음

### 2. SSHJ
 - JSCH의 현대적인 대안으로 많이 언급됨
 - API가 단순하고 직관적
 - 키 관리, 암호화 알고리즘 지원이 더 풍부
 - JSCH보다 코드 작성 난이도가 낮음
 - SFTP도 안정적으로 지원

### 3. Ganymed SSH-2
 - 예전에는 JSCH와 많이 비교되던 라이브러리
 - 한동안 업데이트가 끊겼지만 아직도 사용 중인 레거시 프로젝트가 있음
 - 경량 SSH 클라이언트

### 4. JCraft SCP/SFTP 확장 라이브러리들
 - JSCH 기반 확장 모듈들이 존재
 - SFTP 기능만 간단하게 쓰고 싶을 때 가벼운 옵션

### 5. 비교

| 구분          | JSCH                 | SSHJ                  | Apache Mina SSHD    |
| ----------- | -------------------- | --------------------- | ------------------- |
| 개발/유지보수     | 업데이트 거의 없음 (레거시 수준)  | 꾸준한 유지보수              | 아파치 재단에서 활발히 유지     |
| 사용 난이도      | 난이도 있음, API 직관성이 낮음  | 가장 쉬움, API 구조 명확      | 중간 정도 (확장성 중심 설계)   |
| SSH 클라이언트   | 지원                   | 지원                    | 지원                  |
| SSH 서버 구현   | 불가                   | 불가                    | 가능 (서버/클라이언트 모두 지원) |
| SFTP        | 안정적이지만 오래된 방식        | 안정적, 현대적 암호화 지원       | 완전 지원               |
| 암호화 알고리즘 지원 | 구식, 최신 알고리즘 부족       | 최신 알고리즘 대부분 지원        | 최신 알고리즘 지원          |
| 문서/예제       | 적고 오래됨               | 풍부                    | 공식 문서 매우 자세함        |
| 성능          | 경량, 빠른 편             | 경량, 빠르고 안정적           | 무겁지만 안정적            |
| 장점          | 단순 구조, 오래 사용되어 정보 풍부 | 직관적, 최신 기술 대응, 사용성 좋음 | 확장성 최고, 서버 구현 가능    |
| 단점          | 오래됨, 유지보수 부족         | 약간의 학습 필요             | 무거움, 설정 복잡          |
| 추천 용도       | 레거시 프로젝트 유지          | 최신 프로젝트에서 SSH/SFTP    | 고급 기능·서버 구현이 필요한 경우 |
