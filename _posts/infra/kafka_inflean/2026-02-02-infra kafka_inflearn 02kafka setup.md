---
title: 02 kafka setup
date: 2026-02-02 10:00:00 +09:00
categories: [infra, kafkaInflearn]
tags: [ infra, kafkaInflearn ]
---

## 1. kafka setup
### 1. JDK
 ```
  # Kafka를 실행시키려면 JDK 17 이상이 설치되어 있어야 함.
  $ sudo apt update
  $ sudo apt install openjdk-17-jdk
 ```

## 2. 카프카 설치
 ```
  $ wget https://dlcdn.apache.org/kafka/4.0.0/kafka_2.13-4.0.0.tgz
  $ tar -xzf kafka_2.13-4.0.0.tgz # 압축 풀기
  $ cd kafka_2.13-4.0.0 
 ```

## 2. 초기 셋팅
### 1. 메모리 관련
 ```
 # 사용환경이 메모리가 적을 경우 필요
 # AWS환경에서 무료버전 사용할 경우에 메모리가 작아서 사용 필수!
 $ export KAFKA_HEAP_OPTS="-Xmx400m -Xms400m"
 ```

### 2. swap기능
  ```
  # swap은 일부 디스크 공간을 메모리로 활용하는 기법
  # swap을 활용하면 EC2 인스턴스의 부족한 메모리 사양을 어느 정도 보완할 수 있음 
  $ sudo dd if=/dev/zero of=/swapfile bs=128M count=16 # 2GB짜리의 파일 생성
  $ sudo chmod 600 /swapfile # 파일에 권한 부여
  $ sudo mkswap /swapfile # 2GB 짜리 파일을 swap 공간의 형태로 전환
  $ sudo swapon /swapfile # swap 활성화
  ```

### 3. 시스템 설정 변경
  ```
  # 시스템 부팅 시마다 자동으로 활성화 되도록 파일시스템 수정
  $ sudo vi /etc/fstab
  
  # 아래 내용을 추가하고 저장하기
  /swapfile swap swap defaults 0 0
  ```

### 4. 접근 url 설정
 ```
 # 프로퍼티스 설정필요
 # 카프카 디렉토리/cofnig 안에서 파일 있음
 vi config/server.properties
 ```
 ```
 # 외부에서 접근할 때 사용하는 주소
 # advertised.listeners 가운데쯤에 이거 존재함 
 advertised.listeners=PLAINTEXT://{EC2 Public IP}:9092,CONTROLLER://{EC2 Public IP}:9093
 ```

## 3. 실행
### 1. 최초 실행시 
 ```
 # 초기 로그 폴더 셋팅 (카프카 첫 실행 때만 이 명령어를 입력하면 됨)
 # 공식문서에서 제공되어 있음.
 $ KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
 $ bin/kafka-storage.sh format --standalone -t $KAFKA_CLUSTER_ID -c config/server.properties
 
 $ bin/kafka-storage.sh format \ # 초기화
   --standalone \ # 단일 노드 모드 (컨트롤러 1개)
   -t $KAFKA_CLUSTER_ID \ # 이 노드가 속할 클러스터 ID
   -c config/server.properties # 어떤 설정 파일 기준으로 디렉토리 만들지
 ```

### 2. 추후 일반 실행
 ```
  # 카프카 디렉토리 안에서 실행.
  # 프로퍼티스 파일을 파라미터로 카프카를 실행함. 
  $ bin/kafka-server-start.sh config/server.properties # 포그라운드에서 실행
  $ bin/kafka-server-start.sh -daemon config/server.properties # 백그라운드에서 실행 
  
  # 로그에 실행완료 확인됨(포그라운드에서만 확인 가능)
  # INFO [KafkaRaftServer nodeId=1] Kafka Server started (kafka.server.KafkaRaftServer)
 ```

### 3. 종료
 ```
 # 카프카 종료(포그라운드)
 $ 리눅스 명령어 -> 컨트롤C 실행
 
 # 카프카 종료하기(백그라운드)
 $ bin/kafka-server-stop.sh

 # 잘 종료됐나 확인
 $ sudo lsof -i:9092
 ```

### 4. 백그라운드 로그
 ```
 tail -f logs/kafkaServer.out
 ```
