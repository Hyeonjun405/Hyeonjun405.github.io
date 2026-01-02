---
title: 01 Linux/window 명령어
date: 2026-01-02 10:00:00 +09:00
categories: [os, osNote]
tags: [ note, osNote ]
---

## 1. 구분 
### 1. CMD
  - 오래된 윈도우 기본 쉘 / 배치(.bat) 실행용
  - 문법이 단순하지만 기능이 제한적
  - 현대적인 자동화, 구조적 처리에 취약
  - 레거시 스크립트 유지용으로 주로 남아 있음

### 2. PowerShell
 - 마이크로소프트가 cmd 한계를 해결하려고 만든 현대적 쉘
 - 현재 윈도우 표준 쉘
 - .NET 기반
 - 강력한 필터링, 정렬, 속성 접근 가능
 - 윈도우 관리 자동화에 최적화
 - cmd 명령어 상당수와 호환됨
 - 일부 명령어는 alias로 설정 되어있음 ex) ls가 bash 명령이 아니라 Get-ChildItem의 별칭
 
### 3. bash
 - 리눅스/유닉스 계열 표준 쉘
 - 서버, 컨테이너, CI/CD의 사실상 기준
 - 조합 가능한 작은 명령어 철학
 - awk, sed, grep 같은 텍스트 처리 도구와 궁합이 좋음
 - 서버 운영, 배포, 자동화에서 압도적 사용량


## 2. 파일 디렉토리 관리
### 1. 현재위치 / 디렉토리 이동
  
  | 기능       | Linux (bash) | Windows (cmd)    | Windows (PowerShell) |
  | -------- | ------------ | ---------------- | -------------------- |
  | 현재 위치 확인 | pwd          | cd               | Get-Location         |
  | 디렉터리 이동  | cd           | cd               | Set-Location         |
  | 홈으로 이동   | cd ~         | cd %USERPROFILE% | cd ~                 |


### 2. 파일/디렉토리 목록 조회

  | 기능     | Linux (bash) | Windows (cmd) | Windows (PowerShell) |
  | ------ | ------------ | ------------- | -------------------- |
  | 목록 조회  | ls           | dir           | Get-ChildItem        |
  | 숨김 포함  | ls -a        | dir /a        | Get-ChildItem -Force |
  | 자세히 보기 | ls -l        | dir           | Get-ChildItem        |

### 3. 디렉토리 생성 / 삭제

  | 기능       | Linux (bash) | Windows (cmd) | Windows (PowerShell)         |
  | -------- | ------------ | ------------- | ---------------------------- |
  | 디렉터리 생성  | mkdir        | mkdir         | New-Item -ItemType Directory |
  | 여러 단계 생성 | mkdir -p     | mkdir         | mkdir -p                     |
  | 디렉터리 삭제  | rmdir        | rmdir         | Remove-Item                  |
  | 강제 삭제    | rm -rf       | rmdir /s /q   | Remove-Item -Recurse -Force  |

### 4. 파일 복사 / 이동 / 이름 변경

  | 기능      | Linux (bash) | Windows (cmd) | Windows (PowerShell) |
  | ------- | ------------ | ------------- | -------------------- |
  | 파일 복사   | cp a b       | copy a b      | Copy-Item a b        |
  | 디렉터리 복사 | cp -r        | xcopy /e      | Copy-Item -Recurse   |
  | 파일 이동   | mv           | move          | Move-Item            |
  | 이름 변경   | mv old new   | ren old new   | Rename-Item          |

## 3. 파일 조회
### 1. 파일 내용 보기
  
  | 기능    | Linux (bash) | Windows (cmd) | Windows (PowerShell)         |
  | ----- | ------------ | ------------- |------------------------------|
  | 파일 출력 | cat file.txt | type file.txt | Get-Content file.txt         |
  | 여러 파일 | cat a b      | type a b      | Get-Content a,b              |
  | 페이지 보기 | less file.txt | more file.txt | Get-Content file.txt \| more |
  | 앞부분    | head file.txt   | 없음            | Get-Content file.txt -TotalCount 10 |
  | 뒷부분    | tail file.txt   | 없음            | Get-Content file.txt -Tail 10       |
  | 실시간 로그 | tail -f app.log | 없음            | Get-Content app.log -Wait           |

### 2. 파일검색

  | 기능     | Linux (bash)        | Windows (cmd)       | Windows (PowerShell)                 |
  | ------ | ------------------- | ------------------- | ------------------------------------ |
  | 문자열 검색 | grep "err" file.txt | find "err" file.txt | Select-String "err" file.txt         |
  | 재귀 검색  | grep -r "err" .     | findstr /s err *    | Select-String "err" -Path . -Recurse |


