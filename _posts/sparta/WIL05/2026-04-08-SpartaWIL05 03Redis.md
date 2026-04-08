---
title: 03 Redis
date: 2026-04-08 10:00:00 +09:00
categories: [Sparta, SpartaWIL05]
tags: [ Java, Spring Framework ]
---

## 1. Note
- 튜터님 "DB나 레디스같은거는 장애가 났을때 대응하는게 어렵기때문에 왠만하면 온프레미스는 비추!"
- 스프링이라서 그런건지 간단하게 접근이 가능함
  - 의존성 추가하면 자동으로 Session으로 들어가게됨.
  - 프로퍼티스만 잡아주면 지속적으로 변경도 가능함.
- 이것을 어떻게 활용해야하는지가 많은 관건일듯
  - 캐시가 공유되면 파드가 나뉘어도 공유 메모리를 사용하기 때문에 유리!
    ```
     WAS1 ─┐
     WAS2 ─┼── Redis (세션 공유)
     WAS3 ─┘
    ```

## 2. Redis 셋팅
### 1. Docker
  ```
  docker run -d --name redis-server -p 6379:6379 redis
  
  docker ps

  CONTAINER ID   IMAGE       COMMAND                   CREATED         STATUS         PORTS                                         NAMES
  f99e1b2ea389   mysql:8.0   "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   0.0.0.0:3306->3306/tcp, [::]:3306->3306/tcp   spring-mysql
  fe8fbe6e4de1   redis       "docker-entrypoint.s…"   6 minutes ago   Up 6 minutes   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp   redis-server
  
  # 기본적인 명령어
  set key1 "value1"
  get key1
  ```

### 2. Spring
#### 1. 의존성
```
// 추가할경우 setAttribute를 하게되면 httpSession이 레디스에 저장하는걸로 변경됨.
implementation 'org.springframework.session:spring-session-data-redis'

// 레디스를 연결하는 용도
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

#### 2. application
```
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: # 비밀번호가 없다면 비움.
      # timeout: 1s
      # database: 0
```

#### 3. main
```
@SpringBootApplication
@EnableRedisHttpSession // 간혹버전에 따라서 명시해줘야하는 것도 있음(옛날버전)
public class ExamplePart3Application {

