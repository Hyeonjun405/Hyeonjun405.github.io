---
title: 04 kafka Spring
date: 2026-02-04 10:00:00 +09:00
categories: [infra, kafkaInflearn]
tags: [ infra, kafkaInflearn ]
---

## 1. Spring으로 Producer 생성
### 1. Memo
 - kafka에서 Producer 역할
 - 카프카에 특정 메시지를 생성하는 역할을 함.
 - 메일을 보낸다는 가정하에 카프카에 순서대로 메일을 계속보냄 

### 2. 의존성 & 프로퍼티스
 - 의존성
  ```
   ~~~
   dependencies {
	    implementation 'org.springframework.boot:spring-boot-starter-kafka' // kafka 의존성 추가  
	    implementation 'org.springframework.boot:spring-boot-starter-webmvc'
	    developmentOnly 'org.springframework.boot:spring-boot-devtools'
	    testImplementation 'org.springframework.boot:spring-boot-starter-kafka-test'
	    testImplementation 'org.springframework.boot:spring-boot-starter-webmvc-test'
	    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
  }
  ```
 - 프로퍼티스 값
  ```
   # Kafka 서버 주소 (EC2에 카프카를 설치했기 때문에 EC2 주소를 입력해야 한다.)
   spring.kafka.bootstrap-servers=IP:port
   # 메시지의 key 직렬화 방식 : 자바 객체를 문자열(String)로 변환해서 Kafka에 전송
   spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
   # 메시지의 value 직렬화 방식 : 자바 객체를 문자열(String)로 변환해서 Kafka에 전송
   spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
  ```
 - yaml형태
  ```
  spring:
    kafka:
      bootstrap-servers: 15.164.96.71:9092
      producer:       
        key-serializer: org.apache.kafka.common.serialization.StringSerializer
        value-serializer: org.apache.kafka.common.serialization.StringSerializer
  ```

### 3. Spring Class
#### 1. Controller 관련
 - Controller
  ```
      //특이사항없이 바로 진행함.
      @PostMapping
      public ResponseEntity<String> sendEmail(@RequestBody SendEmailRequestDto sendEmailRequestDto
      ) {
          emailService.sendEmail(sendEmailRequestDto);
          return ResponseEntity.ok("이메일 발송 요청 완료");
      }
  ```
 - sendEmailRequestDTO (넘어온 메일 내용 / getter+setter)
  ```
    private String from;         // 발신자 이메일
    private String to;           // 수신자 이메일
    private String subject;      // 이메일 제목
    private String body;         // 이메일 본문
  ```

#### 2. Service
 - Serivce
   ```
    // <메시지의 Key 타입, 메시지의 Value 타입>
    private final KafkaTemplate<String, String> kafkaTemplate;

    public EmailService(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendEmail(SendEmailRequestDto request) {
        EmailSendMessage emailSendMessage = new EmailSendMessage(
                request.getFrom(),
                request.getTo(),
                request.getSubject(),
                request.getBody()
        );

        // 생성자에서 메시지의 Value타입을 String으로 설정하여 json(DTO)을 Spring으로 변경 처리
        // 토픽 이름: "email.send"
        // 메시지 value: toJsonString(emailSendMessage) 결과값
        this.kafkaTemplate.send("email.send", toJsonString(emailSendMessage));
    }

    // 객체를 Json 형태의 String으로 만들어주는 메서드
    private String toJsonString(Object object) {
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            String message = objectMapper.writeValueAsString(object);
            return message;
        } catch (JacksonException e) {
            throw new RuntimeException("Json 직렬화 실패");
        }
    }   
   ```
 - EmailSendMessage DTO ( 카프카에 보낼 메시지 / getter+setter)
   ```
    private String from;         // 발신자 이메일
    private String to;           // 수신자 이메일
    private String subject;      // 이메일 제목
    private String body;         // 이메일 본문
   ```

## 2. Spring으로 Consumer(기본형) 생성
### 1. Memo
 - 기존에 Producer로 만들었던 패턴 그대로 Consumer에서 사용이 필요.
 - 학습과정에서 Producer과 Consumer이 동일한 컴퓨터라면 서버 분리도 필요!
 
### 2. 의존성 & 프로퍼티스
 - 의존성은 동일
 - 프로퍼티스
  ```
   server:
     port: 0 # 사용 가능한 랜덤 포트를 찾아서 서버를 실행 (Producer 서버와의 포트 충돌을 방지)

   spring:
     kafka:
       # Kafka 서버 주소 
       bootstrap-servers: 15.164.96.71:9092
       consumer:
        # 메시지의 key 역직렬화 방식 : Kafka에서 받아온 메시지를 String으로 변환
        key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        # 메시지의 value 역직렬화 방식 : Kafka에서 받아온 메시지를 String으로 변환
        value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        
        # 컨슈머 그룹이 미리 안 만들어져있는 경우에, 컨슈머 그룹을 직접 생성해서 메시지를 처음부터 읽음.
        # 만약 컨슈머 그룹이 이미 만들어져있다면, 해당 컨슈머 그룹이 읽었던 메시지부터 읽음.
        
        # 이 옵션을 주지 않으면 컨슈머 그룹을 직접 생성해서 메시지를 읽을 때, 
        # 기존에 쌓여있던 메시지를 읽지 않고 컨슈머 그룹이 생성된 이후에 들어온 메시지부터 읽음
        # 그럼 컨슈머 그룹이 생성되기 전에 쌓여있던 메시지들이 처리되지 않고 누락됨
        auto-offset-reset: earliest
  ```

