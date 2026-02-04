---
title: 01 kafka
date: 2026-02-01 10:00:00 +09:00
categories: [infra, kafkaInflearn]
tags: [ infra, kafkaInflearn ]
---

## 1. kafka
### 1. kafka
 - 고성능 데이터 파이프라인, 스트리밍 분석, 데이터 통합 및 미션 크리티컬 애플리케이션에 사용되는 오픈 소스
 - 시스템들 사이에서 발생하는 데이터를 끊기지 않게, 대용량으로, 실시간에 가깝게 흘려보내고 저장하는 분산 이벤트 스트리밍 플랫폼
 - 대규모 데이터를 처리할 수 있는 메시지 큐 

### 2. 메시지 큐(Message Queue)
 - 메시지 큐(Message Queue)는 큐(Queue) 형태에 데이터를 일시적으로 저장하는 임시 저장소를 의미
 - 메시지 큐를 활용하면 비동기적으로 데이터를 처리할 수 있어서 효율적
 - 처리 구분
   - 동기적 처리 ≒ 순차적으로 처리 ≒ A 작업이 다 끝난 이후에 B 작업을 처리
   - 비동기적 처리 ≒ 병렬적으로 처리 ≒ A 작업을 시작한 직후에 B 작업도 바로 시작

### 3. kafka 역할 
 - 메시지 브로커 → 시스템 간 데이터 전달
 - 분산 로그 저장소 → 이벤트를 디스크에 안전하게 저장
 - 스트리밍 데이터 파이프라인 → 데이터가 계속 흐르는 통로
 - 실시간 데이터 백본(backbone) → MSA에서 서비스들을 이어주는 중심 축

## 2. kafka 흐름
### 1. 상황
 - 쇼핑몰 시스템
 - 주문을 처리하는 주문 AP가 존재
 - 메일 발송을 담당하는 별도 메일 AP가 존재
 - 두 시스템 사이 이벤트 전달을 위해 Kafka 사용
 - 주문 데이터는 주문 DB, 메일 관련 처리는 메일 시스템이 담당

### 2. 요청
 - 사용자가 쇼핑몰에서 주문을 수행함
 - “주문 완료”가 비즈니스적으로 발생
 - 주문 저장 + 주문 완료 메일 발송이 필요한 상태 

### 3. 로직의 흐름
#### 1. 로직 흐름
 - 주문 AP 내부 처리
 - 사용자 요청 수신
 - 주문 비즈니스 로직 수행
 - 주문 정보를 주문 DB에 저장
 - 주문이 정상 완료되었음을 확정

#### 2. 이벤트 기록 (Kafka)
 - 주문 AP는 “주문 완료됨” 이벤트 생성 (예: 주문번호, 사용자 이메일, 금액 등 포함)
 - 해당 이벤트를 Kafka 토픽에 기록
 - 여기까지가 주문 AP의 책임 범위

#### 3. 메일 AP 처리
 - 메일 AP는 Kafka를 지속적으로 구독 중
 - Kafka에 “주문 완료” 이벤트가 들어오면 이를 수신
 - 이벤트 내용 기반으로 메일 발송 비즈니스 로직 수행
 - 주문 완료 메일 사용자에게 발송

## 3. Spring - Kafka
### 1. DTO

  ```
  public class OrderCreatedEvent {
    private Long orderId;
    private Long userId;
    private int amount;
    private String email;
    
    ~~~~
  }
  ```

