---
title: 02 Deployment/Service
date: 2025-12-10 10:00:00 +09:00
categories: [infra, kubernetesInflearn]
tags: [ infra, kubernetesInflearn ]
---

## 1. Deployment
### 1. Deployment
 - 파드를 묶음으로 쉽게 관리할 수 있는 기능
 - 디플로이먼트(Deployment) 라는 걸 활용해서 파드(Pod)를 자동으로 배포

### 2. 디플로이먼트(Deployment)의 장점
 - 파드의 수를 지정하는 대로 여러 개의 파드를 쉽게 생성할 수 있음.
   - ex) 파드를 100개를 생성하라고 시키면 디플로이먼트가 알아서 파드를 100개 생성함
 - 파드가 비정상적으로 종료된 경우, 알아서 새로 파드를 생성해 파드 수를 유지함
 - 동일한 구성의 여러 파드를 일괄적으로 일시 중지, 삭제, 업데이트를 하기가 쉬움
   - ex) 디플로이먼트를 활용하면 ‘100개의 파드로 띄워져있는 결제 서버ʼ를 한 번에 일시 중지/삭제/업데이트하는 게 굉장히 쉬움.

### 3. 디플로이먼트 구조
 - 디플로이먼트(Deployment)가 레플리카셋(ReplicaSet)을 관리하고,
 - 레플리카셋(ReplicaSet)이 여러 파드(Pod)를 관리하는 구조

## 2. Deployment 사용
### 1. 메니페스트 ( 디플로이먼트 ) - 스프링부트
 ```
  apiVersion: apps/v1
  kind: Deployment
  
  # Deployment 기본 정보
  metadata:
    name: spring-deployment # Deployment 이름
    
  # Deployment 세부 정보
  spec:
    replicas: 3 # 생성할 파드의 복제본 개수
    selector:
      matchLabels:
        app: backend-app # 아래에서 정의한 Pod 중 'app: backend-app'이라는 값을 가진 파드를 선택
        
    # 배포할 Pod 정의
    template:
      metadata:
        labels: # 레이블 (= 카테고리)
          app: backend-app 
      spec:
        containers:
          - name: spring-container # 컨테이너 이름
            image: spring-server # 컨테이너를 생성할 때 사용할 이미지
            imagePullPolicy: IfNotPresent # 로컬에서 이미지를 먼저 가져온다. 없으면 레지스트리에서 가져온다. 
            ports:
              - containerPort: 8080  # 컨테이너에서 사용하는 포트를 명시적으로 표현
 ```

### 2. 실행
  ```
  # 동일하게 apply로 실행하고 yaml파일체크
  # kubectl apply -f [yaml명]
  kubectl apply -f spring-deployment.yaml
  
  # 만약에 서버를 확장 / 조절 / 이미지 변경 등 기존 파드를 죽이지 안고 알아서 유지해줌.
  # yaml파일을 다시 수정하고 동일하게 실행하면 디플로이먼트가 알아서 수와 설정을 변경해줌 
  kubectl apply -f spring-deployment.yaml
  ```

### 3. 디플로이먼트 확인
  ```
  kubectl get deployment
  ```

### 4. 레플리카셋 확인
  ```
  # 네플리카셋 확인
  # 쿠버네티스 - 네플리카셋+네플리카셋 - 네플리카셋(파드+파드+파드) - 파드(컨테이너+컨테이너+컨테이너)
  kubectl get replicaset
  ```

## 3. Service
### 1. Service
 - 외부로부터 요청을 받는 역할
 - 외부로부터 들어오는 트래픽을 받아, 파드에 균등하게 분배해주는 로드밸런서 역할을 하는 기능
 - 실제 서비스에서 파드(Pod)에 요청을 보낼 때, 포트 포워딩(port-forward)이나 파드 내로 직접 접근(kubectl exec …)해서 요청을 보내진 않음
 - 서비스(Service)를 통해 요청을 보내는 게 일반적

## 4. Service 실행

### 1. 전체 흐름
- ![내 그림](assets/img/infra/kubernetes_inflean/서비스디플로이먼트 이미지.png "이미지")

### 2. 매니페스트 ( yaml )
 ```
  apiVersion: v1
  kind: Service
  
  # Service 기본 정보
  metadata:
    name: spring-service # Service 이름
  
  # Service 세부 정보
  spec:
    type: NodePort # Service의 종류
    selector:
     # 실행되고 있는 파드 중 'app: backend-app'이라는 값을 가진 파드와 서비스를 연결
     # 디플로이먼트로 실행한 앱을 지칭
     app: backend-app 
    ports:
      - protocol: TCP # 서비스에 접속하기 위한 프로토콜
        port: 8080 # 쿠버네티스 내부에서 Service에 접속하기 위한 포트 번호 (컴퓨터 - 서비스 최초접근)
        targetPort: 8080 # 매핑하기 위한 파드의 포트 번호 (서비스 - 파드의 포트번호)
        nodePort: 30000 # 외부에서 사용자들이 접근하게 될 포트 번호 (컴퓨터로 최초접근)
 ```
   - NodePort : 쿠버네티스 내부에서 해당 서비스에 접속하기 위한 포트를 열고 외부에서 접속 가능
   - ClusterIP : 쿠버네티스 내부에서만 통신할 수 있는 IP 주소를 부여. 외부에서는 요청할 수 없음
   - LoadBalancer : 외부의 로드밸런서(AWS의 로드밸런서 등)를 활용해 외부에서 접속할 수 있도록 연결함.

### 3. 실행
 ```
  # 기존과 동일하게 aplly
  kubectl apply -f spring-service.yaml 
 ```

### 4. 확인
 ```
   kubectl get service
 ```