## 4. 권한 관련

  | 기능    | Linux (bash) | Windows (cmd)   | Windows (PowerShell) |
  | ----- | ------------ | --------------- | -------------------- |
  | 권한 확인 | ls -l        | icacls file.txt | Get-Acl file.txt     |
  | 권한 변경  | chmod 755 file   | icacls file /grant user:F | Set-Acl              |
  | 소유자 변경 | chown user:group | 없음                        | Set-Acl              |

## 5. 프로세스 관리
### 1. 프로세스 조회

  | 기능    | Linux (bash)       | Windows (cmd)          | Windows (PowerShell) |
  | ----- | ------------------ | ---------------------- | -------------------- |
  | 전체 조회 | ps -ef             | tasklist               | Get-Process          |
  | 실시간   | top                | 없음                     | Get-Process          |
  | 필터링   | ps -ef | grep java | tasklist | find "java" | Get-Process java     |

### 2. 프로세스 삭제

| 기능    | Linux (bash)       | Windows (cmd)          | Windows (PowerShell) |
  | ----- | ----------- | ----------------- | -------------------- |
  | 정상 종료 | kill PID    | taskkill /pid PID | Stop-Process -Id PID |
  | 강제 종료 | kill -9 PID | taskkill /f       | Stop-Process -Force  |

### 3. 프로세스 실행

  | 기능    | Linux (bash)       | Windows (cmd)          | Windows (PowerShell) |
  |-------------|---|---|---|
  | 포그라운드 실행    |  java -jar app.jar | java -jar app.jar | java -jar app.jar |
  | 백그라운드 실행    |  command & | start app.exe | Start-Process app.exe |
  | 중지 후 백그라운드  |  Ctrl+Z → bg | 불가 | 불가 |
  | 작업 상태 확인    |  jobs | 없음 | Get-Job |
  | 포그라운드 복귀    | fg %1 | 불가 | 불가 |
  | 터미널 종료 후 유지 |  nohup command & | 불완전 | Start-Process | 
  | 완전 독립 실행    |  setsid command | 불가 | Start-Process | 
  | 서비스 실행      |  systemd service | Windows Service | Windows Service | 
  | 스크립트 백그라운드  | ./script.sh & | start script.bat | Start-Job { script } | 

## 6. 네트워크관련
### 1. IP 주소

| 기능    | Linux (bash)       | Windows (cmd)          | Windows (PowerShell) |  
  | ----- | -------- | ------------ | ---------------- |
  | IP 확인 | ip addr  | ipconfig     | Get-NetIPAddress |
  | 라우팅   | ip route | route print  | Get-NetRoute     |

### 2. 포트 연결 및 확인

  | 기능    | Linux (bash)       | Windows (cmd)          | Windows (PowerShell) |
  | ------ | -------------- | ------------ | ---------------------------------- |
  | 포트 확인  | netstat -tulpn | netstat -ano | Get-NetTCPConnection               |
  | 리스닝 포트 | ss -lntp       | netstat -an  | Get-NetTCPConnection -State Listen |

### 3. 통신 테스트

  | 기능    | Linux (bash)       | Windows (cmd)          | Windows (PowerShell) |
  | ---- | -------- | -------- | ----------------- |
  | 핑    | ping     | ping     | ping              |
  | HTTP | curl     | curl     | Invoke-WebRequest |
  | DNS  | nslookup | nslookup | Resolve-DnsName   |

## 7. 시스템 관련
### 1. OS/커널

  | 기능    | Linux (bash)       | Windows (cmd)          | Windows (PowerShell) |  
  | ----- | ------------------- | ------------ | ---------------- |
  | OS 정보 | uname -a            | ver          | Get-ComputerInfo |
  | 배포판   | cat /etc/os-release | systeminfo   | Get-ComputerInfo |

### 2. 리소스

  | 기능    | Linux (bash)       | Windows (cmd)          | Windows (PowerShell) |
  | --- | ------- | ------------ | ------------------------------------- |
  | CPU | lscpu   | systeminfo   | Get-CimInstance Win32_Processor       |
  | 메모리 | free -h | systeminfo   | Get-CimInstance Win32_OperatingSystem |
  | 디스크 | df -h   | dir          | Get-PSDrive                           |


## 8. 로그 관리

  | 기능    | Linux (bash)       | Windows (cmd)          | Windows (PowerShell) |
  | ------ | ------------------------ | ------------ | ----------------- |
  | 로그 조회  | tail -f /var/log/app.log | type log.txt | Get-Content -Wait |
  | 시스템 로그 | journalctl               | eventvwr     | Get-EventLog      |
