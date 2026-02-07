---
title: 06 kafka 장애대응
date: 2026-02-07 10:00:00 +09:00
categories: [infra, kafkaInflearn]
tags: [ infra, kafkaInflearn ]
---

## 1. kafka 인프라 관련 개념
### 1. 전체 흐름
 ![내 그림](assets/img/infra/kafka_inflearn/06/노드 및 클러스터.png "이미지")

### 2. 노드(node)
 - 카프카가 설치되어 있는 서버 단위를 의미
 - 노드가 고장나게 되면 메시지를 전달하는 것 자체가 막히기 때문에 서비스 장애가됨
 - 실무에서는 위 그림과 같이 노드를 1대만 두지 않고, 최소 3대의 노드를 구성
 
### 3. 클러스터(cluster)
 - 유기적으로 작동하는 노드들을 묶어서 클러스터
 - 여러 대의 서버가 연결되어 하나의 시스템처럼 동작하는 서버들의 집합을 의미
 - 메시지를 나눠 저장 / 복제본을 생성해서 유지하는 패턴
 
### 4. 노드의 구성
#### 1. 브로커(broker), 컨트롤러(controller)
 - kafka 서버는 크게 컨트롤러(controller)와 브로커(broker)로 구성
   - 브로커는 메시지를 저장하고 클라이언트의 요청을 처리하는 역할
   - 컨트롤러는 브로커들간의 연동과 전반적인 클러스터의 상태를 총괄
 - 노드에서 브로커는 9092번 포트 / 컨트롤러는 9093번 포트

#### 2. 레플리케이션(replication)
 - 데이터의 안정성과 가용성을 높이기 위해 토픽의 파티션을 여러 노드에 복제하는 것
 - 복제된 파티션들은 리더 파티션(원본) / 팔로워 파티션(복제본) 구분
   - 리더 파티션은 프로듀서나 컨슈머가 직접적으로 메시지를 쓰고 읽는 파티션
   - 팔로워 파티션은 프로듀서나 컨슈머가 직접적으로 메시지를 쓰고 읽지 않음
   - 팔로워 파티션은 리더 파티션의 메시지를 실시간으로 복제하며 유지
 - 리더 파티션에 장애가 발생하면 팔로워 파티션이 리더 역할을 대신 수행

## 2. 카프카 클러스터 설정하기
### 1. 서버 프로퍼티스 변경(config/server.properties)
 ```
  # kafka 노드를 식별하는 ID
  node.id=1

  # 클러스터를 구성할 컨트롤러의 노드 주소 목록을 설정
  # (추가적인 노드의 컨트롤러를 19093, 29093번 포트에 실행시킬 예정)
  # 일반적으로 다른 컴퓨터에 설최되므로, EC2 Public IP는 다르고 포트번호가 같아짐
  controller.quorum.bootstrap.servers={EC2 Public IP}:9093,{EC2 Public IP}:19093,{EC2 Public IP}:29093

  # 브로커, 컨트롤러 프로세스를 실행시킬 포트를 지정
  # (브로커를 PLAINTEXT, 컨트롤러를 CONTROLLER라고 지칭)
  listeners=PLAINTEXT://:9092,CONTROLLER://:9093

  # 외부에서 접근할 수 있는 주소
  advertised.listeners=PLAINTEXT://{EC2 Public IP}:9092,CONTROLLER://{EC2 Public IP}:9093

  # kafka가 데이터(kafka 설정, 브로커가 받은 메시지, 로그 등)를 저장할 디렉터리 경로 설정
  # 동일한 컴퓨터에서 진행할 경우에는 디렉토리 구분 필요
  log.dirs=/tmp/kafka-logs-1
 ```

### 2. 카프카 실행
 ```
  # 처음 실행하는 kafka 노드는 아래 명령어로 실행
  $ KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
  $ KAFKA_CONTROLLER_ID="$(bin/kafka-storage.sh random-uuid)"
  $ bin/kafka-storage.sh format \ # 초기화
    -t $KAFKA_CLUSTER_ID \ # 클러스터 번호
 	  -c config/server.properties \  # 프로세스 ID
	  --initial-controllers "1@localhost:9093:$KAFKA_CONTROLLER_ID" 
 	  # 마지막 명령어는 클러스터의 최초 컨트롤러가 누구인지 등록하는 작업으로, 첫번째 노드만 사용함.
	  # 1 : node.id (이 서버 ID)
	  # localhost:9093 : 이 노드 주소
	  # $KAFKA_CONTROLLER_ID : 이 컨트롤러 프로세스 고유 ID
	 
	$ bin/kafka-storage.sh format \ # 초기화
    -t $KAFKA_CLUSTER_ID \ # 이 클러스터 소속, 이값은 최초 컴퓨터에서 복사해서 붙여넣기해야함.
	  -c config/server2.properties \ # 설정파일
	  --no-initial-controllers # 만들어진 클러스터에 참여함.
 ```

