---
title: 03 ConfigMap/Secret
date: 2025-12-12 10:00:00 +09:00
categories: [infra, kubernetesInflearn]
tags: [ infra, kubernetesInflearn ]
---

## 1. 환경변수
### 1. 환경변수
 - 환경변수로 등록한 값을 설정하는 것
 - 스프링부트에서 @Value로 값을 획득하는 것들 
   ```
    @Value("${MY_PASSWORD:default}")
    private String myPassword;
   ```

### 2. deployment yaml
 ```
  ~~~~~~
   template:
     metadata:
       labels:
        app: backend-app
     spec:
       containers:
         - name: spring-container
           image: spring-server
           imagePullPolicy: IfNotPresent 
           ports:
            - containerPort: 8080
           env: # 환경변수 등록 
            - name: MY_ACCOUNT
              value: ACCOOUNT value
            - name: MY_PASSWORD
              value: pwd1234
 ```

## 2. ConfingMap
### 1. ConfingMap
 - Spring Boot에서는 설정값을 application.yml / Nest.js에서도 설정값을 .env 으로 관리 
 - 쿠버네티스에서 변수를 관리하는 역할을 가진 오브젝트를 컨피그맵(ConfigMap)이라고 함.
 - 기존 Deployment로 환경변수를 설정하게 되면 local/dev/sit/prd등 환경에서 고정값을 사용하게됨.
 - 따라서 별도 파일에서 환경에 맞게 분리 사용함.

### 2.사용
#### 1.configMap 파일 생성
 - 파일명 고정값 X
 ```
   apiVersion: v1
   kind: ConfigMap
  
   # ConfigMap 기본 정보
   metadata:
     name: spring-config # ConfigMap 이름
  
     # Key, Value 형식으로 설정값 저장
   data:
     my-account: accountValue
     my-password: password123
 ```

#### 2. 도커파일 변동사항
 ```
 ~~~~
 # 기존 환경변수 입력한 곳
  env:
   - name: MY_ACCOUNT
     valueFrom:
       configMapKeyRef:
         name: spring-config # ConfigMap의 이름 / 생성한 configMap에서 metadata -> name 설정한 값
         key: my-account # ConfigMap에 설정되어 있는 Key값
    - name: MY_PASSWORD
      valueFrom:
        configMapKeyRef:
          name: spring-config
          key: my-password
 ```

#### 3. config 등록 
 ```
  kubectl apply -f spring-config.yaml
 ```

## 3. Secret
### 1. Secret
 - 시크릿(Secret)은 컨피그맵(ConfigMap)과 비슷하게 환경 변수를 분리해서 관리하는 오브젝트
 - 시크릿(Secret)은 비밀번호와 같이 보안적으로 중요한 값을 관리하기 위한 오브젝트

### 2. 사용
#### 1. sercret 파일 생성
 - 파일명 고정값 X
  ```
  apiVersion: v1
  kind: Secret # Secret 기본 정보
  
  metadata:
    name: spring-secret # Secret 이름
  
    # Key, Value 형식으로 값 저장
    stringData:
      my-password: my-secret-password
  ```

#### 2. 도커파일 변동사항
 ```
 ~~~~
 # 기존 환경변수 입력한 곳
  env:
   - name: MY_ACCOUNT
     valueFrom:
       configMapKeyRef: # 기존 config 방식
         name: spring-config 
         key: my-account 
    - name: MY_PASSWORD
      valueFrom:
       secretKeyRef:  # sercret 방식
          name: spring-secret
          key: my-password
 ```

#### 3. secret 등록
 ```
   kubectl apply -f spring-secret.yam
 ```
 
