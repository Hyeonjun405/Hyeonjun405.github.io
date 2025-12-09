---
title: 01 kubernetes
date: 2025-12-09 10:00:00 +09:00
categories: [infra, kubernetesInflearn]
tags: [ infra, kubernetesInflearn ]
---

## 1. kubernetes
### 1. kubernetes
 - 다수의 컨테이너를 효율적으로 배포, 확장 및 관리 하기 위한 오픈 소스 시스템
 - 쿠버네티스(Kubernetes)는 Docker Compose와 비슷함.
 - 쿠버네티스(Kubernetes)의 대략적인 이미지를 그릴 때는 Docker Compose의 확장판으로 보면 편함.

### 2. 쿠버네티스의 장점
 - 컨테이너 관리 자동화 (배포, 확장, 업데이트)
 - 부하 분산 (로드 밸런싱)
 - 쉬운 스케일링
 - 셀프 힐링


## 2. POD
### 1. pod
 - 도커에서는 하나의 프로그램을 실행시키는 단위를 컨테이너
 - 쿠버네티스에서는 하나의 프로그램을 실행시키는 단위를 파드(Pod)
 - 파드를 띄우는 방법 `CLI를 활용하는 방법`, `yaml 파일을 활용하는 방법`

### 2. 파드를 띄우기 예시
#### 1. yaml 만들기
 - 파일명 : nginx_pod.yaml
   - 변경 O, (도커파일은 변경 불가능함)
   - 매니페스트파일(Manifest file) => 다양한 리소스(파드, 서비스, 볼륨 등)를 생성하고 관리하기 위해 사용하는 파일
  - 내용
  ```
  apiVersion: v1 # Pod를 생성할 때는 v1이라고 기재한다. (공식 문서)
  kind: Pod # Pod를 생성한다고 명시
  metadata:
    name: nginx-pod # Pod에 이름 붙이는 기능
  spec:
    containers:
      - name: nginx-container # 생성할 컨테이너의 이름
        image: nginx # 컨테이너를 생성할 때 사용할 Docker 이미지
        ports:
          - containerPort: 80 # 해당 컨테이너가 어떤 포트를 사용하는 지 명시적으로 표현
  ```

#### 2. 실행 
  ```
   kubectl apply -f nginx_pod.yaml
   # 쿠버네티스컨트롤 # 시도 #-f 파일 #nginx_pod.yaml을 실행
  ```

#### 3. 확인
 ```
  # 파드의 존재 확인
  kubectl get pods
  
  # nginx-pod 파드의 세부 정보 조회
  # kubectl describe pods [파드명]  
  kubectl describe pods nginx-pod 
  
  # kubectl logs [파드명]
  kubectl logs nginx-pod # 파드 로그 확인하기
 ```

## 3. POD의 네트워크
### 1. POD 네트워크 
 - 도커는 컨테이너 내부와 컨테이너 외부의 네트워크가 서로 독립적으로 분리되어 있음
 - 쿠버네티스에서는 파드(Pod) 내부의 네트워크를 컨테이너가 공유해서 같이 사용함.
 - 파드(Pod)의 네트워크는 로컬 컴퓨터의 네트워크와는 독립적으로 분리됨
 - 파드(Pod)로 띄운 Nginx에 아무리 요청을 보내도 응답이 없던 것 

### 2. POD의 접근방법
#### 1. 파드(Pod) 내부로 들어가서 접근하기
  ```
   # kubectl exec -it [파드명] -- bash
   # 도커에서 컨테이너로 접속하는 명령어(docker exec -it [컨테이너 ID bash)와 비슷하다.
   kubectl exec -it nginx-pod -- bash # nginx-pod 내부 환경으로 접속
  
   # ---Pod 내부---
   curl localhost:80 # Nginx로 요청보내기
  ```

#### 2. 파드(Pod)의 내부 네트워크를 외부에서도 접속할 수 있도록 포트 포워딩(= 포트 연결시키기) 활용하기
  ```
   # 파드외부에서 실행, 즉 HostOS에서 실행함.
   # kubectl port-forward pod/[파드명] [로컬에서의 포트]/[파드에서의 포트]
   kubectl port-forward pod/nginx-pod 80:80
  ```

## 4. yaml & 빌드
### 1. SpringBoot
#### 1. Dockerfile (기존 Docker 설정과 동일)
  ```
   FROM eclipse-temurin:17-jdk
  
   COPY build/libs/*SNAPSHOT.jar app.jar
   
   ENTRYPOINT ["java", "-jar", "/app.jar"]
  ```

#### 2. 중간과정(Docker와 동일함.)
 - Spring Boot 프로젝트 빌드하기 `./gradlew clean build`
 - Dockerfile을 바탕으로 이미지 빌드하기 ` docker build -t spring-server .`
 - 도커 이미지 확인 `docker image ls`

#### 3.메니페스트 작성
  ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: spring-pod
   spec:
     containers:
       - name: spring-container
         image: spring-server
         ports:
          - containerPort: 8080
         imagePullPolicy: Always
  ```
  - 이미지 pull 정책
    - Always(Default값) : 로컬에서 이미지를 가져오지 않고, 무조건 레지스트리(= Dockerhub, ECR과 같은 원격 이미지 저장소)에서 가져옴
    - IfNotPresent : 만약 로컬에 이미지가 없는 경우에만 레지스트리에서 가져옴
    - Never : 로컬에서 이미지를 먼저 가져옴
    

#### 4. 실행은 동일
 - `kubectl apply -f spring-pod.yaml`


### 2. 다중 Container 
#### 1. 메니페스트파일에서 반복 작성
  ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: spring-pod-1
   spec:
     containers:
       - name: spring-container
         image: spring-server
         imagePullPolicy: IfNotPresent       ports:
           - containerPort: 8080
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: spring-pod-2
   spec:
     containers:
       - name: spring-container
         image: spring-server
         imagePullPolicy: IfNotPresent       ports:
           - containerPort: 8080
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: spring-pod-3
   spec:
     containers:
       - name: spring-container
         image: spring-server
         imagePullPolicy: IfNotPresent
         ports:
   - containerPort: 8080
  ```