### 3. 컨트롤러 쿼럼 등록
 ```
 $ bin/kafka-metadata-quorum.sh \ 
	 --command-config config/server2.properties \
	 --bootstrap-server localhost:9092 \
	 add-controller
	 
 # 최초로 등록했던
 --initial-controllers "1@localhost:9093:$KAFKA_CONTROLLER_ID" 
 카프카가 죽을 경우
 다음 컨트롤러 후보군으로 등록함. 	 
 ```

### 4. 토픽 등록
 ```
 $ bin/kafka-topics.sh \
	--bootstrap-server localhost:9092 \
	--create \
	--topic email.send \
	--partitions 1 \
	--replication-factor 3 # 여기서 
 ```

## 3. 카프카 현황 및 관계
### 1. 현황 조회 CLI
 ```
  $ bin/kafka-topics.sh \
   --bootstrap-server localhost:9092 \
   --describe \
   --topic email.send
   
  # Replicas와 Isr에 3개의 숫자(1, 2, 3)가 다 있다면 3개의 Kafka 서버가 정상적으로 잘 연동되고 있다는 뜻
 ```
  - `PartitionCount` : 해당 토픽의 파티션 수
  - `ReplicationFactor` : 해당 토픽의 레플리케이션 수
  - `Partition` : 파티션 번호
  - `Leader` : 해당 토픽의 리더 파티션을 가지고 있는 노드 id
  - `Replicas` : 해당 토픽의 파티션을 복제하기로 설정된 노드들의 id
  - `Isr`(In-Sync Replicas) : 리더 파티션과 똑같은 상태로 복제(동기화)가 완료된 노드들의 id

### 2. Memo
### 1. 파티션 / 레플리케이션관계
 ```
  partitions = N
  replication.factor = M
 ```

### 2. 1:M
 - 파티션이 1개, 브로커가 여러개 일경우
 - 리더의 데이터를 다른 브로커가 가지고 있는 구조가됨. 
 - 구성

  | 브로커 | 역할       | 가지고 있는 데이터 |
  | --- | -------- | ---------- |
  | 서버1 | Leader   | P0 원본 로그   |
  | 서버2 | Follower | P0 복제 로그   |
  | 서버3 | Follower | P0 복제 로그   |
  | ... | ...      | ...        |
  | 서버M | Follower | P0 복제 로그   |


### 3. N:1
 - 파티션이 N개, 브로커가 1개
 - 그러면 많은 파티션을 가진 브로커가 됨.
 - 구성

  | 파티션 | Leader | Follower |
  | --- | ------ | -------- |
  | P0  | 서버1    | 없음       |
  | P1  | 서버1    | 없음       |
  | P2  | 서버1    | 없음       |
  | ... | 서버1    | 없음       |

### 4. 2:3
 - 설정에 따라서 각 서버는 모든 파티션의 데이터를 가지고 있을수도 없을 수도 있음.
   - 따라서 여러 서버를 운용중일때 특정 서버의 묶음이 죽으면, 일부 파티션의 데이터 손실이 있을 수도 있음.
   - 그렇다고 모든 파티션의 데이터를 가지고 있으면 물리적한계로 인한 문제가 발생할 수도 있음
 - 파티션의 리더는 카프카에서 알아서 결정
   - 각 파티션별로 알아서 로그 작성을 진행하고
   - 팔로워데이터들은 설정에 따라서 다운로드 하는 상황이됨.
 - 구성
 
  | 파티션 | S1       | S2       | S3       |
  | --- | -------- | -------- | -------- |
  | P0  | Leader   | Follower | Follower |
  | P1  | Follower | Leader   | Follower |



## 4. SpringBoot에서 설정
### 1. Producer / application.yaml
  ```
  server:
    port: 0
  
  spring:
    kafka:
      bootstrap-servers:
        - {Kafka 서버 IP 주소}:9092
        - {Kafka 서버 IP 주소}:19092
        - {Kafka 서버 IP 주소}:29092
      consumer:
        key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
        auto-offset-reset: earliest
  ```

### 2. Consumer / application.yaml
 ```
 server:
  port: 0

 spring:
   kafka:
     bootstrap-servers:
       - {Kafka 서버 IP 주소}:9092
       - {Kafka 서버 IP 주소}:19092
       - {Kafka 서버 IP 주소}:29092
     consumer:
       key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
       value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
       auto-offset-reset: earliest
 ```
