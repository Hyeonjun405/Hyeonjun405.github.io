---
title: 01 Container infra 전체 흐름
date: 2025-12-31 10:00:00 +09:00
categories: [note, containerNote]
tags: [ note, containerNote ]
---

## 1. 흐름 (빌드 - push - 실행)
### 1. 컨테이너 기반 표준 흐름
 - 베이스 이미지로 Docker/Podman에서 이미지를 빌드
 - 이미지를 레지스트리에 push
 - Kubernetes(OpenShift)가 레지스트리에서 pull해서 실행

### 2. 단일 서버 컨테이너 흐름
 - 베이스 이미지로 Docker/Podman에서 이미지를 빌드
 - 이미지를 레지스트리에 push
 - 서버에서 docker run / podman run 으로 실행

### 3. 컨테이너 미사용(WAS 전통 방식)
 - 서버에 WildFly를 직접 설치
 - WAR/EAR 배포
 - 레지스트리/도커 개념 없음


## 2. 단계별 Point
### 1. 생성 (이미지 빌드)
 - 소스 코드와 설정을 묶어 컨테이너 이미지를 만듬
 - 이 단계에서는 실행이 일어나지 않음
 - 결과물은 항상 “이미지”
 - 종류
  
    | 종류(도구)        | 주요 특징                                 |
    | ------------- | ------------------------------------- |
    | Docker        | 가장 대중적, Dockerfile 표준, 로컬 개발에 최적      |
    | Podman        | 데몬리스 구조, rootless 실행 가능, 보안 친화적       |
    | Buildah       | 이미지 빌드에 특화, CI/CD에서 자주 사용             |
    | OpenShift S2I | 소스 코드 기반 자동 빌드, Dockerfile 없이도 이미지 생성 |

### 2. 저장 (이미지 레지스트리)
 - 빌드된 이미지를 저장하고 버전/접근을 관리
 - 저장만 한다, 실행 기능 없음
 - pull / push가 전부
 - 운영 환경에서는 대부분 private 레지스트리 사용
 - 종류
    
    | 종류(레지스트리)                   | 주요 특징                      |
    | --------------------------- | -------------------------- |
    | Docker Hub                  | 퍼블릭 기본 레지스트리, 설정 없으면 기본 사용 |
    | AWS ECR                     | IAM 기반 인증, EKS/ECS와 강결합    |
    | GCP Artifact Registry       | GKE 연동, 멀티 아티팩트 지원         |
    | Azure ACR                   | AKS 연동, VNet 통합 쉬움         |
    | Harbor                      | 사내 구축, 권한/프로젝트/취약점 스캔 강점   |
    | Nexus                       | Maven/npm/Docker 통합 관리 가능  |
    | OpenShift Internal Registry | 클러스터 내부 전용, OpenShift와 일체화 |

### 3. 실행 (배포 / 런타임)
 - 레지스트리에서 이미지를 가져와 실제로 애플리케이션을 실행
 - 서비스가 뜸 / 장애 복구, 스케일링 책임이 있음
 - Docker/Podman은 “개별 실행”, Kubernetes/OpenShift는 “집단 관리”
 - 종류 

    | 종류(실행 주체)  | 주요 특징                    |
    | ---------- | ------------------------ |
    | docker run | 단일 서버 실행, 가장 단순          |
    | podman run | docker run과 유사, 보안 규칙 엄격 |
    | Kubernetes | 다수 컨테이너 관리, 스케일링/복구 자동화  |
    | OpenShift  | Kubernetes + 보안/운영 표준 내장 |
    | AWS ECS    | AWS 관리형 실행 환경, 쿠버네티스 비의존 |


## 3. 단계별 조합
### 1. 조합 특징
 - 생성 / 저장 / 실행은 완전히 분리된 책임 / 생성 도구 ≠ 저장소 ≠ 실행 주체
 - 생성은 “빌드 도구 문제” / 저장은 “레지스트리 선택 문제” / 실행은 “운영 플랫폼 문제”

### 2. 자주 사용하는 조합
  
  | 조합(생성 → 저장 → 실행)                           | 자주 쓰는 이유                      |
  | ------------------------------------------ | ----------------------------- |
  | Docker → Docker Hub → docker run           | 로컬 개발·테스트, 설정 최소, 진입 장벽 낮음    |
  | Docker → ECR → EKS                         | AWS 표준 조합, IAM 인증, 운영 안정성     |
  | Podman → ECR → EKS                         | 보안 강화(rootless), Docker 호환 유지 |
  | Docker → Harbor → Kubernetes               | 사내망, 이미지 통제·취약점 관리            |
  | Podman → Harbor → OpenShift                | Red Hat 계열 표준, 보안 규정 대응       |
  | S2I → Internal Registry → OpenShift        | 소스→배포 자동화, 개발자 편의             |
  | Docker(GitHub Actions) → GHCR → Kubernetes | CI/CD 연계 쉬움, 깃허브 중심 조직        |


## 4. 오픈시프트(OpenShift)
### 1. 오픈시프트
 - 쿠버네티스를 기업 환경에서 바로 운영 가능하게 만든 Red Hat의 컨테이너 플랫폼
 - 쿠버네티스 “엔진”에 보안, 빌드, 배포, 운영 규칙을 얹은 완성형 플랫폼

### 2. 레이어
 - 쿠버네티스는 “안에 들어 있고”, 오픈시프트는 “그걸 감싼 운영 플랫폼”
 - 구성
   - 애플리케이션 (컨테이너)
   - OpenShift 기능 (보안, 빌드, 배포 자동화)
   - Kubernetes
   - 컨테이너 런타임 (CRI-O)
   - Linux (RHEL 계열)

### 3. 구성
  
  | 계층    | 핵심 역할                        |
  | ----- | ---------------------------- |
  | 엔진    | Kubernetes + CRI-O           |
  | 실행    | Pod / Service / Route        |
  | 빌드·저장 | S2I / Registry / ImageStream |
  | 보안    | SCC / RBAC                   |
  | 운영    | Console / Operator           |

 
### 4. 쿠버네티스와 오픈시프트 
  
  | 구분      | Kubernetes  | OpenShift          |
  | ------- | ----------- | ------------------ |
  | 설치 난이도  | 구성 요소 직접 선택 | 표준 아키텍처 제공         |
  | 보안 기본값  | 느슨함         | 매우 엄격              |
  | 컨테이너 실행 | root 가능     | 기본적으로 root 불가      |
  | 이미지 빌드  | 외부 도구 필요    | S2I/BuildConfig 내장 |
  | 레지스트리   | 외부 구성       | 내부 레지스트리 포함 가능     |
  | 운영 도구   | 조합 필요       | 웹 콘솔/CLI 기본 제공     |
  | 지원      | 커뮤니티 중심     | Red Hat 엔터프라이즈 지원  |

### 5. 체크(쿠버네티스-자바/오픈시프트-스프링)

  | Kubernetes  | Java         | OpenShift | Spring              |
  | ----------- | ------------ | --------- | ------------------- |
  | Pod/Service | JVM/Bytecode | SCC/Route | Security/Dispatcher |
  | yaml 구성     | main 메서드     | 표준 배포 흐름  | Auto Configuration  |
  | 애드온 선택      | 라이브러리 선택     | 내장 기능     | Starter 의존성         |
  | 운영 설계       | JVM 튜닝       | 운영 정책 기본값 | Opinionated 설계      |

