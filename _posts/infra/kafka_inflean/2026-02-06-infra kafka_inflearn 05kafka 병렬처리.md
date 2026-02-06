---
title: 04 kafka 병렬처리
date: 2026-02-06 10:00:00 +09:00
categories: [infra, kafkaInflearn]
tags: [ infra, kafkaInflearn ]
---

## 1. kafka partition
### 1. partition
 - 큐(메시지를 임시로 저장할 수 있는 공간)를 여러개로 늘려서 병렬 처리를 가능하게 하는 기본 단위
 - 메시지를 순차적으로 처리하는 것보다 병렬적으로 처리하는 게 훨씬 빠름.

### 2. 파티션(Partition)의 특징
 - 토픽은 하나 이상의 파티션으로 구성(default값이 1개)
 - Producer가 특정 토픽에 메시지를 넣으면, 여러 파티션에 메시지가 적절하게 분산
 - 하나의 파티션은 하나의 컨슈머에게만 할당하지만 하나의 컨슈머가 여러 파티션을 처리할 수 있음.
 - 하나의 파티션에 할당된 하나의 컨슈머는 메시지를 순서대로 처리
  
## 2. 파티션 설정하기
### 1. 파티션 정보 조회
 ```
 # bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic email.send
 $ bin/kafka-topics.sh
   --bootstrap-server localhost:9092 
 	--describe --topic email.send #email.send의 파티션 조회
	
  	# `PartitionCount` : 토픽이 가지고 있는 파티션의 총 개수
   # `Partition` : 파티션 번호 (0번 부터 시작)
 ```

### 2. 파티션 설정 후 생성
 ```
 $ bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test.topic --partitions 3
 $ bin/kafka-topics.sh 
 	 --bootstrap-server #kafka 주소  
	 --create 
	 --topic # 토픽명
	 --partitions # 파티션 수
 ```

### 3. 기존 토픽 조정
 ```
 $ bin/kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic test.topic --partitions 5
 $ bin/kafka-topics.sh  
	 --bootstrap-server # kafka 주소
	 --alter 
	 --topic # 토픽명
	 --partitions # 변경할 최종 파티션 수
 
 # 파티션의 수는 늘릴 수는 있지만 줄이는 것은 불가능함.
 # 기존에 있던 것을 다른곳에 옮겨야하기 때문에 지원하지 않음.
 ```

## 3. 카프카 파티션 / key
### 1. 카프카 key
 - 메시지를 어떤 파티션으로 보낼지 결정하는 기준값
 - “이 이벤트들은 하나의 흐름이다”라고 묶는 식별자

### 2. 예시

  ```
  key = orderId (100) # 100번주문의 
  value = OrderCreated # 주문
  value = OrderPaid # 지불
  value = OrderShipped # 출고
  # 100번주문을 한번에 묶는 단위
  ```

### 3. 카프카 key 특징
 - 같은 키의 경우에는 같은 파티션에 들어가지만 순서는 보장이 안됨.
 - 업무 비즈니스 로직에 맞는 키값에 넣음

  ```
   # 주문 순서
   파티션1 100번 주문
   파티션2 101번 주문
   파티션2 101번 결제
   파티션1 102번 주문
   파티션1 100번 결제
   
   # 파티션 1에는 100번 주문 / 102번 주문 / 100번 결제 순으로 데이터가 저장됨.        
  ```

### 4. Memo
 - MSA환경에서 일관된 작업의 오류나 상황에 따라서 초기화 또는 트랜잭션이 불가능함.
 - DB에 넣으면 DB와 커넥션이 지속적으로 유지되어야 하므로 카프카의 목적이랑 안맞음.
 - 그래서 "주문 - 결제"패턴이라면, 결제가 안되면 컨슈머에서 주문을 처리안하게 되는 패턴
 - 최종적으로 Kafka key는 순서/묶음을 보장하고, 처리 조건(결제 여부 등)은 컨슈머 로직에서 결정함.
 - 흐름
   - Producer (주문 서비스)
    ```
     Producer (주문 서비스)
     주문 생성 이벤트 발행
     토픽: order-events
     key: orderId
     value: OrderCreated
    ```

   - Consumer 1 (주문 처리 서비스)
     - 주문 상태 확인, 재고 차감 등
     - 결제 완료 여부 확인 필요
     - 처리 로직에서 결제 안 됐으면 보류

   - Consumer 2 (결제 서비스)
     - 결제 완료 이벤트 처리
     - 토픽: order-events 혹은 payment-events
     - 결제 완료 이벤트 발행


