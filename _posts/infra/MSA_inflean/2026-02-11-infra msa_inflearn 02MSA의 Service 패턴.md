---
title: 02 MSA의 Service 패턴
date: 2026-02-11 10:00:00 +09:00
categories: [infra, MSAInflearn]
tags: [ infra, MSAInflearn ]
---

## 1. memo
 - 각기 기능을 하는 Service들을 만들어놓고 필요에 따라서 상호보완으로 작업함.
 - MVC를 만들어서 그것을 기준으로 할경우
   - web에서 action이 발생 -> 서버 다른 AP요청보다는
   - Web에서 비동기방식으로 특정 AP에 요청 구조가 적합.
 - 스프링에서는 Spring Gateway 방식을 사용해서 요청을 뿌려주는 역할로도 사용함.
 - SSR/CSR + 모놀리식/MSA
   - 모놀리식 + SSR이 잘 맞았던 이유는 화면 렌더링 / 비즈니스 로직 / 데이터 접근 과정이 자기 안에서 모든 것이 해결 가능했으나
   - MSA는 SSR 서버로 와도 어차피 다른 서비스에 요청을 해야하기 때문에 차이가 없음.
   - 경계가 사라지고, 영향이 줄어듬!
   - 장애를 줄이자!

## 2. User Service
### 1. memo
 - 유저 Service에서는 유저에 대한 정보만 처리함.

### 2. UserController
  ```
  //유저 가입
  @PostMapping("/sign-up")
  public ResponseEntity<Void> signUp(@RequestBody SignUpRequestDto signUpRequestDto) {
      userService.signUp(signUpRequestDto);
      return ResponseEntity.noContent().build();
  }

  // 유저 정보 조회
  @GetMapping("{userId}")
  public ResponseEntity<UserResponseDto> getUser(@PathVariable Long userId) {
      UserResponseDto userResponseDto = userService.getUser(userId);
      return ResponseEntity.ok(userResponseDto);
  }
  ```

### 3. UserService
  ```
  // 신규가입
  @Transactional
  public void signUp(SignUpRequestDto signUpRequestDto) {
     User user = new User(
             signUpRequestDto.getEmail(),
             signUpRequestDto.getName(),
             signUpRequestDto.getPassword()
     );

     this.userRepository.save(user);
  }

  // 유저정보 return
  public UserResponseDto getUser(Long id) {
      User user = userRepository.findById(id)
              .orElseThrow(() -> new IllegalArgumentException("사용자를 찾을 수 없습니다."));

      return new UserResponseDto(
              user.getUserId(),
              user.getEmail(),
              user.getName()
      );
  }
  ```

### 4. DTO
 - SignUpRequestDto : email / name / password
 - UserResponseDto :  email / name / password

### 5. domain
 - User : userId / email / name/ password


## 3. board Service
### 1. memo
 - board Service는 user정보를 가지고 내용을 생성해야하기 때문에, 중간에 UserService와 커넥션이 필요함.
 - 지금 예시에서는 " RestClient " 을 사용함
 - 필요에 따라서 여러개 동시에 사용하기도 하고 비즈니스로직에 따라 다름!
 - 굳이 중심이 되는 gateway나 MVC, 웹에 다녀올 필요없음.
 
### 2. boardController
  ```
  @PostMapping
  public ResponseEntity<Void> create(@RequestBody CreateBoardRequestDto createBoardRequestDto) {
      boardService.create(createBoardRequestDto);
      return ResponseEntity.noContent().build();
  }
  
  @GetMapping("/{boardId}")
  public ResponseEntity<BoardResponseDto> getBoard(@PathVariable Long boardId) {
      BoardResponseDto boardResponseDto = boardService.getBoard(boardId);
      return ResponseEntity.ok(boardResponseDto);
  }
  ```

### 3. boardservice
  ```
  private final BoardRepository boardRepository;
  private final UserClient userClient;
    
  @Transactional
  public void create(CreateBoardRequestDto createBoardRequestDto) {
      Board board = new Board(
              createBoardRequestDto.getTitle(),
              createBoardRequestDto.getContent(),
              createBoardRequestDto.getUserId()
      );

      this.boardRepository.save(board);
  }

  public BoardResponseDto getBoard(Long boardId) {
      // 게시글 불러오기
      Board board = boardRepository.findById(boardId)
              .orElseThrow(() -> new IllegalArgumentException("게시글을 찾을 수 없습니다."));

      // user-service로부터 사용자 정보 불러오기
      Optional<UserResponseDto> optionalUserResponseDto = userClient.fetchUser(board.getUserId());
      
      UserDto userDto = null; // 기본값이 Null
      // 유저정보를 가져왔다면 dto에 셋팅
      // 가져오지 않았따면 if작업 X 메모리 보존
      if (optionalUserResponseDto.isPresent()) { 
        UserResponseDto userResponseDto = optionalUserResponseDto.get();
        userDto = new UserDto(
            userResponseDto.getUserId(),
            userResponseDto.getName()
        );
      }
            
      BoardResponseDto boardResponseDto = new BoardResponseDto(
              board.getBoardId(),
              board.getTitle(),
              board.getContent(),
              userDto
      );


      return boardResponseDto;
  }
  ```

### 4. UserClient
  ```
  private final RestClient restClient;

  // 지금 프로젝트는 RestClient를 사용하여 통신함.
  public UserClient( @Value("${client.user-service.url}") String userServiceUrl ) {
      this.restClient = RestClient.builder()
              .baseUrl(userServiceUrl)
              .build();
  }

  // 최종적으로 사용 가능한 통신 방법으로 특정 정보를 얻어줌
  public Optional<UserResponseDto> fetchUser(Long userId) {
    try {
      UserResponseDto userResponseDto = this.restClient.get()
          .uri("/users/{userId}", userId)
          .retrieve()
          .body(UserResponseDto.class);
      return Optional.ofNullable(userResponseDto);
    } catch (RestClientException e) {
      // 장애가 있을 경우 비어있는 값으로 리턴함       
      return Optional.empty();
    }
  }
  ```

### 5. DTO
 - BoardResponseDto : boardId / title / content / user 
 - CreateBoardRequestDto :  title / content / userId
 - UserDto : UserId / name
 - UserResponseDto : userId / email / board

### 6. domain
 - Board : boardId / title / content / userId

