---
title: 04 Volume
date: 2025-12-14 10:00:00 +09:00
categories: [infra, kubernetesInflearn]
tags: [ infra, kubernetesInflearn ]
---

## 1. Volume
### 1. Volume
#### 1. Volume(Dokcer와 동일)
 - 쿠버네티스는 기존 파드에서 변경된 부분을 수정하지 않고, 새로운 파드를 만들어서 통째로 갈아끼우는 방식으로 교체
 - 데이터를 영속적으로 저장하기 위한 방법
 - 데이터 Volume 방식
   - 로컬 볼륨 : 파드 내부의 공간 일부를 볼륨(Volume)으로 활용하는 방식 / POD초기화시 사라짐
   - 퍼시스턴트 볼륨(Persistent Volume, PV) : 파드 외부의 공간 일부를 볼륨(Volume)으로 활용하는 방식 / LocalHost에 Volume공간을 둠

#### 2. 퍼시스턴트 볼륨 클레임
 - 쿠버네티스 환경 내에서 일어나는 일, 호스트와는 다름!
 - 파드(Pod)가 퍼시스턴트 볼륨(PV)에 직접 연결할 수 없음
 - 퍼시턴트 볼륨 클레임(PVC)이라는 중개자가 있어야 함.
 - 파드 - 퍼시스턴트 볼륨 클레임(중개자) - 퍼시스턴트 볼륨
 - ![내 그림](assets/img/infra/kubernetes_inflean/쿠버네티스 - Volume, 퍼시스턴트 볼륨 클레임.png "이미지")

## 2. Volume + mysql 기본 셋팅
### 1. mysql-deployment
  ```
  apiVersion: apps/v1
  kind: Deployment
  
  # Deployment 기본 정보
  metadata:
    name: mysql-deployment # Deployment 이름
  
  # Deployment 세부 정보
  spec:
    replicas: 1 # 생성할 파드의 복제본 개수
    selector:
      matchLabels:
          app: mysql-db # 아래에서 정의한 Pod 중 'app: mysql-db'이라는 값을 가진 파드를 선택
          
    # 배포할 Pod 정의
    template:
      metadata:
        labels: # 레이블 (= 카테고리)
          app: mysql-db
      spec:
        containers:
          - name: mysql-container # 컨테이너 이름
            image: mysql # 컨테이너를 생성할 때 사용할 이미지
            ports:
              - containerPort: 3306  # 컨테이너에서 사용하는 포트를 명시적으로 표현
            env:
              - name: MYSQL_ROOT_PASSWORD
                valueFrom:
                  secretKeyRef: # 시크릿
                    name: mysql-secret # Secret 이름
                    key: mysql-root-password
  
              - name: MYSQL_DATABASE
                valueFrom:
                  configMapKeyRef: # 일반 컨피그
                    name: mysql-config # ConfigMap 이름
                    key: mysql-database
  ```

### 2. mysql-config
  ```
  apiVersion: v1
  kind: ConfigMap
  
  # ConfigMap 기본 정보
  metadata:
    name: mysql-config # ConfigMap 이름
  
  # Key, Value 형식으로 설정값 저장
  # 초기 데이터 생성 - kub-practice DB를 만들기 설정
  data:
    mysql-database: kub-practice
  ```

### 3. mysql-secret.yaml
  ```
  apiVersion: v1
  kind: Secret # Secret 기본 정보
  
  metadata:
    name: mysql-secret # Secret 이름
  
  # Key, Value 형식으로 값 저장
  stringData:
    mysql-root-password: password123
  ```

### 4. mysql-service
  ```
  apiVersion: v1
  kind: Service
  
  # Service 기본 정보
  metadata:
    name: mysql-service # Service 이름
  
  # Service 세부 정보
  spec:
    type: NodePort # Service의 종류
    selector:
      app: mysql-db # 실행되고 있는 파드 중 'app: mysql-db'이라는 값을 가진 파드와 서비스를 연결
    ports:
      - protocol: TCP # 서비스에 접속하기 위한 프로토콜
        port: 3306 # 쿠버네티스 내부에서 Service에 접속하기 위한 포트 번호
        targetPort: 3306 # 매핑하기 위한 파드의 포트 번호
        nodePort: 30002 # 외부에서 사용자들이 접근하게 될 포트 번호
  ```

### 5. 매니페스트 오브젝트 만들기
  ```
  # 시크릿이랑 컨피그부터 해야 디플로이할때 환경변수값이 들어감.
  kubectl apply -f mysql-secret.yaml # 시크릿
  kubectl apply -f mysql-config.yaml # 컨피그
  kubectl apply -f mysql-deployment.yaml # 디플로이
  kubectl apply -f mysql-service.yaml #서비스
  ```

## 3. Volume + mysql + PVC 
### 1. 퍼시스턴트 볼륨(PV)
  ```
  apiVersion: v1
  kind: PersistentVolume
  
  # PersistentVolume 기본 정보
  metadata:
    name: mysql-pv # PersistentVolume 이름
  
  # PersistentVolume 세부 정보
  spec:
    storageClassName: my-storage # PV와 PVC의 storageClassName이 같다면 볼륨이 연결된다.
    capacity:
      storage: 1Gi # 볼륨이 사용할 용량을 설정
    accessModes:
     - ReadWriteOnce # 아래 hostPath 타입 활용 시 이 옵션만 사용 가능
    hostPath: # hostPath 타입을 활용 (hostPath : 쿠버네티스 내부 공간을 활용)
      path: "/mnt/data" # 쿠버네티스 내부의 공간에서 /mnt/data의 경로를 볼륨으로 사용
  ```