## 4. 카프카 파티션 설정
### 1. Memo
 - “어떤 파티션을 읽고 쓰는지”는 대부분 Kafka가 자동으로 결정
   - 중간에 컨슈머가 추가되면 자동으로 재배정됨.
 - 프로듀서/컨슈머
   - Producer: key를 주거나 직접 지정하지 않으면 Kafka가 파티션 결정
   - Consumer: Kafka가 파티션 할당을 관리, 한 파티션은 한 Consumer 스레드만 처리
 - 기본값
   - Producer가 메시지를 보낼 때 key를 지정하면 기본적으로 Key 기반 파티셔닝이 적용됨
   - key가 없으면 스티키(Sticky) 파티셔닝이 기본
 - 종류

   | 전략          | 특징                  | 장점      | 단점               | 적합 사례               |
   | ----------- | ------------------- | ------- | ---------------- | ------------------- |
   | Key 기반      | key 해시로 파티션 결정      | 순서 보장   | 핫 파티션 가능         | 주문, 계좌, 상태 변화 이벤트   |
   | Round-robin | key 없는 경우 순서대로 분산   | 부하 균등   | 순서 보장 없음         | 로그, 클릭 이벤트          |
   | Sticky      | key 없는 경우 연속 메시지 묶음 | 처리량 최적화 | 전체 순서 X          | 배치형 이벤트, 대량 로그      |
   | 직접 지정       | Producer가 파티션 지정    | 완전 제어   | 핫 파티션 위험, 운영 어려움 | 특수 목적, 테스트, VIP 이벤트 |


### 2. Key 기반 (key O defualt값)
 - 개념
   - 메시지의 key를 해시 → 파티션 선택
   - 같은 key → 항상 같은 파티션
   - 순서 보장이 필요한 이벤트에 적합
 - 특징
   - 순서 보장 가능 (한 엔티티 단위)
   - 특정 key가 몰리면 핫 파티션 발생 가능
   - key 없는 메시지는 라운드로빈 또는 스티키로 처리(☆)
 - 설정 예시
  ```
   kafkaTemplate.send("order-events", orderId, event);
  ```

### 3. Round-robin 파티셔닝
 - 개념
   - key가 없으면, 메시지를 순서대로 라운드로빈으로 파티션에 배분
   - 파티션 간 부하 균등화 목적
 - 특징
   - 순서 보장 X (key 없는 메시지)
   - 단순 로그, 모니터링 이벤트에 적합
 - 설정 예시
  ```
  spring:
    kafka:
      bootstrap-servers: 15.164.96.71:9092
	      producer:
        key-serializer: org.apache.kafka.common.serialization.StringSerializer
        value-serializer: org.apache.kafka.common.serialization.StringSerializer
        # 프로퍼티스에 라운드로빈을 등록하면 라운드로빈으로 됨.
        properties:
          partitioner.class: org.apache.kafka.clients.producer.RoundRobinPartitioner
  ```

### 4. Sticky 파티셔닝 (key X defualt값)
 - 개념
   - key 없는 메시지도 연속 메시지를 같은 파티션에 묶었다가
   - 일정 수 메시지 전송 후 다른 파티션으로 이동
   - batch 처리 효율을 높임
 - 특징
   - 처리량 최적화 목적
   - 순서 일부 보장 가능 (같은 배치 안)
   - key 없는 이벤트 처리에 유리

### 5. 직접 파티션 지정
 - 개념
   - Producer가 직접 파티션 번호 지정
   - 특정 메시지를 특정 파티션에 넣고 싶을 때
 - 특징
   - 특정 파티션 집중 처리 가능
   - 핫 파티션 주의
   - 일반적으로는 key 기반 파티셔닝이 안전
 - 설정 예시
  ```
  kafkaTemplate.send("order-events", 2, orderId, event);
  ```

