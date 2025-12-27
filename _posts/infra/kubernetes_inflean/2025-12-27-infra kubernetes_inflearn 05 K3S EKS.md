---
title: 05 K3S EKS
date: 2025-12-27 10:00:00 +09:00
categories: [infra, kubernetesInflearn]
tags: [ infra, kubernetesInflearn ]
---

## 1. K8S, K3S
### 1. K8S(Kubernetes)
 - CNCF가 관리하는 표준 쿠버네티스 프로젝트
 - 여러 컴포넌트가 분리되어 있고, 대규모·프로덕션 환경을 전제로 설계

### 2. K3S
 - Rancher(SUSE)가 만든 경량 쿠버네티스 배포판
 - K8S를 그대로 쓰되, 불필요한 요소를 제거하고 단일 바이너리로 묶음

## 2. 생성
### 1. EC 생성
  - 특이사항 X 기본설치

### 2. Docker 설치
  ```
    sudo apt-get update && \
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
    sudo apt-key fingerprint 0EBFCD88 && \
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
    sudo apt-get update && \
    sudo apt-get install -y docker-ce && \
    sudo usermod -aG docker ubuntu && \
    newgrp docker && \
    sudo curl -L "https://github.com/docker/compose/releases/download/2.27.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
    sudo chmod +x /usr/local/bin/docker-compose && \
    sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
  ```
  ```
    # Docker 버전 확인
    docker -v
    
    # Docker Compose 버전 확인
    docker compose version 
  ```

### 3. K3S 설치
  ```
    curl -sfL https://get.k3s.io | sh - # k3s 설치
    sudo chmod 644 /etc/rancher/k3s/k3s.yaml # 권한 부여
  ```
  ```
    # 설치확인
    sudo kubectl version
  ```

### 4. 그외 설정 동일함.
 - 2025-12-26-infra Docker_inflearn 06EC2 Docker 동일

## 3. EKS(Amazon Elastic Kubernetes Service)
### 1. EKS
 - AWS에서 제공하는 쿠버네티스 관리형 서비스
 - DB를 별도 설치, 셋팅이 필요없이 자동으로해주는 RDS랑 느낌이 비슷함.

### 2. 대신해주는 것
  
  | 영역         | EKS가 대신 해주는 것                                |
  | ---------- | -------------------------------------------- |
  | 쿠버네티스 설치   | 마스터 노드 설치 및 초기 구성                            |
  | 컨트롤 플레인 운영 | API Server, Scheduler, Controller Manager 실행 |
  | etcd 관리    | etcd 설치, 운영, 장애 대응                           |
  | 고가용성       | 컨트롤 플레인 멀티 AZ 구성                             |
  | 보안 패치      | 컨트롤 플레인 보안 업데이트                              |
  | 업그레이드      | 쿠버네티스 버전 업그레이드 지원                            |
  | 인증 연동      | IAM ↔ Kubernetes 인증 연동                       |
  | API 가용성    | API Server SLA 제공                            |
  | 장애 대응      | 마스터 장애 자동 복구                                 |
  | 내부 통신 보안   | 컨트롤 플레인 통신 암호화                               |


### 3. 아키텍처
  - ![내 그림](assets/img/infra/kubernetes_inflean/8EKS 아키텍처.png "이미지")
  - `쿠버네티스 클러스터` : 하나의 **마스터 노드**와 여러 **워커 노드**들을 한 묶음으로 부르는 단위
  - `마스터 노드` : 쿠버네티스 클러스터 전체를 관리하는 서버
  - `워커 노드` : 쿠버네티스의 파드를 실행시키는 서버

## 4. EKS 클러스터 생성
### 1. EKS 진입 및 기본 설정
  - ![내 그림](assets/img/infra/kubernetes_inflean/8EKS생성.png "이미지") 
  - ![내 그림](assets/img/infra/kubernetes_inflean/8EKS생성2.png "이미지")

### 2. 클러스터구성
  - ![내 그림](assets/img/infra/kubernetes_inflean/8EKS생성3.png "이미지")
  - ![내 그림](assets/img/infra/kubernetes_inflean/8EKS생성4.png "이미지")
  - ![내 그림](assets/img/infra/kubernetes_inflean/8EKS생성5.png "이미지")
 
## 5. 워커 노드 추가
### 1. 노드 그룹 생성
  - ![내 그림](assets/img/infra/kubernetes_inflean/8워커노드추가.png "이미지")
  - ![내 그림](assets/img/infra/kubernetes_inflean/8워커노드추가2.png "이미지")
  - ![내 그림](assets/img/infra/kubernetes_inflean/8워커노드추가3.png "이미지")
  - ![내 그림](assets/img/infra/kubernetes_inflean/8워커노드추가4.png "이미지")
  - ![내 그림](assets/img/infra/kubernetes_inflean/8워커노드추가5.png "이미지")

### 2. 확인
  - ![내 그림](assets/img/infra/kubernetes_inflean/8워커노드추가6.png "이미지")
  - ![내 그림](assets/img/infra/kubernetes_inflean/8워커노드추가7.png "이미지")
    - 신규 인스턴스가 생성됨.

## 6. 클러스터 명령어 조작
### 1. 로컬에서 EKS클러스터추가
 ```
  # aws eks --rgeion ap-northeast-2 update-kubeconfig --name <EKS 클러스터 이름>
  $ aws eks --region ap-northeast-2 update-kubeconfig --name kube-practice
 ```

### 2. 클러스터 조회
 ```
  # 지금 등록된 클러스터 조회
  # 맨앞에 * 붙어있는 대상이 지금 연결 대상
  # kubectl 을 사용하면 *이 붙어있는 대상에 명령어가 보내짐.
  $ kubectl config get-contexts
 ```

### 3. 관리
 ```
  # 다른 클러스터로 전환
  $ kubectl config use-context <컨텍스트 이름>

  # 특정 컨텍스트 삭제
  $ kubectl config unset contexts.<컨텍스트 이름>
 ```

### 4. 운영
 - 클러스터가 변경되면 해당 메니페스트파일들을 로컬에 두고, 필요한 작업들을 바로 진행할 수 있음.
 - 서비스에서 로드밸런서
   ```
    apiVersion: v1
    kind: Service
    
    # Service 기본 정보
    metadata:
    name: spring-service
    
    # Service 세부 정보
    spec:
      type: LoadBalancer # 외부 로드밸런서 사용함. 요거 사용 가능해짐
    selector:
      app: backend-app 
    ports:
      - protocol: TCP 
        port: 80 # 외부에서 사용자가 요청을 보낼 때 사용하는 포트 번호
        targetPort: 8080 # 매핑하기 위한 파드의 포트 번호
   ```