### 2. 송신
#### 1. 소스

  ```
  @Service
  public class OrderService {
  
      @Autowired
      private OrderRepository orderRepository;
  
      @Autowired
      private KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate;
  
      @Transactional
      /*
        1) Transactional는 DB 트랜잭션을 보장함. Kafka는 영향X
        2) Kafka와 DB는 별개의 시스템으로 운영됨
          => DB는 성공해도 Kafka가 실패할수도 있고,
          => DB가 성공하고 -> Kafka는 성공 -> DB 롤백 가능함.  
      */
      public void createOrder(CreateOrderRequest request) {
  
          // 1. DB에 주문 저장
          Order order = new Order(request.getUserId(), request.getAmount());
          orderRepository.save(order);
  
          // 2. 이벤트 생성
          OrderCreatedEvent event = new OrderCreatedEvent(
                  order.getId(),
                  order.getUserId(),
                  order.getAmount(),
                  order.getEmail()
          );
  
          // 3. Kafka로 발행
          kafkaTemplate.send("order-created", event);
  
          // 서비스 책임 종료
      }
  }
  ```

#### 2. Memo
 - @Transactional
   - Transactional는 DB 트랜잭션을 보장함. Kafka는 영향X
   - Kafka와 DB는 별개의 시스템으로 운영됨
     - DB는 성공해도 Kafka가 실패할수도 있고,
     - DB가 성공하고 -> Kafka는 성공 -> DB 롤백 가능함.
 - ListenableFuture<SendResult<K, V>> send(String topic, Integer partition, K key, V data)
  
  | 자리              | 의미     | 내용                       |
  | --------------- |--------|--------------------------|
  | topic | 토픽 이름  | 어디 저장할지                  |
  | partition  | 파티션명   | 특정 파티션 지정 가능  |
  | key             | 메시지 키  | 해시 기준 파티션 결정에 사용 가능 |
  | event           | 실제 데이터 | param/value              |


### 2. 수신
#### 1. kafka 수신
 - kafka에 추가로 등록된 기록이 있는지는 AP에서 지속적으로 확인(폴링/polling)이 필요함

   ```
    while (true) {
    records = consumer.poll();
    있으면 → @KafkaListener 메서드 호출
    }
   ```
   
 - Spring에서는 kafka 컨테이너를 만들어서 사용하고, 그 컨테이너에서 폴링작업을 대신함.
 - 그래서 Kafka을 위한 AP는 “consumer 전용 서비스”라고 부름
   - MVC모델이 아니기 때문에 굳이 컨트롤러가 불필요.
   - 단순하게 서비스와 필요한 기능들을 모아두고, 그룹화해서 사용하는 용도
   - 분리가 중요!
 - 스프링 빈으로 등록된 객체에만 @KafkaListener 붙을 수 있음.
   - 어노테이션을 실제로 인식하고 리스너 컨테이너랑 연결해주는 주체가 스프링 컨테이너
   - 기본적으로 빈으로 등록 후에 작업이 필요함.

#### 2. kafka Container
   ```
  @Configuration
  public class KafkaConsumerConfig {
  
      // 스프링부트에서 프로퍼티스 설정하면 자동으로 처리는 됨
      @Bean
      public ConsumerFactory<String, OrderCreatedEvent> consumerFactory() {
  
          // JSON → OrderCreatedEvent 객체로 변환해주는 역직렬화기
          JsonDeserializer<OrderCreatedEvent> deserializer =
                  new JsonDeserializer<>(OrderCreatedEvent.class);
  
          // 신뢰할 패키지 설정 (DTO 클래스 위치 허용)
          deserializer.addTrustedPackages("*");
  
          // Kafka Consumer 기본 설정 값들
          Map<String, Object> props = new HashMap<>();
  
          // Kafka 서버 주소 (브로커 위치)
          props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
  
          // 이 Consumer가 속한 그룹 이름
          // 같은 groupId끼리는 메시지를 나눠서 처리
          // Default값으로 애노테이션에 붙은게 우선순위가 더 높음. => 따라서 default는 필수 X
          props.put(ConsumerConfig.GROUP_ID_CONFIG, "mail-service");
  
          // key를 문자열로 변환하는 역직렬화기
          StringDeserializer keyDeserializer = new StringDeserializer();
  
          // value(JSON 데이터)를 DTO 객체로 변환하는 역직렬화기
          // 이게 있어야 @KafkaListener 파라미터로 객체가 들어옴
          return new DefaultKafkaConsumerFactory<>(
                  props,
                  keyDeserializer,
                  deserializer
          );
      }
  
      @Bean
      public ConcurrentKafkaListenerContainerFactory<String, OrderCreatedEvent>
      kafkaListenerContainerFactory() {
  
          // @KafkaListener가 사용할 컨테이너 팩토리
          ConcurrentKafkaListenerContainerFactory<String, OrderCreatedEvent> factory =
                  new ConcurrentKafkaListenerContainerFactory<>();
  
          // 위에서 만든 consumerFactory 연결
          factory.setConsumerFactory(consumerFactory());
  
          // 여러 스레드로 동시에 메시지 처리 가능 (기본 1)
          // factory.setConcurrency(3);
  
          return factory;
      }
  }
  ```