## 5. SpringBoot 멀티스레드 kafka
### 1. memo
- 굳이 컨슈머수를 늘리지 않고, 스프링부트 멀티스레드로 처리하는 방법.
- AP가 메모리가 어떠한 상태인지 확인이 필요함.
- 무겁고 복잡한 작업일수록 점점 컴퓨터가 버거워져 적정량 설정이 필요함.

### 2. 소스 변경
  ```
  @KafkaListener(
        topics = "email.send",
        groupId = "email-send-group",
        concurrency = "3" // 멀티스레드 수
    )
    @RetryableTopic(
        attempts = "5", 
        backoff = @Backoff(delay = 1000, multiplier = 2), 
        dltTopicSuffix = ".dlt" 
    )
    public void consume(String message) {
    ~~~~
  ```

## 6. 적정 파티션 개수 구하는법
### 1. 공식
  ```
   프로듀서가 보내는 메시지량 ≤ 하나의 쓰레드가 처리하는 메시지량 x 파티션 수
  ```

### 2. 흐름
 - 쓰레드 필요 수 확인
   - 몇 개의 쓰레드를 사용해야 요청을 가장 많이 처리할 수 있는 지 측정해야함.
   - ex) 100개의 쓰레드를 활용하는 게 가장 효율적이라고 측정했다고 가정
   
 - 하나의 컨슈머 서버가 처리할 수 있는 최대 처리량(Throughput) 확인
   - 컨슈머 서버가 적절한 쓰레드 개수를 기반으로 요청을 처리한다고 했을 때, 최대 처리량(Throughput)이 얼마나 되는 지 측정
   - ex) 하나의 컨슈머 서버(100개의 쓰레드를 활용)가 1초에 처리할 수 있는 처리량(Throughput)이 30 가정

 - 프로듀서가 보내는 평균 메시지량 확인
   - 사용자가 API 요청을 얼마나 보내는 지
   - 사용자가 1초당 평균적으로 얼마나 요청을 보내는 지를 측정하거나 예상
   - ex) 사용자가 평균적으로 1초당 100개의 메일을 보낸다고 가정

 - 처리가 지연되지 않는 선에서 파티션 개수 계산하기
    - 1초당 프로듀서가 보내는 메일의 수 : 120개(약간 더 크게 잡음)
    - 1초에 1개의 AP가 처리하는 수 : 1개 AP - 100개 스레드 - 30개 처리
    - 1초에 하나의 서버가 처리할 수 있는 수 : 0.3
    - 처리가 지연되지 않는 수 : 120 = 0.3 * 400 (필요)
    
### 3. 컨슈머 랙(Consumer Lag)
#### 1. 컨슈머 랙 
  - 지연된 메시지 수 / 컨슈머가 아직 처리하지 못한 메시지 수
  - 프로듀서의 메시지 생산량보다 컨슈머의 메시지 처리량이 작을 때 컨슈머 랙(Consumer Lag)이 발생
  - 컨슈머의 존재와 관계없이 일단 읽지 않은 데이터들은 다 컨슈머 랙

#### 2. CLI에서 조회
 ```
  bin/kafka-consumer-groups.sh \
	--bootstrap-server localhost:9092 \
	--group email-send-group \
	--describe
	
	# LAG라는 항목 존재함. 
 ```

#### 3. 외부 모니터링 툴로 조회
 - 외부 모니터링 툴을 사용해서 컨슈머 랙(Conusmer Lag)을 지속적으로 모니터링,
 - 특정 케이스에 대해 알림을 발송하게 만들어서 빠르게 대처할 수 있게함.
 - 종류
   - Datadog (유료)
   - Burrow (무료 오픈소스)
   - Prometheus, Grafana (무료 오픈소스)

#### 4. 매니지드 서비스(Managed Service)
 - 현업에서는 카프카를 직접 구축해서 사용하지 않고, 클라우드의 카프카 서비스를 사용하는 경우도 많음
 - 이런 서비스를 사용하면 자체적으로 컨슈머 랙(Consumer Lag)에 대한 모니터링 기능을 같이 제공함.
 - 종류
   - AWS MSK
   - Confluent Cloud
