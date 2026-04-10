---
title: 05 Redis 캐싱 / 영속성
date: 2026-04-10 10:00:00 +09:00
categories: [Sparta, SpartaWIL05]
tags: [ Java, Spring Framework ]
---

## 1. Note
 - 정답은 없다. 
   - 모든 상황에 대해서 케바케에 따라서 전략을 달리해야함.
   - 하지만 어디부터 어디까지 케바케로 봐야할지 고민이 필요할듯.
   - 뭐 그냥 상황마다 다르니까 그냥 여기는 그러겟지? 라고간다면
   - 굉장히 대안없이 수용하는 모습이 될듯.


## 2. Reids 캐싱

```
redisTemplate.opsForValue().set(
    CACHE_KEY_CATEGORY,   // key: Redis에 저장될 키 (데이터 식별자)
    cacheCategories,      // value: 저장할 실제 데이터 (객체/List 등, 직렬화됨)
    1,                    // timeout: 만료 시간 값
    TimeUnit.HOURS        // unit: 시간 단위 (여기서는 1시간 뒤 만료)
);
```

## 3. Redis - SpringBoot - 패턴 구현
### 1. Cache-aside (캐시-어사이드) 패턴 구현

```
@Transactional(readOnly = true)
public List<CategoryResponse> findAllForCacheAside() {
  
  // 캐시에서 카테고리 구조 데이터 조회 시도
  String cachedCategories = redisTemplate.opsForValue().get(CACHE_KEY_CATEGORY);

  // 캐시 히트
  if (!ObjectUtils.isEmpty(cachedCategories)) {
    log.info("Cache Hit: categoryStruct for key = {}", CACHE_KEY_CATEGORY);
    return JsonUtil.fromJsonList(cachedCategories, CategoryResponse.class);
  }

  // 캐시 미스, 데이터베이스에서 조회 (findAll() 호출)
  log.info("Cache Miss: categoryStruct for key = {}", CACHE_KEY_CATEGORY);
  List<CategoryResponse> categories = findAll();

  // 데이터베이스에서 조회한 데이터를 캐시에 저장
  if (!categories.isEmpty()) {
    String cacheCategories = JsonUtil.toJson(categories);
    redisTemplate.opsForValue().set(CACHE_KEY_CATEGORY, cacheCategories,1,TimeUnit.HOURS);
  }
  return categories;
}
```

### 2. Write-through (라이트-스루) 전략

```
  @Transactional
  public void saveWriteThrough(CategoryRequest request) {
    try {
    
      create(request); // 데이터 베이스 신규 카테고리 저장
      
      // 캐시 업데이트
      try {
        // 데이터를 한번 전체 로딩해버리면 필요한것만 분류 작업하거나 그런게 필요없음
        // 그래서 그냥 전체 한번 로드해서 영속성 엔티티에 띄우고 캐싱함.
        // findAll은 카테고리 전체 만드는 로직임. (지금 내부로직 중요 X)
        List<CategoryResponse> categories = findAll();
  
        if (!categories.isEmpty()) {
          String cacheCategories = JsonUtil.toJson(categories);
          redisTemplate.opsForValue().set(CACHE_KEY_CATEGORY, cacheCategories);
        }
      } catch (Exception e) {
        log.error("Error updating cache key {}: {}", CACHE_KEY_CATEGORY, e.getMessage());
      }
      
    } catch (Exception e) {
      log.error("Failed to save category with Write-through: {}", e.getMessage(), e);
    }
  }
```

### 3. Write-back (라이트-백) 전략

