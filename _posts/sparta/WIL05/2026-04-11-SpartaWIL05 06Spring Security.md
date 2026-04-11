---
title: 06 Spring Security
date: 2026-04-11 10:00:00 +09:00
categories: [Sparta, SpartaWIL05]
tags: [ Java, Spring Framework ]
---

## 1. Note
- 오 간단한 듯 복잡한듯 어렵
- 기본적인 흐름만 먼저 파악해두고
  - 보안과 관련된 공부가 먼저 필요할듯함.
  - "무엇인가를 막는다?" 또는 "예방한다" 하는데 그게 뭔지 모르니까
  - 점점 어려워짐

## 2. Spring Security
### 1. Spring Security
- 스프링 기반 애플리케이션에서 인증(Authentication)과 권한(Authorization) 관리를 담당하는 프레임워크
- 웹, REST API, 메서드 수준 보안까지 포괄적 보안 기능 제공

### 2. 핵심기능
- 인증(Authentication)
  - 사용자가 누구인지 확인
  - 로그인 폼, JWT, OAuth2, SSO 등 다양한 방식 지원
- 권한(Authorization)
  - 인증된 사용자가 어떤 리소스에 접근할 수 있는지 결정
  - URL 단위, 메서드 단위(@PreAuthorize, @Secured) 등
- 보안 필터 체인(Filter Chain)
  - 요청(Request)이 들어오면 Filter들이 순서대로 처리
  - 인증, 권한 체크, 세션 관리, CSRF, CORS 등
- SecurityContext / ThreadLocal
  - 인증 정보(Authentication)를 요청 동안 공유
  - Controller, Service, AOP, Filter 등 어디서든 접근 가능
- 세션/토큰 관리
  - 로그인 상태 유지, 다음 요청에서 인증 정보 사용 가능
- 패스워드 암호화
  - PasswordEncoder를 통한 안전한 비밀번호 저장 및 검증

### 3. 전반적인 흐름
```
- 사용자가 로그인 요청
- Spring Security → UserDetailsService 호출
- CustomUserDetailsService.loadUserByUsername() 실행
- DB에서 사용자 조회
- CustomUserDetails로 감싸서 반환
- Security가 비밀번호 비교 및 인증 수행
```


## 3. Spring Security 사전 셋팅
### 1. 의존성
```
dependencies {
    // Spring Security 프레임워크를 통합하기 위한 의존성
    implementation 'org.springframework.boot:spring-boot-starter-security'
    
    // Spring Session과 Redis를 연동하기 위한 의존성
    // 이 의존성을 추가하면 Redis를 세션 저장소로 사용할 수 있도록 Spring Session이 자동 구성
    implementation 'org.springframework.session:spring-session-data-redis'
    
    // Spring Data Redis: Redis 클라이언트를 추상화하고 Spring의 데이터 접근 계층과 통합합
    // 내부적으로 Lettuce(기본) 또는 Jedis 클라이언트를 사용합니다.
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
```