#### 3. 서비스로직

  ```
  @Component
  public class MailEventConsumer {
  
      @KafkaListener(topics = "order-created", groupId = "mail-service")
      public void handleOrderCreated(OrderCreatedEvent event) {
          // 여기 들어오면 "주문 발생" 이벤트를 읽은 상태
  
          sendMail(event);
      }
  
      private void sendMail(OrderCreatedEvent event) {
          System.out.println("메일 발송: " + event.getEmail());
      }
  }
  ```

#### 2. Memo
 - topics = "order-created"
   - Send할때 설정한 Topics
   - 데이터 묶음
 
 - groupId = "mail-service" 
   - Kafka는 "소비자 그룹 단위"로 메시지를 나눠줌.
   - 메일 서버가 2개라면, 메일그룹을 묶어서 각자 요청하면 겹치지 않게 받을 수 있음.
   - Point!
     - 다른 group이 같은 메시지를 받는 건 정상 동작이고 오히려 Kafka가 그렇게 쓰라고 만든 구조
     - “작업 지시”라기보다 “시스템에서 일어난 사건 기록”이므로 그 기록을 여러 기능에서 접근할 가능성을 열어둠.

## 4. Kafka Memo
### 1. DB로 대체
#### 1. 상황
   - 주문 AP가 DB에 주문 저장
   - 메일 AP가 그 DB를 보고 “새 주문” 처리
   - DB가 데이터 저장소 + 시스템 간 통신 수단 역할을 동시에 함.
 
#### 2. 문제
 - 강결합 생김
   - 메일 AP가 주문 DB 스키마를 알아야 함
   - 테이블 구조 바꾸면 다른 AP 다 영향

 - 실시간 처리에 약함
   - “새 데이터 있는지 계속 조회” 해야 함
   - 폴링 구조 → DB 부하 증가

 - 트래픽 커지면 DB가 병목
   - 원래 DB는 트랜잭션 처리용이지
   - 이벤트 브로커 용도로 설계된 게 아님

 - 재처리 구조가 불편
   - “지난 한 달 주문 다시 메일 보내기”

### 2. FIFO
  
  | 항목         | 일반 FIFO 큐 | Kafka           |
  | ---------- | --------- | --------------- |
  | 읽으면 메시지    | 삭제됨       | 그대로 남음          |
  | 여러 소비자     | 복잡        | 기본 기능           |
  | 과거 데이터 재처리 | 거의 불가     | 가능 (offset 되감기) |
  | 성격         | 작업 분배     | 이벤트 기록 시스템      |

### 3. Redis
  
  | 구분     | Kafka                     | Redis            |
  | ------ | ------------------------- | ---------------- |
  | 정체     | 이벤트 스트리밍 플랫폼              | 인메모리 데이터 저장소     |
  | 핵심 목적  | “흐름”을 전달                  | “데이터”를 보관        |
  | 데이터 성격 | 로그(이벤트 기록)                | 현재 상태 값          |
  | 저장 방식  | 디스크 기반, 오래 보관 가능          | 메모리 기반, 빠르지만 휘발성 |
  | 주 용도   | 비동기 처리, 시스템 연결, 이벤트 기반 구조 | 캐시, 세션, 카운터, 락   |