```
@Transactional
public void saveWriteBack(CategoryRequest request) {
  try {
    // 캐시에서 카테고리 구조 데이터 조회 시도
    String cachedCategories = redisTemplate.opsForValue().get(CACHE_KEY_CATEGORY);

    List<CategoryResponse> categories = new ArrayList<>();

    // 캐싱 데이터가 있다면 캐싱 데이터 불러오기
    if (!ObjectUtils.isEmpty(cachedCategories)) {
      categories = JsonUtil.fromJsonList(cachedCategories, CategoryResponse.class);
    }

    // 캐시에 새로운 카테고리 데이터 추가
    CategoryResponse newCacheCategory = CategoryResponse.builder()
        .name(request.getName())
        .childCategories(new ArrayList<>())
        .build();

    // ID가 auto-increment면 ID를 넣을 방법이 없음
    // 그래서 back-pattern은 UUID로 처리하거나, 다른 전략을 사용해야함.
    categories.add(newCacheCategory);

    // Redis에 우선 저장
    String cacheCategories = JsonUtil.toJson(categories);
    redisTemplate.opsForValue().set(CACHE_KEY_CATEGORY, cacheCategories);

    // 데이터베이스 저장 작업은 비동기로 처리
    saveToDatabaseAsync(request);

  } catch (Exception e) {
    log.error("Write-back 패턴 저장 실패: {}", e.getMessage());
  }
}

@Async // 비동기 방식
@Transactional(propagation = Propagation.REQUIRES_NEW) // 트랜잭션 분리
public void saveToDatabaseAsync(CategoryRequest request) {
  try {
    create(request);
  } catch (Exception e) {
    log.error("비동기 DB 저장 실패: {}", e.getMessage(), e);
  }
}
```

## 4. Redis 최적화 - TTL
### 1. 만료 시간 설정 (TTL: Time To Live)
 - TTL(Time To Live)
 - Redis에 저장된 데이터가 얼마 동안 유지될지를 설정하는 시간
 - 설정된 시간이 지나면 해당 데이터는 자동으로 삭제

### 2. 필요성
- 메모리 효율성
  - 불필요하거나 더 이상 사용되지 않는 데이터를 자동으로 제거함으로써 Redis의 메모리 사용량을 안정적으로 유지
  - Redis는 메모리 기반 저장소이기 때문에, 이러한 자동 정리 메커니즘은 자원 관리 측면에서 매우 중요
- 데이터 일관성 보조
  - 캐시 데이터는 원본 데이터와 시점 차이가 발생할 수밖에 없음
  - TTL을 설정하면 일정 시간이 지난 후 캐시가 제거되고, 이후 요청 시 최신 데이터를 다시 조회
  - 이를 통해 데이터 불일치 문제를 완화
- 휘발성 데이터 관리
  - 세션 정보, 인증 토큰, 일시적인 알림과 같이 유효 기간이 명확한 데이터는 TTL을 통해 자연스럽게 관리
  - 별도의 삭제 로직 없이도 자동으로 만료되기 때문에 구현이 단순해지고 실수 가능성도 줄어듬

### 3. 명령어

```
# mykey를 "hello"로 설정하고 5분(300초) 후 만료
# 동일 명령어
SET mykey "hello" EX 300  
SETEX mykey 300 "hello"   

# session:user:123 키에 1시간(3600초)의 만료 시간 설정
EXPIRE session:user:123 3600 

# temp_data 키를 5초(5000밀리초) 후 만료
PEXPIRE temp_data 5000 

# mykey의 남은 만료 시간 확인
TTL mykey 

# temp_data 키의 만료 시간을 제거
PERSIST temp_data 
```

## 5. LRU & LFU
### 1. LRU 
#### 1. LRU (Least Recently Used)
- 가장 최근에 사용되지 않은 데이터를 먼저 삭제하는 방식
- 동작 방식
  - 각 데이터는 “마지막으로 접근된 시간”을 기준으로 관리
  - 오랫동안 조회되지 않은 데이터일수록 우선적으로 제거
- 특징
  - 최근에 사용된 데이터는 다시 사용될 가능성이 높다는 가정 기반
  - 구현이 단순하고 직관적
  - 일반적인 캐시에서 가장 널리 사용됨

#### 2. 장단점
- 장점
  - 최근 사용된 데이터를 기준으로 하기 때문에 최신 트렌드 반영이 빠름
  - 구현이 단순하고 대부분의 캐시 상황에서 안정적으로 동작
  - 사용자 행동 변화(패턴 변화)에 빠르게 적응
- 단점
  - 자주 사용되던 데이터라도 한동안 안 쓰이면 바로 삭제될 수 있음
  - 단 한 번 사용된 데이터라도 최근이면 불필요하게 살아남을 수 있음
  - “사용 빈도”를 고려하지 않음