### 2. 퍼시스턴트 볼륨 클레임(PVC)
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  
  # PersistentVolumeClaim 기본 정보
  metadata:
    name: mysql-pvc # PersistentVolumeClaim 이름
  
  # PersistentVolumeClaim 세부 정보
  spec:
    storageClassName: my-storage # PV와 PVC의 storageClassName이 같다면 볼륨이 연결된다.
    accessModes:
     - ReadWriteOnce # 볼륨에 접근할 때의 권한
    resources: # PVC가 PV에 요청하는 리소스의 양을 정의
      requests: # 필요한 최소 리소스
        storage: 1Gi # PVC가 PV에 요청하는 스토리지 양 (PV가 최소 1Gi 이상은 되어야 한다.)
  ```
### 3. deployment 변경사항
  ```
  
    # 배포할 Pod 정의
  template:
    metadata:
      labels: 
        app: mysql-db
    spec:
      containers:
        - name: mysql-container 
          image: mysql
          ports:
            - containerPort: 3306 
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: mysql-root-password
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: mysql-config
                  key: mysql-database
                  
          # 이 부분부터 추가됨 - env와 동일한 라인
          
          # 컨테이너 내에서 어떤 경로를 볼륨으로 사용할 지 지정
          volumeMounts:
          - name: mysql-persistent-storage # 밑에서 설정할 volumes.name과 값이 같아야 함
            mountPath: /var/lib/mysql  # mysql 컨테이너 내부에 있는 경로
            
      # containers 와 동일한 라인 
      # 파드가 사용할 볼륨을 지정
      volumes:
      - name: mysql-persistent-storage # 위에서 설정할 volumeMounts.name과 일치해야 함
        persistentVolumeClaim:
          claimName: mysql-pvc # 연결시킬 PVC의 name과 동일해야 함  
  ```

### 4. 오브젝트 생성
  ```
  # 다른 오브젝트 생성과 동일함.
  cubectl apply -f mysql-pv.yaml 
  cubectl apply -f mysql-pvc.yaml  
  ```

## 4. 쿠버네티스에서 스프링부트 + MYSQL
### 1. Spring-deployment.yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  
  # Deployment 기본 정보
  metadata:
    name: spring-deployment # Deployment 이름# Deployment 세부 정보
    
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
            env:
              - name: DB_HOST
                value: mysql-service # Service의 name만 입력하면 다른 서비스와 통신할 수 있다. 
              - name: DB_PORT
                value: "3306" # 숫자값을 문자로 인식하게 만들기 위해 쌍따옴표 붙여야 한다.
              - name: DB_NAME
                value: kub-practice
              - name: DB_USERNAME
                value: root
              - name: DB_PASSWORD
                value: password123
  ```

### 2. Note
#### 1. Note
 - 스프링부트와 MYSQL은 같은 파드에서 띄우지 않는다.
   - pod가 확장될 경우 부트와 MYSQL이 같이 확장되는 현상이 나타나기 때문에, 별도의 파드로 분리하여 관리해야한다.
   - 또한 MYSQL이 리스타트가 필요한경우, 스프링부트가 버전업이 된 경우에도 파드자체를 다시 실행해야하기 때문에 같이 사용 X
 - 정리
   - 쿠버네티스의 디플로이먼트로 레플리카셋을 만들어서 안에 있는 파드를 관리하고
   - 파드는 AP이미지를 담은 컨테이너를 만들어서 관리하는데, 필요에 따라서 사이드카 컨테이너를 만들어서 쓰기도하고,
   - 컨테이너 확장이 필요한경우 디플로이먼트로 네플리카셋 안에 파드를 여러개 복제해서 관리하는 형태,
   - 새로운 AP이미지는 별도 디플로이를 만들어서 쿠버네티스안에서 소통하게 만듬.

#### 2. 사이드카패턴은
 - 사이드카패턴은 밀접하게 협력하지만, 독립 서비스는 아닌 경우에 사용
 - 사이드카 패턴
   ```
   Pod
   ├─ Main Container (주 역할)
   └─ Sidecar Container (보조 역할)
   ```
 - 예시
 
   | 목적     | 메인 컨테이너            | 사이드카 컨테이너             | 사이드카 역할          | 왜 같은 파드인가          |
   | ------ | ------------------ | --------------------- | ---------------- | ------------------ |
   | 로그 수집  | Spring Boot / Node | fluent-bit / filebeat | 로그 파일 수집 후 외부 전송 | 동일 볼륨을 바로 읽어야 함    |
   | 서비스 메시 | 애플리케이션             | Envoy                 | 트래픽 제어, TLS, 인증  | localhost 트래픽 가로채기 |
   | 모니터링   | 애플리케이션             | Prometheus exporter   | 메트릭 수집           | 프로세스 수준 접근         |
   | 설정 동기화 | 애플리케이션             | config-sync           | 설정 서버에서 파일 동기화   | 파일 공유 필수           |
   | 보안     | 애플리케이션             | Vault Agent           | 인증서·토큰 자동 갱신     | 민감 정보 로컬 주입        |
   | 프록시    | 애플리케이션             | nginx / envoy         | 요청 중계·필터링        | 네트워크 밀접 결합         |
   | 데이터 변환 | 애플리케이션             | transformer           | 입력/출력 데이터 전처리    | 실시간 처리 필요          |