### 2. SecurityFilterChain
```
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
  
  // 필터에서 사용할 화이트리스트
  public static final String[] SECURITY_EXCLUDE_PATHS = {
      "/public/**", "/api/swagger-ui/**", "/swagger-ui/**", "/swagger-ui.html",
      "/api/v3/api-docs/**", "/v3/api-docs/**", "/favicon.ico", "/actuator/**",
      "/swagger-resources/**", "/external/**", "/api/auth/**"
  };
  
  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
          // CSRF 공격 방어 기능을 끄는 설정
          // 보통 Spring은 Nginx를 거쳐서 오기떄문에 csrf를 nginx에서 잡는 편
          .csrf(AbstractHttpConfigurer::disable) 
          
          // 인증 정보(SecurityContext)를 어디에 저장하고, 어디서 꺼낼지 결정하는 설정
          // 부가 bean
          .securityContext(context -> context
              .securityContextRepository(securityContextRepository())
          )
          
          // 세션을 어떻게 만들고, 몇 개까지 허용할지 제어하는 설정
          .sessionManagement(session -> session
              .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED) // 필요시 생성해서 사용(총 4종)
              .maximumSessions(1) // 한 계정당 동시 로그인 가능한 세션 수 제한
              .maxSessionsPreventsLogin(false) 
              // 동시 로그인 발생 시 어떻게 처리할지 결정 (false -> 신규 로그인 허용, 기존은 강제 로그아웃)
          )
          
          // 인증이 필요한 url과 아닌 url 구분
          .authorizeHttpRequests(auth -> auth
              .requestMatchers(SECURITY_EXCLUDE_PATHS).permitAll() // 인증 필요 X
              .requestMatchers("/api/**").hasRole("USER") // 인증 필요 X
              .anyRequest().authenticated() // 그외에는 인증 필요 O
          )
          
           // 기본 로그인 페이지 + 폼 기반 로그인 기능
          .formLogin(AbstractHttpConfigurer::disable)
          
           // HTTP Basic 인증 방식
          .httpBasic(AbstractHttpConfigurer::disable)
          
          // 인증 실패(로그인 안 된 상태)일 때 어떤 응답을 줄지 직접 정의하는 설정
          .exceptionHandling(ex -> ex
              .authenticationEntryPoint((request, response, authException) -> {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // 상태 설정 
                response.setContentType("application/json;charset=UTF-8"); // 컨텐츠 설정
                
                // 기존에 사용하던 ApiResponse 객체 
                ApiResponse<Void> errorResponse = ApiResponse.<Void>builder()
                    .error(ApiResponse.Error.of("UNAUTHORIZED", "Authentication required"))
                    .build();
                
                // 리스폰 처리
                response.getWriter().write(objectMapper.writeValueAsString(errorResponse));
              })
          );
      
      return http.build();
  }
  ~~~~~~~~~~~~
  @Bean
  public SecurityContextRepository securityContextRepository() {
  // httpSession에 저장함
  // httpsession이 redis로 잡혀있으면 redis로 감
  return new HttpSessionSecurityContextRepository();
  }
  ~~~~~~~~
}
```

### 3. passwordEncoder
#### 1. passwordEncoder
- 스프링 시큐리티가 패스워드를 해석하는 방식
- Bean으로 등록해두면 Spring Security가 자동으로 사용

#### 2. 종류 

| 이름                        | 특징             | 보안 수준 | 사용 여부      |
| ------------------------- | -------------- | ----- | ---------- |
| BCryptPasswordEncoder     | salt 자동, 느린 해싱 | 높음    |  기본        |
| Pbkdf2PasswordEncoder     | 반복 해싱, 설정 가능   | 높음    |  선택        |
| SCryptPasswordEncoder     | 메모리 기반, 강력     | 매우 높음 |  제한적       |
| DelegatingPasswordEncoder | 여러 알고리즘 혼용     | 상황별   |  확장용       |
| NoOpPasswordEncoder       | 평문 저장          | 없음    |  금지        |
| StandardPasswordEncoder   | SHA-256 기반     | 낮음    | deprecated |

#### 3. 소스
```
@Bean
public PasswordEncoder passwordEncoder() {
  return new BCryptPasswordEncoder();
}
```

## 4. 사용자 조회 & 인증 정보 객체
### 1. 사용자 조회
#### 1. CustomUserDetailsService (사용자 조회)
- 로그인 시 사용자 정보를 DB에서 조회
  - 특정한 사용자를 조회하는 방식
  - DB를 쓴다면 이 패턴으로 구현해서 “우리 서비스 방식으로 사용자 조회”
- 조회한 데이터를 UserDetails 형태로 변환해서 반환
  - 스프링 시큐리티는 내부적으로 UserDetailsService를 통해 사용자 정보를 가져옴

#### 2. 소스
```
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

  private final UserRepository userRepository;

  @Override
  public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
    User user = userRepository.findByEmail(email)
        .orElseThrow(() -> new UsernameNotFoundException("User not found with email: " + email));

    return CustomUserDetails.from(user); //CustomUSerDtails로 반환
  }
}
```

### 2. 인증 정보 객체 (Security용 Wrapper) 
#### 1. 소스
 - DB에서 가져온 사용자 정보를 스프링 시큐리티가 이해할 수 있는 형태로 감싸는 객체
 - User 엔티티 → Security 전용 객체로 변환
 - 스프링 시큐리티는 내부적으로 UserDetails 인터페이스를 기준으로 동작