  public static void main(String[] args) {
    SpringApplication.run(ExamplePart3Application.class, args);
  }
```

#### 4. 주입
```
@Autowired
private RedisTemplate<String, String> redisTemplate;
```

## 3. Spring & Redis
### 1. String
#### 1. 주요 메소드

  | 메소드                             | 설명     | 리턴타입 |
  | ------------------------------- | ------ | ---- |
  | `opsForValue().set(key, value)` | 키-값 저장 | void |
  | `opsForValue().get(key)`        | 값 조회   | V    |
  | `opsForValue().increment(key)`  | 숫자형 증가 | Long |
  | `opsForValue().decrement(key)`  | 숫자형 감소 | Long |


#### 2. 기본 set/get
```
// myRedisKey로 "Hello World!" 저장
redisTemplate.opsForValue().set("myRedisKey", "Hello World!");

// myRedisKey키로 레디스에서 조회
String myRedisValue = redisTemplate.opsForValue().get("myRedisKey");
```

#### 3. 연산
```
// 레디스는 기본값이 0

// 1 증가 (jedis.incr)
// 키가 없어서 0에서 시작하여 1을 반환함.
Long pageViewsAfterIncr = redisTemplate.opsForValue().increment(pageViewsKey);
log.info("{}: 1 증가 후: {}", pageViewsKey, pageViewsAfterIncr); // 1

// 1 감소 (jedis.decr)
Long pageViewsAfterDecr = redisTemplate.opsForValue().decrement(pageViewsKey);
log.info("{}: 1 감소 후: {}", pageViewsKey, pageViewsAfterDecr); // 0

// 3. 특정 값(5) 증가 (jedis.incrBy)
Long likesAfterIncrBy = redisTemplate.opsForValue().increment(likesKey, 5);
log.info("{}: 5 증가 후: {}", likesKey, likesAfterIncrBy); // 5

// 4. 데이터 조회 및 Long 변환
String pageViewsStr = redisTemplate.opsForValue().get(pageViewsKey);
long pageViews = Long.parseLong(pageViewsStr);
log.info("최종 조회된 페이지 뷰 수 (Long 변환): {}", pageViews); // 0
```

### 2. list
#### 1. 주요메소드
  
  | 메소드                                   | 설명        | 리턴타입          |
  | ------------------------------------- | --------- | ------------- |
  | `opsForList().leftPush(key, value)`   | 왼쪽 삽입     | Long (리스트 길이) |
  | `opsForList().rightPush(key, value)`  | 오른쪽 삽입    | Long          |
  | `opsForList().leftPop(key)`           | 왼쪽에서 꺼내기  | V             |
  | `opsForList().rightPop(key)`          | 오른쪽에서 꺼내기 | V             |
  | `opsForList().range(key, start, end)` | 지정 구간 조회  | List<V>       |


#### 2. 예시
```
// 작업 대기열 생성 (LIFO 스택 방식 구현 - Head(왼쪽)에 삽입)
// 왼쪽부터 한개씩 들어감.
// Task1 들어가고 그다음에 왼쪽에 Task2넣고 그다음에 왼쪽에 Task3 넣음
// 최종구조가 Task3 - Task2 - Task1
Long list = redisTemplate.opsForList().leftPushAll(TASKS_KEY, "Task1", "Task2", "Task3");

// rightPop
// Queue => FIFO (First In First Out)
String task1 = redisTemplate.opsForList().rightPop(TASKS_KEY);
log.info("RightPop (Task1 처리): {}", task1); // Task1

// leftPop 
// Stack => LIFO (Last In First Out)
String task2 = redisTemplate.opsForList().leftPop(TASKS_KEY);
log.info("RightPop (Task2 처리): {}", task2); // Task3 
```

### 3. Set
#### 1. 주요메소드
  
  | 메소드                                 | 설명       | 리턴타입            |
  | ----------------------------------- | -------- | --------------- |
  | `opsForSet().add(key, value...)`    | 멤버 추가    | Long (추가된 멤버 수) |
  | `opsForSet().remove(key, value...)` | 멤버 삭제    | Long (삭제된 멤버 수) |
  | `opsForSet().members(key)`          | 전체 멤버 조회 | Set<V>          |
  | `opsForSet().isMember(key, value)`  | 멤버 존재 확인 | Boolean         |


#### 2. 기본형태 
```
Long addedCount = redisTemplate.opsForSet().add(PARTICIPANTS_KEY, "User1", "User2", "User3", "User1");
log.info("Participants에 추가된 고유 사용자 수: {}", addedCount); // 3

Set<String> participants = redisTemplate.opsForSet().members(PARTICIPANTS_KEY);
log.info("{}: 고유 사용자 목록: {}", PARTICIPANTS_KEY, participants); 
// {"User1", "User2", "User3"} (순서 보장 안됨)

// 집합 데이터
redisTemplate.opsForSet().add(EVENT1_KEY, "User1", "User2", "User4");
redisTemplate.opsForSet().add(EVENT2_KEY, "User2", "User3", "User4");

// 교집합 구하기
// intersect: EVENT1과 EVENT2의 공통 멤버를 Set으로 반환
Set<String> commonUsers = redisTemplate.opsForSet().intersect(EVENT1_KEY, EVENT2_KEY);
log.info("교집합 (Common Users): {}", commonUsers); // {"User2", "User4"}

// 합집합 구하기 (jedis.sunion)
// union: EVENT1과 EVENT2의 모든 멤버를 합쳐 Set으로 반환
Set<String> allUsers = redisTemplate.opsForSet().union(EVENT1_KEY, EVENT2_KEY);
log.info("합집합 (All Users): {}", allUsers); // {"User1", "User2", "User3", "User4"}

// 차집합 구하기 (EVENT1에는 있지만 EVENT2에는 없는 사용자)
// difference: 첫 번째 키(EVENT1)에는 있지만 나머지 키(EVENT2)에는 없는 멤버를 반환합니다.
Set<String> onlyInEvent1 = redisTemplate.opsForSet().difference(EVENT1_KEY, EVENT2_KEY);
log.info("차집합 (Only in Event1): {}", onlyInEvent1); // {"User1"}
```

### 3. Hash
#### 1. 주요메소드

| 메소드                                   | 설명          | 리턴타입            |
| ------------------------------------- | ----------- | --------------- |
| `opsForHash().put(key, field, value)` | 필드-값 저장     | void            |
| `opsForHash().get(key, field)`        | 필드 값 조회     | V               |
| `opsForHash().entries(key)`           | 전체 필드-값 조회  | Map<H, V>       |
| `opsForHash().delete(key, field...)`  | 필드 삭제       | Long (삭제된 필드 수) |
| `opsForHash().hasKey(key, field)`     | 필드 존재 여부 확인 | Boolean         |


#### 2. 기본형태
```
// hSet: Hash에 특정 필드(Field)와 값(Value)을 저장합니다.
redisTemplate.opsForHash().put(USER_PROFILE_KEY, "name", "John Doe");
redisTemplate.opsForHash().put(USER_PROFILE_KEY, "email", "john.doe@example.com");
redisTemplate.opsForHash().put(USER_PROFILE_KEY, "age", "30");
redisTemplate.opsForHash().put(USER_PROFILE_KEY, "city", "Seoul");

String name = (String) redisTemplate.opsForHash().get(USER_PROFILE_KEY, "name");
String email = (String) redisTemplate.opsForHash().get(USER_PROFILE_KEY, "email");

// entries: Hash Key의 모든 필드-값 쌍을 Map<Object, Object> 형태로 반환합니다.
Map<Object, Object> userProfile = redisTemplate.opsForHash().entries(USER_PROFILE_KEY);

log.info("{}: 모든 필드-값 쌍 조회: {}", USER_PROFILE_KEY, userProfile);
// 출력: {name=John Doe, email=john.doe@example.com, age=30, city=Seoul}
```

### 4. Sorted Set (정렬된 집합)
#### 1. Sorted Set
- *값(Value)*과 함께 *숫자 형태의 점수(Score)*를 저장하는 데이터 구조
- 저장된 요소들은 이 점수를 기준으로 항상 오름차순 또는 내림차순으로 자동 정렬
- 중복된 값은 허용하지 않지만, 중복된 점수는 허용 (점수가 같으면 값이 사전식으로 정렬)
- 특정 점수 범위나 순위 범위에 해당하는 데이터를 효율적으로 조회

#### 2. 주요 메소드

| 메소드                                                    | 설명            | 리턴타입               |
| ------------------------------------------------------ | ------------- | ------------------ |
| `opsForZSet().add(key, member, score)`                 | 멤버 점수와 함께 추가  | Boolean            |
| `opsForZSet().incrementScore(key, member, delta)`      | 점수 증가/감소      | Double             |
| `opsForZSet().range(key, start, end)`                  | 오름차순 조회       | Set<V>             |
| `opsForZSet().reverseRange(key, start, end)`           | 내림차순 조회       | Set<V>             |
| `opsForZSet().rangeWithScores(key, start, end)`        | 점수 포함 오름차순 조회 | Set<TypedTuple<V>> |
| `opsForZSet().reverseRangeWithScores(key, start, end)` | 점수 포함 내림차순 조회 | Set<TypedTuple<V>> |
| `opsForZSet().rank(key, member)`                       | 오름차순 순위 조회    | Long               |
| `opsForZSet().reverseRank(key, member)`                | 내림차순 순위 조회    | Long               |
| `opsForZSet().score(key, member)`                      | 멤버 점수 조회      | Double             |
| `opsForZSet().remove(key, member...)`                  | 멤버 삭제         | Long               |

#### 3. 저장/조회
```
// add: 값(member)과 점수(score)를 함께 저장합니다.
redisTemplate.opsForZSet().add(LEADERBOARD_KEY, "Player1", 1500.0);
redisTemplate.opsForZSet().add(LEADERBOARD_KEY, "Player2", 2000.0);
redisTemplate.opsForZSet().add(LEADERBOARD_KEY, "Player3", 1800.0);

// reverseRange - 점수가 높은 순(내림차순)으로 지정된 범위(0번째부터 1번째까지, 즉 상위 2명)의 멤버를 조회
// range - 오름차순( ※ Score들은 보통 Score가 높으면 등수가 높음)
Set<String> topPlayers = redisTemplate.opsForZSet().reverseRange(LEADERBOARD_KEY, 0, 1);

log.info("상위 2명 조회 (내림차순): {}", topPlayers);
// 결과: [Player2, Player3] (Redis의 순서대로 출력되며, Set이 아닌 List와 유사한 순서로 반환됨)

// incrementScore - 특정 멤버의 점수를 지정된 값만큼 증가
// 감소의 경우에는 마이너스로 넣음
Double newScore = redisTemplate.opsForZSet().incrementScore(LEADERBOARD_KEY, "Player1", 100.0);
log.info("Player1의 점수 업데이트: 1500.0 -> {}", newScore); // 1600.0

// rangeByScore - 지정된 점수 범위 내의 멤버를 조회
Set<String> playersInRange = redisTemplate.opsForZSet().rangeByScore(LEADERBOARD_KEY, 1600.0, 1900.0);
log.info("점수 1600.0 ~ 1900.0 범위 플레이어: {}", playersInRange);
// 결과: [Player1, Player3]

// rank - 점수가 낮은 순(오름차순)으로 해당 멤버의 순위를 반환
Long rankOfPlayer3 = redisTemplate.opsForZSet().rank(LEADERBOARD_KEY, "Player3");
log.info("Player3의 오름차순 순위 (0부터 시작): {}", rankOfPlayer3); 
// 1 (0:Player1, 1:Player3, 2:Player2)
```


