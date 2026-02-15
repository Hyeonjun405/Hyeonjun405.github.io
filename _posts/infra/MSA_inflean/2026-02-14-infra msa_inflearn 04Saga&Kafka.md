---
title: 04 Saga&kafka
date: 2026-02-14 10:00:00 +09:00
categories: [infra, MSAInflearn]
tags: [ infra, MSAInflearn ]
---

## 1. 사용자 포인트 부여
### 1. memo
 - 게시글 작성전 포인트차감 -> 게시글 작성 -> 활동점수 부여 서비스 로직
 - 카프카를 이용해서 동기와 비동기 어떻게 나눌 것인가가 중요한 포인트
   - 사용자가 AP에 접근하여 액션을 취하고 그 결과에 대한 작업을 처리하는 과정
   - 사용자가 진행한 이벤트에 대한 결과를 제공해야하고,
   - 사용자 이벤트로 인한 추가 작업을 적용해줘야함.
 - 지금 작업에서는
   - 활동 점수를 부여한다는 것은 사용자 입장에서는 상대적으로 크게 중요하지 않은 부분이라 판단,
   - 그래서 활동점수 부여를 카프카로 넘겨서, 추후에 작업할 수 있도록하고
   - 작업 자체를 완료하면 완료했다고 사용자에게 알림을 보냄.
   - 만약 활동점수 부여의 실패는 게시글 작성과 무관하므로 추후에 수동으로 처리가 가능한 부분 
 - 사용자의 입장과 전체 비즈니스로직의 흐름을 구분하여 동기/비동기 작업 구분이 필요함.

  
### 2. 상황
![내 그림](assets/img/infra/msa_inflearn/04/msa&kafka.png "이미지")

### 3. 자바소스
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
      System.out.println("활동 점수 적립 성공");
      
      // '게시글 작성 완료' 이벤트 발행
      BoardCreatedEvent boardCreatedEvent = new BoardCreatedEvent(createBoardRequestDto.getUserId());
      this.kafkaTemplate.send("board.created", toJsonString(boardCreatedEvent));
      System.out.println("게시글 작성 완료 이벤트 발행");

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
 
 ```

## 2. MSA환경의 서비스간 소통
### 1. memo
- 게시글을 조회할때 유저서비스와 커넥션해서 데이터를 가져와서 사용해야함.
- 그러면 게시글을 조회할때 유저서비스와 통신을 반복적으로 해야하는 상황이 생김
- 따라서
  - 게시글 서비스에 게시글에 필요한 유저정보를 저장함.
  - 보드서비스에서는 카프카로 유저정보를 계속 확인해서 저장하는 형태로 변경.
  - 유저서비스에서는 신규 생성/변경이 되면 카프카로 유저정보에 관한 정보를 계속 보내둠
  - 그러면 따로 계속 소통할 필요는 없고, 게시글에 관한 최소한의 정보만 보드서비스에서 가지고 있는 형태가됨.
  - 이렇게 되면, 보드서비스 외에 다른 곳에서 유저ID와 관련된 내용이 필요하면 그 카프카정보만 가져다가 쓰면됨!
  - 상세한 정보는 유저서비스에서 직접 보는걸 유지함!
  - 그러면 유저서비스에 트래픽이 몰리는게 막아짐!!!!

### 2. 상황
![내 그림](assets/img/infra/msa_inflearn/04/msa&kafka2.png "이미지")

### 3. 자바소스
#### 1. 유저 서비스(MSA AP)
  ```
  @Transactional
  public void signUp(SignUpRequestDto signUpRequestDto) {
    User user = new User(
      signUpRequestDto.getEmail(),
      signUpRequestDto.getName(),
      signUpRequestDto.getPassword()
    );

    User savedUser = this.userRepository.save(user);

    // 회원가입하면 포인트 1000점 적립
    pointClient.addPoints(savedUser.getUserId(), 1000);

    // '회원가입 완료' 이벤트 발행
    UserSignedUpEvent userSignedUpEvent = new UserSignedUpEvent(
        savedUser.getUserId(),
        savedUser.getName()
    );
    // 카프카에 지속적으로 신규 유저에 대한 정보를 보냄.
    this.kafkaTemplate.send(
        "user.signed-up",
        toJsonString(userSignedUpEvent)
    );
  }
  ```

#### 2. 보드 서비스(MSA AP)
  ```
  @KafkaListener(
      topics = "user.signed-up",
      groupId = "board-service"
  )
  public void consume(String message) {
    UserSignedUpEvent userSignedUpEvent = UserSignedUpEvent.fromJson(message);

    // 사용자 정보 저장
    // 여기서 서비스는 보드서비스 내에서 유저정보를 저장하는 서비스
    SaveUserRequestDto saveUserRequestDto
        = new SaveUserRequestDto(
            userSignedUpEvent.getUserId(),
            userSignedUpEvent.getName()
        );
    userService.save(saveUserRequestDto);
  }  
  ```
