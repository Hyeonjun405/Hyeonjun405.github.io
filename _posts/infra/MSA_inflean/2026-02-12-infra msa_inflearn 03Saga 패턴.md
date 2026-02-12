---
title: 03 Saga 패턴
date: 2026-02-12 10:00:00 +09:00
categories: [infra, MSAInflearn]
tags: [ infra, MSAInflearn ]
---

## 1. MSA에서 @Transactional 패턴
### 1. memo
 - 게시글 작성전 포인트차감 -> 게시글 작성 -> 활동점수 부여 서비스 로직

### 2. 상황
   ![내 그림](assets/img/infra/msa_inflearn/06/트랜잭션 패턴.png "이미지")

### 3. 자바소스
 ```
 @Transactional
 public void create(CreateBoardRequestDto createBoardRequestDto) {
    // <다른 서비스> 게시글 작성 전 100 포인트 차감 
    pointClient.deductPoints(createBoardRequestDto.getUserId(), 100);

    // <작업 서비스> 게시글 작성
    Board board = new Board(
      createBoardRequestDto.getTitle(),
      createBoardRequestDto.getContent(),
      createBoardRequestDto.getUserId()
    );

    this.boardRepository.save(board); // 롤백
  
    // <다른 서비스> 게시글 작성 시 작성자에게 활동 점수 10점 부여 - 다른 서비스
    userClient.addActivityScore(createBoardRequestDto.getUserId(), 10); // 에러 발생
  }
 ```

### 3. memo
 - 위 패턴에서 다른 서비스는 MSA 구조로 구성된 다른 서비스에서 작업을 진행함.
 - 활동 점수를 부여도중 에러가 발생하면,
   - 해당 로직의 작성자에게 점수 부여는 커밋X
   - 게시글작성은 해당 메소드가 예외가 터졌으므로 Transactional에 의해 롤백
   - 작성 전 100포인트 차감은 다른 서비스에서 작업을 하고 완료했기 때문에 커밋이 완료된 상태, Transactional 영향권이 X 
 - Transactional은 비즈니스로직이 수행되고 있는 프로젝트의 DB에만 영향을 주게됨.
 - 모놀리식 아키텍쳐 구조에서는 동일한 서비스, 디비에서 작업하기 때문에 트랜잭션이 문제가 없었으나
 - MSA구조에서는 별도의 방법(2PC, TCC, Saga)를 이용해서 전반적으로 데이터를 맞추는 작업이 들어가야함.

## 2. 별도의 방법
### 1. memo
 - "분산 트랜잭션을 정합성 있게 처리하는 패턴"
 - 주문 → 결제 → 재고 로직에서의 흐름

### 2. 2PC (Two-Phase Commit)
 - 여러 서비스(또는 DB)가 하나의 분산 트랜잭션처럼 동작하도록 2단계 절차로 커밋을 결정하는 방식.
 - 중앙에 Coordinator가 있고 모든 참여자가 “커밋 가능”이라고 응답하면
   - 마지막에 한 번에 전부 커밋
   - 하나라도 실패하면 전부 롤백
 - 흐름
   - 작업 1
     - 주문 서비스: 커밋 가능
     - 결제 서비스: 커밋 가능
     - 재고 서비스: 실패
   - 작업 2
     - 전체 롤백처리
 
### 3. TCC (Try-Confirm-Cancel)
 - 각 서비스가 “임시 처리 → 확정 → 취소” 3단계를 제공하는 분산 트랜잭션 패턴.
 - 먼저 실제 확정이 아니라 “예약” 작업
   - 모든 예약이 성공하면 확정
   - 중간에 실패하면 이미 성공한 예약을 취소
 - 흐름
   - Try 단계
      - 주문: 주문 임시 생성
      - 결제: 금액 홀딩
      - 재고: 수량 홀딩
   - Confirm 단계
      - 전부 성공하면 / 전부 확정 처리
      - 중간에 실패하면 결제 홀딩 취소 / 재고 수량 복구 / 주문 취소

### 4. Saga
 - 각 서비스가 자기 로컬 트랜잭션을 바로 커밋하고, 실패 시 보상 트랜잭션으로 이전 작업을 되돌리는 방식.
 - 보상트랜잭션은 수동으로 트랜잭션의 롤백(Rollback) 효과를 내는 방법
 - 흐름
   - 주문 생성 (커밋)
   - 결제 승인 (커밋)
   - 재고 차감 (실패)
   - 결제 취소
   - 주문 취소

## 3. Saga
### 1. Memo
 - 실질적으로 특별한 기능이 아니라 비슷하게 구현을 해놓은 형태라고 보면됨.
 - 예시에서는 각기 포인트마다 boolean 값을 두고,
   - 로직이 성공하면 true로 변경
   - 실패하면 catch로 빠지게되니 true작업에 대해서만 원복 작업처리
 - 작업을 진행하다보면  Eventual Consistency(최종적 일관성)이 발생함.
   - 처음에는 잠깐 불일치하는 데이터가 발생하지만,
   - 일정 시간 이후에는 모든 데이터의 일관성이 맞아지는 현상을 의미

### 2. 보상 트랜잭션 진행
  ```
  public void create(CreateBoardRequestDto createBoardRequestDto) {
    // 게시글 저장을 성공했는 지 판단하는 플래그
    boolean isBoardCreated = false;
    Long savedBoardId = null;
  
    // 포인트 차감을 성공했는 지 판단하는 플래그
    boolean isPointDeducted = false;
  
    try {
      // 게시글 작성 전 100 포인트 차감
      pointClient.deductPoints(createBoardRequestDto.getUserId(), 100);
      isPointDeducted = true; // 포인트 차감 성공 플래그
      System.out.println("포인트 차감 성공");
  
      // 게시글 작성
      Board board = new Board(
          createBoardRequestDto.getTitle(),
          createBoardRequestDto.getContent(),
          createBoardRequestDto.getUserId()
      );
  
      Board savedBoard = this.boardRepository.save(board);
      savedBoardId = savedBoard.getBoardId();
      isBoardCreated = true; // 게시글 저장 성공 플래그
      System.out.println("게시글 저장 성공");
  
      // 게시글 작성 시 작성자에게 활동 점수 10점 부여
      userClient.addActivityScore(createBoardRequestDto.getUserId(), 10);
      System.out.println("포인트 적립 성공");
      
    } catch (Exception e) {
    
      if (isBoardCreated) {
        // 게시글 작성 보상 트랜잭션 => 게시글 삭제
        this.boardRepository.deleteById(savedBoardId);
        System.out.println("[보상 트랜잭션] 게시글 삭제");
      }
      
      if (isPointDeducted) {
        // 포인트 차감 보상 트랜잭션 => 포인트 적립
        pointClient.addPoints(createBoardRequestDto.getUserId(), 100);
        System.out.println("[보상 트랜잭션] 포인트 적립");
      }
      
      // 실패 응답으로 처리하기 위해 예외 던지기
      throw e;
    }
  }
  ```