### 3. Spring Class
#### 1. Controller
  ```
  @KafkaListener(topics = "email.send", groupId = "email-send-group") // 컨슈머 그룹 이름
  public void consume(String message) {
     System.out.println("Kafka로부터 받아온 메시지: " + message);
    
     EmailSendMessage emailSendMessage = EmailSendMessage.fromJson(message);

     // 메일 발송 로직

     System.out.println("이메일 발송 완료");
  }
  ```

#### 2. EmailSendMessage DTO (카프카로부터 받을 객체)
  ```
    private String from;         // 발신자 이메일
    private String to;           // 수신자 이메일
    private String subject;      // 이메일 제목
    private String body;         // 이메일 본문
  
    // 역직렬화(String 형태의 카프카 메시지 -> Java 객체)시 필요함
    public EmailSendMessage() { }
    
    // Json 값을 EmailSendMessage로 역직렬화하는 메서드
    // Json형태를 String으로 변환해서 카프카에 줬으므로,
    // Consumer에서도 다시 JSon으로 변경해서 사용이 필요함.
    public static EmailSendMessage fromJson(String json) {
      try {
        ObjectMapper objectMapper = new ObjectMapper();
        return objectMapper.readValue(json, EmailSendMessage.class);
      } catch (JsonProcessingException e) {
        throw new RuntimeException("JSON 파싱 실패");
      }
  }
  ```

## 3. Spring으로 Consumer 개선 - 재시도
### 1. Memo
- Producer + Consumer의 한계는 Producer에서 응답에 대한 답변이 먼저 가기 때문에,
- Consumer에서 실제로 그 작업이 정상처리인지 요청자에게 확인이 안됨.

### 2. 기본 설정
 - 에러로그
   ```
   Backoff FixedBackOff{interval=0, currentAttempts=10, maxAttempts=9} …
   ```
  - `interval` : 재시도를 하는 시간 간격 (ms) // `interval=0`일 경우 실패하자마자 즉시 재시도
  - `maxAttempts` : 최대 재시도 횟수 
  - `currentAttempts` : 지금까지 시도한 횟수 (최초 시도 횟수 + 재시도 횟수)

### 3. 값 변경
 ```
  @KafkaListener(
      topics = "email.send",
      groupId = "email-send-group" // 컨슈머 그룹 이름
  )
  // 추가된부분 start
  @RetryableTopic(
      // 총 시도 횟수 (최초 시도 1회 + 재시도 4회)
      attempts = "5", 
      // 재시도 간격 (1000ms -> 2000ms -> 4000ms -> 8000ms 순으로 재시도 시간이 증가한다.)
      backoff = @Backoff(delay = 1000, multiplier = 2) 
  )
  // 추가된부분 end
  public void consume(String message) {
 ```

## 3. Spring으로 Consumer 개선 - 재시도 후 실패
### 1. memo
 - 지속적인 시도에도 불구하고 실패한경우 DLT를 이용해서 진행함.

### 2. Dead Letter Topic
 - 오류로 인해 처리할 수 없는 메시지를 임시로 저장하는 토픽
 - Kafka에서는 재시도까지 실패한 메시지를 다른 토픽에 따로 저장해서 유실을 방지하고 후속 조치를 가능하게함.
 - 이점
   - 실패한 메시지를 DLT 토픽에 저장해놓기 때문에, 실패한 메시지가 유실되는 걸 방지할 수 있음.
   - DLT 토픽에 실패한 메시지가 저장되어 있기 때문에, 사후에 실패 원인을 분석할 수 있음.
   - DLT 토픽에 실패한 메시지가 저장되어 있기 때문에, 처리되지 못한 메시지를 수동으로 처리함.

### 3. 기본설정
 - @RetryableTopic을 사용하면 자동으로 DLT 토픽을 생성하고 메시지를 전송
 - Default DLT 토픽 이름은 {기존 토픽명}-dlt 형태

### 4. DLT 설정 변경작업
  ```
    @KafkaListener(
      topics = "email.send",
      groupId = "email-send-group" // 컨슈머 그룹 이름
    )
    @RetryableTopic( // 이것을 사용하면 자동으로 DLT를 생성함.
      // 총 시도 횟수 (최초 시도 1회 + 재시도 4회)
      attempts = "5",

      // 재시도 간격 (1000ms -> 2000ms -> 4000ms -> 8000ms 순으로 재시도 시간이 증가한다.)
      backoff = @Backoff(delay = 1000, multiplier = 2),

      // DLT 토픽 이름에 붙일 접미사 (수동)
      dltTopicSuffix = ".dlt"
    )
    public void consume(String message) {
  ```
 - 실패할경우
  ```
   Received message in dlt listener : email.send.dlt-8@8 패턴
  ```

### 5. DLT 처리하는 Consumer
  ```
   @KafkaListener(
            topics = "email.send.dlt", // dlt 토픽을 읽는
            groupId = "email-send-dlt-group" // 그룹 설정
    )
    public void consume(String message) {

        // 로그 전송
        // 알림등
    }
  ```