### 2. LFU
#### 1. LFU (Least Frequently Used)
- 가장 적게 사용된 데이터를 먼저 삭제하는 방식
- 동작 방식
  - 각 데이터마다 “사용 횟수”를 카운트
  - 사용 횟수가 적은 데이터부터 제거
- 특징
  - “자주 사용되는 데이터는 중요하다”는 가정 기반
  - 장기적인 사용 패턴을 반영
  
#### 2. 장단점
- 장점
  - 자주 사용되는 데이터를 유지하기 때문에 핵심 데이터 보존에 유리
  - 반복 조회가 많은 서비스에서 캐시 효율이 높음
  - 인기 데이터 중심 구조(랭킹, 조회수 등)에 적합
- 단점
  - 과거에 많이 사용된 데이터가 계속 남아 최신 트렌드 반영이 느림
  - 사용 패턴이 바뀌어도 적응 속도가 느림
  - 구현 및 관리가 LRU보다 상대적으로 복잡

### 3. redis.conf(설정파일)

```
# 스프링부트에서는 그냥 사용만하고,
# 레디스의 redis.conf파일에서 LRU / LFU 설정

# 최대 메모리 512MB 설정
maxmemory 512mb

# 모든 키에 대해 LRU 정책 사용 (가장 보편적)
maxmemory-policy allkeys-lru 

# 또는 모든 키에 대해 LFU 정책 사용 (사용 빈도 기반)
# maxmemory-policy allkeys-lfu

# 또는 만료 시간이 설정된 키에 대해서만 LRU 사용
# maxmemory-policy volatile-lru

# 또는 메모리 가득 차면 쓰기 거부 (데이터 손실 없음)
# maxmemory-policy noeviction
```

## 6. Redis의 영속성
### 1. Redis의 영속성
 - 데이터를 디스크에 저장하는 다양한 영속성(Persistence) 옵션을 제공
 

### 2. RDB 방식
#### 1. RDB (Snapshot) 방식
- 특정 시점의 메모리 데이터를 통째로 복사해서 디스크에 파일(.rdb)로 저장하는 방식
- 카메라로 스냅샷을 찍듯이 현재 상태를 저장

#### 2. 장단점
- 장점
  - 파일 크기가 작고, 복구 속도가 매우 빠릅니다. 특히 대량의 데이터를 복구할 때 효율적
  - 스냅샷 생성 과정이 백그라운드에서 수행되므로, Redis 서버의 성능 저하가 비교적 적음
- 단점
  - 스냅샷 주기 사이의 변경 데이터는 유실될 수 있음(스냅샷과 스냅샷 사이에 롤백되는 경우)
  - fork()로 인해 대용량 환경에서 CPU와 메모리 오버헤드가 발생할 수 있음

#### 3. redis.conf

```
# 60초마다 최소 1000개의 키가 변경되었을 경우 스냅샷 저장
save 60 1000  

# 300초(5분)마다 최소 10개의 키가 변경되었을 경우 스냅샷 저장
save 300 10   
```

### 3. AOF 방식
#### 1. AOF (Append-Only File) 방식
- 서버에서 발생하는 모든 쓰기(Write) 연산을 명령어 형태로 로그 파일(.aof)에 순차적으로 기록하는 방식
- 데이터베이스의 트랜잭션 로그와 유사하다고 볼 수 있음

#### 2. 장단점
- 장점
  - RDB보다 데이터 유실 가능성이 낮으며, 설정에 따라 유실 범위를 최소화할 수 있음
  - 명령어 로그 기반으로 저장되어, 장애 발생 시 재실행을 통해 데이터 복구가 가능
- 단점
  - 모든 쓰기 연산을 기록하므로 RDB보다 파일 크기가 커질 수 있음
  - 쓰기 작업이 많을 경우 디스크 I/O로 인해 성능 저하가 발생할 수 있음

#### 3. redis.conf

```
# AOF 기능 활성화
appendonly yes 

# 매 초마다 디스크에 로그 동기화 (기본값)
appendfsync everysec 

# 모든 쓰기 명령마다 디스크에 동기화 (느리지만 최고 안전)
# appendfsync always 

# 운영체제가 알아서 동기화 (가장 빠르지만 위험)
# appendfsync no 
```