#### 2. 소스
```
@Getter
public class CustomUserDetails implements UserDetails, Serializable {

  private static final long serialVersionUID = 1L;

  private final Long userId;
  private final String email;
  private final String password;

  public CustomUserDetails(Long userId, String email, String password) {
    this.userId = userId;
    this.email = email;
    this.password = password;
  }
 
  public static CustomUserDetails from(User user) {
    return new CustomUserDetails(user.getId(), 7user.getEmail(), user.getPassword());
  }

  @Override
  public Collection<? extends GrantedAuthority> getAuthorities() {
    // 이 유저가 가진 권한(Role) 정보를 시큐리티에 알려줌
    // 예시 1: 단일 권한만 부여
    return Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER"));
    
    // 예시 2: 여러 권한 부여 가능
    // return Arrays.asList(
    //     new SimpleGrantedAuthority("ROLE_USER"),
    //     new SimpleGrantedAuthority("ROLE_ADMIN")
    // );
  }

  @Override
  public String getPassword() {
    // 비밀번호 검증에 사용됨
    return password;
  }

  @Override
  public String getUsername() {
    // email = username 역할
    // 시큐리티는 email을 “아이디”처럼 사용
    // 이미 한번 필터된거라 별도 작업 X
    return email; 
  }

  @Override
  public boolean isAccountNonExpired() {
    // 계정 만료 여부 체크
    // 예시: DB에서 accountExpiryDate를 가져와서 현재 날짜와 비교
    // return accountExpiryDate.isAfter(LocalDate.now());
    return true; // 현재는 만료 없음으로 처리
  }

  @Override
  public boolean isAccountNonLocked() {
    // 계정 잠금 여부 체크
    // 예시: DB에서 failedLoginAttempts 확인
    // return failedLoginAttempts < 5;
    return true; // 현재는 잠금 없음으로 처리
  }

  @Override
  public boolean isCredentialsNonExpired() {
    // 비밀번호 만료 여부 체크
    // 예시: DB에서 passwordLastChangedDate 확인
    // return passwordLastChangedDate.plusMonths(3).isAfter(LocalDate.now());
    return true; // 현재는 비밀번호 만료 없음으로 처리
  }

  @Override
  public boolean isEnabled() {
    // 계정 활성화 여부 체크
    // 예시: DB에서 active 컬럼 확인
    // return active == true;
    return true; // 현재는 항상 활성으로 처리
  }
}
```

## 5. AuthService
### 1. 유저등록
```
  private final UserRepository userRepository;
  private final PasswordEncoder passwordEncoder;
  private final AuthenticationManager authenticationManager;
  private final SecurityContextRepository securityContextRepository;


  @Transactional
  public void registration(RegistrationRequest request) {
    userRepository.save(User.builder()
        .name(request.getName())
        .phone(request.getPhone())
        .email(request.getEmail())
        .password(passwordEncoder.encode(request.getPassword())) //암호화처리
        .gender(request.getGender())
        .build());
  }
```

### 2. 로그인 / 인증정보 저장
```
public LoginResponse login(LoginRequest loginRequest, HttpServletRequest request,
      HttpServletResponse response) {

    // 입력받은 이메일과 비밀번호로 AuthenticationToken 생성 후 인증 시도
    // 인증 실패 시 예외로 튕겨나오고, 로그인 성공 로직은 실행되지 않음
    Authentication authentication = authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(
            loginRequest.getEmail(),      // 로그인 폼에서 입력받은 이메일
            loginRequest.getPassword()    // 로그인 폼에서 입력받은 비밀번호
        )
    );

    // 비어있는 컨테이너(SecurityContext) 생성
    SecurityContext context = SecurityContextHolder.createEmptyContext();

    // 인증 정보를 저장함.
    context.setAuthentication(authentication);

    // 현재 쓰레드에 SecurityContext 세팅
    // 해당 쓰레드에서는 어디에서도 사용 가능 해짐.(필수X)
    SecurityContextHolder.setContext(context);

    // 외부 저장소에 저장함.
    // 저장해놔야 다른 스레드나 요청에서 시큐리티가 내부 검토하고 바로 통과 / 검증필요 작업을 함. 
    securityContextRepository.saveContext(context, request, response);

    // 인증된 유저 정보를 CustomUserDetails로 가져오기(Response용)
    CustomUserDetails userDetails = (CustomUserDetails) authentication.getPrincipal();

    // 로그인 성공 응답 생성
    return LoginResponse.builder()
        .userId(userDetails.getUserId())   // DB User id
        .email(userDetails.getEmail())     // 로그인 이메일
        .build();
}
```
