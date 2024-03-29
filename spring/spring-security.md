# Introduction

이 글에서는 Spring Boot + JWT + Security 를 사용해서 회원가입/로그인 로직을 구현했습니다.

JWT 와 Spring Security 코드는 [인프런 Spring Boot JWT Tutorial (정은구)](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-jwt#) 강의를 수강하면서 만들고 제 스타일에 맞게 수정했습니다.

오로지 Security 와 인증 로직에 초점을 맞추기 위해 불필요한 코드는 제거했습니다.

구현하고자 하는 전체 로직은 다음과 같습니다.

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/security-1.png?raw=true)

<br>

**(2022. 11. 22 추가)**

- Spring Security 의 `WebSecurityConfigurerAdapter` 가 deprecated 되어 이에 맞게 `SecurityConfig` 파일의 수정이 있었습니다.
  - 관련 내용은 [Spring Blog - Spring Security without the WebSecurityConfigurerAdapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter) 을 참고했습니다.
- Spring version up: 2.4.3 -> 2.7.5

<br>

# 1. JWT

JWT 에 관련된 글은 따로 작성했기 때문에 [링크](https://github.com/ParkJiwoon/PrivateStudy/blob/master/web/jwt.md)로 대체합니다.

<br>

# 2. Spring Security

Spring Security 는 사용자 정보 (ID/PW) 검증 및 유저 정보 관리 등을 쉽게 사용할 수 있도록 제공합니다.

JWT 와 같이 소개되는 경우가 많은데 스프링 시큐리티는 원래 세션 기반 인증을 사용하기 때문에 JWT 와 별개로 생각해야 합니다.

사용자 로그인 뿐만 아니라 보안 관련된 여러가지 설정들도 제공하기 때문에 실제 업무에서 사용한다면 꼭 개념을 미리 학습하고 사용하는 것을 권장합니다.

여기서는 겉핥기식으로 필요한 시큐리티 정보만 세팅하고 어떤식으로 동작하는 지만 파악합니다.

- User Role 을 꼭 설정해야 하나요?
    - Spring Security 자체에서 내부적으로 사용하는 것 같음
    - `ROLE_USER` 처럼 정확히 형식을 지켜줘야 함

<br>

```java
// build.gradle
plugins {
	id 'org.springframework.boot' version '2.7.5'
	id 'io.spring.dependency-management' version '1.0.15.RELEASE'
	id 'java'
}

group = 'com.tutorial'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

        // security 관련 의존성
	implementation 'org.springframework.boot:spring-boot-starter-security'

	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.security:spring-security-test'

        // jwt 관련 의존성
	compile group: 'io.jsonwebtoken', name: 'jjwt-api', version: '0.11.2'
	runtime group: 'io.jsonwebtoken', name: 'jjwt-impl', version: '0.11.2'
	runtime group: 'io.jsonwebtoken', name: 'jjwt-jackson', version: '0.11.2'
}

test {
	useJUnitPlatform()
}
```

<br>

# 3. Member 도메인 설계

시큐리티 설정을 테스트하기 위한 기본적인 사용자 도메인을 만듭니다.

시큐리티 자체적으로 UserDetails 의 구현체인 User 를 사용하기 때문에 헷갈리지 않도록 Account 또는 Member 로 이름 짓는게 좋습니다. (개인적인 생각)

- Member 도메인
    - Member
    - MemberRepository
    - MemberService
    - MemberController
    - `application.yml`: h2 database 설정과 jwt secret key 설정

<br>

## 3.1. Member

```java
@Getter
@NoArgsConstructor
@Table(name = "member")
@Entity
public class Member {

    @Id
    @Column(name = "member_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    private String password;

    @Enumerated(EnumType.STRING)
    private Authority authority;

    @Builder
    public Member(String email, String password, Authority authority) {
        this.email = email;
        this.password = password;
        this.authority = authority;
    }
}
```

- 최소한의 정보만을 갖고 있는 Member Entity 입니다.

<br>

```java
public enum Authority {
    ROLE_USER, ROLE_ADMIN
}
```

- 권한은 Enum 클래스로 만들었습니다.

<br>

## 3.2. MemberRepository

```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    Optional<Member> findByEmail(String email);
    boolean existsByEmail(String email);
}
```

- 마찬가지로 최소한의 쿼리만 갖고있습니다.
- Email 을 Login ID 로 갖고 있기 때문에 `findByEmail` 와 중복 가입 방지를 위한 `existsByEmail` 만 추가합니다.

<br>

## 3.3. MemberService

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberService {
    private final MemberRepository memberRepository;

    public MemberResponseDto findMemberInfoById(Long memberId) {
        return memberRepository.findById(memberId)
                .map(MemberResponseDto::of)
                .orElseThrow(() -> new RuntimeException("로그인 유저 정보가 없습니다."));
    }

    public MemberResponseDto findMemberInfoByEmail(String email) {
        return memberRepository.findByEmail(email)
                .map(MemberResponseDto::of)
                .orElseThrow(() -> new RuntimeException("유저 정보가 없습니다."));
    }
}
```

- `memberId` 와 `email` 로 회원 정보를 가져오는 Service 입니다.

<br>

## 3.4. MemberController

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/member")
public class MemberController {
    private final MemberService memberService;

    @GetMapping("/me")
    public ResponseEntity<MemberResponseDto> findMemberInfoById() {
        return ResponseEntity.ok(memberService.findMemberInfoById(SecurityUtil.getCurrentMemberId()));
    }

    @GetMapping("/{email}")
    public ResponseEntity<MemberResponseDto> findMemberInfoByEmail(@PathVariable String email) {
        return ResponseEntity.ok(memberService.findMemberInfoByEmail(email));
    }
}
```

- 내 정보를 가져올 때는 `SecurityUtil.getCurrentMemberId()` 를 사용합니다.
- API 요청이 들어오면 필터에서 Access Token 을 복호화 해서 유저 정보를 꺼내 `SecurityContext` 라는 곳에 저장합니다.
- `SecurityContext` 에 저장된 유저 정보는 전역으로 어디서든 꺼낼 수 있습니다.
- `SecurityUtil` 클래스에서는 유저 정보에서 Member ID 만 반환하는 메소드가 정의되어 있습니다.
- `MemberService` 가 아닌 `MemberController` 에서 사용하는 이유는 `MemberService` 테스트가 `SecurityContext` 에 의존적이지 않게 하기 위해서입니다.

<br>

## 3.5. application.yml

```yaml
spring:

  h2:
    console:
      enabled: true

  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:

  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: create-drop
    properties:
      hibernate:
        format_sql: true
        show_sql: true

logging:
  level:
    com.tutorial: debug

# HS512 알고리즘을 사용할 것이기 때문에 512bit, 즉 64byte 이상의 secret key를 사용해야 한다.
# Secret 값은 특정 문자열을 Base64 로 인코딩한 값 사용 (아래 명령어를 터미널에 쳐보면 그대로 나옴)
# $ echo 'spring-boot-security-jwt-tutorial-jiwoon-spring-boot-security-jwt-tutorial' | base64
jwt:
  secret: c3ByaW5nLWJvb3Qtc2VjdXJpdHktand0LXR1dG9yaWFsLWppd29vbi1zcHJpbmctYm9vdC1zZWN1cml0eS1qd3QtdHV0b3JpYWwK
```

- H2 Database 를 사용하기 위한 기본 설정과 JWT 시크릿 키를 설정해둡니다.
- 시크릿 키도 원래는 깃헙에 올라가지 않게 별도로 보관하는 것이 안전합니다.

<br>

# 4. JWT 와 Security 설정

- JWT 관련
    - `TokenProvider`: 유저 정보로 JWT 토큰을 만들거나 토큰을 바탕으로 유저 정보를 가져옴
    - `JwtFilter`: Spring Request 앞단에 붙일 Custom Filter
- Spring Security 관련
    - `JwtSecurityConfig`: JWT Filter 를 추가
    - `JwtAccessDeniedHandler`: 접근 권한 없을 때 403 에러
    - `JwtAuthenticationEntryPoint`: 인증 정보 없을 때 401 에러
    - `SecurityConfig`: 스프링 시큐리티에 필요한 설정
    - `SecurityUtil`: SecurityContext 에서 전역으로 유저 정보를 제공하는 유틸 클래스

<br>

## 4.1. TokenProvider

```java
@Slf4j
@Component
public class TokenProvider {

    private static final String AUTHORITIES_KEY = "auth";
    private static final String BEARER_TYPE = "Bearer";
    private static final long ACCESS_TOKEN_EXPIRE_TIME = 1000 * 60 * 30;            // 30분
    private static final long REFRESH_TOKEN_EXPIRE_TIME = 1000 * 60 * 60 * 24 * 7;  // 7일

    private final Key key;

    public TokenProvider(@Value("${jwt.secret}") String secretKey) {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        this.key = Keys.hmacShaKeyFor(keyBytes);
    }

    public TokenDto generateTokenDto(Authentication authentication) {
        // 권한들 가져오기
        String authorities = authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(","));

        long now = (new Date()).getTime();

        // Access Token 생성
        Date accessTokenExpiresIn = new Date(now + ACCESS_TOKEN_EXPIRE_TIME);
        String accessToken = Jwts.builder()
                .setSubject(authentication.getName())       // payload "sub": "name"
                .claim(AUTHORITIES_KEY, authorities)        // payload "auth": "ROLE_USER"
                .setExpiration(accessTokenExpiresIn)        // payload "exp": 1516239022 (예시)
                .signWith(key, SignatureAlgorithm.HS512)    // header "alg": "HS512"
                .compact();

        // Refresh Token 생성
        String refreshToken = Jwts.builder()
                .setExpiration(new Date(now + REFRESH_TOKEN_EXPIRE_TIME))
                .signWith(key, SignatureAlgorithm.HS512)
                .compact();

        return TokenDto.builder()
                .grantType(BEARER_TYPE)
                .accessToken(accessToken)
                .accessTokenExpiresIn(accessTokenExpiresIn.getTime())
                .refreshToken(refreshToken)
                .build();
    }

    public Authentication getAuthentication(String accessToken) {
        // 토큰 복호화
        Claims claims = parseClaims(accessToken);

        if (claims.get(AUTHORITIES_KEY) == null) {
            throw new RuntimeException("권한 정보가 없는 토큰입니다.");
        }

        // 클레임에서 권한 정보 가져오기
        Collection<? extends GrantedAuthority> authorities =
                Arrays.stream(claims.get(AUTHORITIES_KEY).toString().split(","))
                        .map(SimpleGrantedAuthority::new)
                        .collect(Collectors.toList());

        // UserDetails 객체를 만들어서 Authentication 리턴
        UserDetails principal = new User(claims.getSubject(), "", authorities);

        return new UsernamePasswordAuthenticationToken(principal, "", authorities);
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token);
            return true;
        } catch (io.jsonwebtoken.security.SecurityException | MalformedJwtException e) {
            log.info("잘못된 JWT 서명입니다.");
        } catch (ExpiredJwtException e) {
            log.info("만료된 JWT 토큰입니다.");
        } catch (UnsupportedJwtException e) {
            log.info("지원되지 않는 JWT 토큰입니다.");
        } catch (IllegalArgumentException e) {
            log.info("JWT 토큰이 잘못되었습니다.");
        }
        return false;
    }

    private Claims parseClaims(String accessToken) {
        try {
            return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(accessToken).getBody();
        } catch (ExpiredJwtException e) {
            return e.getClaims();
        }
    }
}
```

- JWT 토큰에 관련된 암호화, 복호화, 검증 로직은 다 이곳에서 이루어집니다.
- 생성자
    - `application.yml` 에 정의해놓은 `jwt.secret` 값을 가져와서 JWT 를 만들 때 사용하는 암호화 키값을 생성합니다.
- `generateTokenDto`
    - 유저 정보를 넘겨받아서 Access Token 과 Refresh Token 을 생성합니다.
    - 넘겨받은 유저 정보의 `authentication.getName()` 메소드가 `username` 을 가져옵니다.
    - 저는 `username` 으로 Member ID 를 저장했기 때문에 해당 값이 설정될 겁니다.
    - Access Token 에는 유저와 권한 정보를 담고 Refresh Token 에는 아무 정보도 담지 않습니다.
- `getAuthentication`
    - JWT 토큰을 복호화하여 토큰에 들어 있는 정보를 꺼냅니다.
    - Access Token 에만 유저 정보를 담기 때문에 명시적으로 `accessToken` 을 파라미터로 받게 했습니다.
    - Refresh Token 에는 아무런 정보 없이 만료일자만 담았습니다.
    - `UserDetails` 객체를 생생성해서 `UsernamePasswordAuthenticationToken` 형태로 리턴하는데 `SecurityContext` 를 사용하기 위한 절차라고 생각하면 됩니다..
    - 사실 좀 불필요한 절차라고 생각되지만 `SecurityContext` 가 `Authentication` 객체를 저장하기 때문에 어쩔수 없습니다.
    - `parseClaims` 메소드는 만료된 토큰이어도 정보를 꺼내기 위해서 따로 분리했습니다.
- `validateToken`
    - 토큰 정보를 검증합니다.
    - `Jwts` 모듈이 알아서 Exception 을 던져줍니다.

<br>

## 4.2. JwtFilter

```java
@RequiredArgsConstructor
public class JwtFilter extends OncePerRequestFilter {

    public static final String AUTHORIZATION_HEADER = "Authorization";
    public static final String BEARER_PREFIX = "Bearer ";

    private final TokenProvider tokenProvider;

    // 실제 필터링 로직은 doFilterInternal 에 들어감
    // JWT 토큰의 인증 정보를 현재 쓰레드의 SecurityContext 에 저장하는 역할 수행
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws IOException, ServletException {

        // 1. Request Header 에서 토큰을 꺼냄
        String jwt = resolveToken(request);

        // 2. validateToken 으로 토큰 유효성 검사
        // 정상 토큰이면 해당 토큰으로 Authentication 을 가져와서 SecurityContext 에 저장
        if (StringUtils.hasText(jwt) && tokenProvider.validateToken(jwt)) {
            Authentication authentication = tokenProvider.getAuthentication(jwt);
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }

        filterChain.doFilter(request, response);
    }

    // Request Header 에서 토큰 정보를 꺼내오기
    private String resolveToken(HttpServletRequest request) {
        String bearerToken = request.getHeader(AUTHORIZATION_HEADER);
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith(BEARER_PREFIX)) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

- `OncePerRequestFilter` 인터페이스를 구현하기 때문에 요청 받을 때 단 한번만 실행됩니다.
- `doFilterInternal`
    - 실제 필터링 로직을 수행하는 곳입니다.
    - Request Header 에서 Access Token 을 꺼내고 여러가지 검사 후 유저 정보를 꺼내서 `SecurityContext` 에 저장합니다.
    - 가입/로그인/재발급을 제외한 모든 Request 요청은 이 필터를 거치기 때문에 토큰 정보가 없거나 유효하지 않으면 정상적으로 수행되지 않습니다.
    - 그리고 요청이 정상적으로 Controller 까지 도착했다면 `SecurityContext` 에 Member ID 가 존재한다는 것이 보장됩니다.
    - 대신 직접 DB 를 조회한 것이 아니라 Access Token 에 있는 Member ID 를 꺼낸 거라서, 탈퇴로 인해 Member ID 가 DB 에 없는 경우 등 예외 상황은 Service 단에서 고려해야 합니다.

<br>

## 4.3. JwtSecurityConfig

```java
// 직접 만든 TokenProvider 와 JwtFilter 를 SecurityConfig 에 적용할 때 사용
@RequiredArgsConstructor
public class JwtSecurityConfig extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {
    private final TokenProvider tokenProvider;

    // TokenProvider 를 주입받아서 JwtFilter 를 통해 Security 로직에 필터를 등록
    @Override
    public void configure(HttpSecurity http) {
        JwtFilter customFilter = new JwtFilter(tokenProvider);
        http.addFilterBefore(customFilter, UsernamePasswordAuthenticationFilter.class);
    }
}
```

- `SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity>` 인터페이스를 구현하는 구현체입니다.
- 여기서 직접 만든 JwtFilter 를 Security Filter 앞에 추가합니다.

<br>

## 4.4. JwtAuthenticationEntryPoint

```java
@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException {
        // 유효한 자격증명을 제공하지 않고 접근하려 할때 401
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED);
    }
}
```

- 유저 정보 없이 접근하면 `SC_UNAUTHORIZED (401)` 응답을 내려줍니다.

<br>

## 4.5. JwtAccessDeniedHandler

```java
@Component
public class JwtAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        // 필요한 권한이 없이 접근하려 할때 403
        response.sendError(HttpServletResponse.SC_FORBIDDEN);
    }
}
```

- 유저 정보는 있으나 자원에 접근할 수 있는 권한이 없는 경우 `SC_FORBIDDEN (403)` 응답을 내려줍니다.

<br>

## 4.6. SecurityConfig

```java
@Configuration
@RequiredArgsConstructor
public class SecurityConfig {
    private final TokenProvider tokenProvider;
    private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    private final JwtAccessDeniedHandler jwtAccessDeniedHandler;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    // h2 database 테스트가 원활하도록 관련 API 들은 전부 무시
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring()
                .antMatchers("/h2-console/**", "/favicon.ico");
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
            // CSRF 설정 Disable
        http.csrf().disable()

            // exception handling 할 때 우리가 만든 클래스를 추가
            .exceptionHandling()
            .authenticationEntryPoint(jwtAuthenticationEntryPoint)
            .accessDeniedHandler(jwtAccessDeniedHandler)

            .and()
            .headers()
            .frameOptions()
            .sameOrigin()

            // 시큐리티는 기본적으로 세션을 사용
            // 여기서는 세션을 사용하지 않기 때문에 세션 설정을 Stateless 로 설정
            .and()
            .sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)

            // 로그인, 회원가입 API 는 토큰이 없는 상태에서 요청이 들어오기 때문에 permitAll 설정
            .and()
            .authorizeRequests()
            .antMatchers("/auth/**").permitAll()
            .anyRequest().authenticated()   // 나머지 API 는 전부 인증 필요

            // JwtFilter 를 addFilterBefore 로 등록했던 JwtSecurityConfig 클래스를 적용
            .and()
            .apply(new JwtSecurityConfig(tokenProvider));

        return http.build();
    }
}
```

- Spring Security 의 가장 기본적인 설정이며 JWT 를 사용하지 않더라도 이 설정은 기본으로 들어갑니다.
- 각 설정에 대한 설명은 주석을 확인하면 됩니다.
- **(2022. 11. 23 추가)** Spring Security 의 `WebSecurityConfigurerAdapter` 가 deprecated 되어 이에 맞게 `SecurityConfig` 파일의 수정이 있었습니다.
  - 관련 내용은 [Spring Blog - Spring Security without the WebSecurityConfigurerAdapter](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter) 을 참고했습니다.

<br>

## 4.7. SecurityUtil

```java
@Slf4j
public class SecurityUtil {

    private SecurityUtil() { }

    // SecurityContext 에 유저 정보가 저장되는 시점
    // Request 가 들어올 때 JwtFilter 의 doFilter 에서 저장
    public static Long getCurrentMemberId() {
        final Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (authentication == null || authentication.getName() == null) {
            throw  new RuntimeException("Security Context 에 인증 정보가 없습니다.");
        }

        return Long.parseLong(authentication.getName());
    }
}
```

- `JwtFilter` 에서 `SecurityContext` 에 세팅한 유저 정보를 꺼냅니다.
- 저는 무조건 `memberId` 를 저장하게 했으므로 꺼내서 Long 타입으로 파싱하여 반환합니다.
- `SecurityContext` 는 `ThreadLocal` 에 사용자의 정보를 저장합니다.

<br>

# 5. Refresh Token 저장소

Access Token 과 Refresh Token 을 함께 사용하기 때문에 저장이 필요합니다.

보통은 Token 이 만료될 때 자동으로 삭제 처리 하기 위해 Redis 를 많이 사용하지만, 귀찮으니 일단 임시로 RDB 에 저장하는 방식으로 구현했습니다.

만약 지금 예제처럼 RDB 를 저장소로 사용한다면 배치 작업을 통해 만료된 토큰들을 삭제해주는 작업이 필요합니다.

<br>

## 5.1. RefreshToken

```java
@Getter
@NoArgsConstructor
@Table(name = "refresh_token")
@Entity
public class RefreshToken {

    @Id
    @Column(name = "rt_key")
    private String key;

    @Column(name = "rt_value")
    private String value;

    @Builder
    public RefreshToken(String key, String value) {
        this.key = key;
        this.value = value;
    }

    public RefreshToken updateValue(String token) {
        this.value = token;
        return this;
    }
}
```

- key 에는 Member ID 값이 들어갑니다.
- value 에는 Refresh Token String 이 들어갑니다.
- 위에서 언급한대로 RDB 로 구현하게 된다면 생성/수정 시간 컬럼을 추가하여 배치 작업으로 만료된 토큰들을 삭제해주어야 합니다.

<br>

## 5.2. RefreshTokenRepository

```java
@Repository
public interface RefreshTokenRepository extends JpaRepository<RefreshToken, Long> {
    Optional<RefreshToken> findByKey(String key);
}
```

- Member ID 값으로 토큰을 가져오기 위해 `findByKey` 만 추가했습니다.

<br>

# 6. 사용자 인증 과정

지금까지 스프링 시큐리티와 JWT 를 사용하기 위한 설정들을 전부 끝냈습니다.

지금부터는 실제로 사용자 로그인 요청이 들어왔을 때 인증 처리 후에 JWT 토큰을 발급하는 과정을 알아봅니다.

- AuthController
- AuthService
- CustomUserDetailsService

<br>

## 6.1. AuthController

```java
@RestController
@RequestMapping("/auth")
@RequiredArgsConstructor
public class AuthController {
    private final AuthService authService;

    @PostMapping("/signup")
    public ResponseEntity<MemberResponseDto> signup(@RequestBody MemberRequestDto memberRequestDto) {
        return ResponseEntity.ok(authService.signup(memberRequestDto));
    }

    @PostMapping("/login")
    public ResponseEntity<TokenDto> login(@RequestBody MemberRequestDto memberRequestDto) {
        return ResponseEntity.ok(authService.login(memberRequestDto));
    }

    @PostMapping("/reissue")
    public ResponseEntity<TokenDto> reissue(@RequestBody TokenRequestDto tokenRequestDto) {
        return ResponseEntity.ok(authService.reissue(tokenRequestDto));
    }
}
```

- 회원가입 / 로그인 / 재발급 을 처리하는 API 입니다.
- `SecurityConfig` 에서 `/auth/**` 요청은 전부 허용했기 때문에 토큰 검증 로직을 타지 않습니다.
- `MemberRequestDto` 에는 사용자가 로그인 시도한 ID / PW String 이 존재합니다.
- `TokenRequestDto` 에는 재발급을 위한 AccessToken / RefreshToken String 이 존재합니다.

<br>

## 6.2. AuthService

```java
@Service
@RequiredArgsConstructor
public class AuthService {
    private final AuthenticationManagerBuilder authenticationManagerBuilder;
    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;
    private final TokenProvider tokenProvider;
    private final RefreshTokenRepository refreshTokenRepository;

    @Transactional
    public MemberResponseDto signup(MemberRequestDto memberRequestDto) {
        if (memberRepository.existsByEmail(memberRequestDto.getEmail())) {
            throw new RuntimeException("이미 가입되어 있는 유저입니다");
        }

        Member member = memberRequestDto.toMember(passwordEncoder);
        return MemberResponseDto.of(memberRepository.save(member));
    }

    @Transactional
    public TokenDto login(MemberRequestDto memberRequestDto) {
        // 1. Login ID/PW 를 기반으로 AuthenticationToken 생성
        UsernamePasswordAuthenticationToken authenticationToken = memberRequestDto.toAuthentication();

        // 2. 실제로 검증 (사용자 비밀번호 체크) 이 이루어지는 부분
        //    authenticate 메서드가 실행이 될 때 CustomUserDetailsService 에서 만들었던 loadUserByUsername 메서드가 실행됨
        Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);

        // 3. 인증 정보를 기반으로 JWT 토큰 생성
        TokenDto tokenDto = tokenProvider.generateTokenDto(authentication);

        // 4. RefreshToken 저장
        RefreshToken refreshToken = RefreshToken.builder()
                .key(authentication.getName())
                .value(tokenDto.getRefreshToken())
                .build();

        refreshTokenRepository.save(refreshToken);

        // 5. 토큰 발급
        return tokenDto;
    }

    @Transactional
    public TokenDto reissue(TokenRequestDto tokenRequestDto) {
        // 1. Refresh Token 검증
        if (!tokenProvider.validateToken(tokenRequestDto.getRefreshToken())) {
            throw new RuntimeException("Refresh Token 이 유효하지 않습니다.");
        }

        // 2. Access Token 에서 Member ID 가져오기
        Authentication authentication = tokenProvider.getAuthentication(tokenRequestDto.getAccessToken());

        // 3. 저장소에서 Member ID 를 기반으로 Refresh Token 값 가져옴
        RefreshToken refreshToken = refreshTokenRepository.findByKey(authentication.getName())
                .orElseThrow(() -> new RuntimeException("로그아웃 된 사용자입니다."));

        // 4. Refresh Token 일치하는지 검사
        if (!refreshToken.getValue().equals(tokenRequestDto.getRefreshToken())) {
            throw new RuntimeException("토큰의 유저 정보가 일치하지 않습니다.");
        }

        // 5. 새로운 토큰 생성
        TokenDto tokenDto = tokenProvider.generateTokenDto(authentication);

        // 6. 저장소 정보 업데이트
        RefreshToken newRefreshToken = refreshToken.updateValue(tokenDto.getRefreshToken());
        refreshTokenRepository.save(newRefreshToken);

        // 토큰 발급
        return tokenDto;
    }
}
```

<br>

#**회원가입 (signup)**

- 평범하게 유저 정보를 받아서 저장합니다.

<br>

#**로그인 (login)**

- `Authentication`
    - 사용자가 입력한 Login ID, PW 로 인증 정보 객체 `UsernamePasswordAuthenticationToken`를 생성합니다.
    - 아직 인증이 완료된 객체가 아니며 `AuthenticationManager` 에서 `authenticate` 메소드의 파라미터로 넘겨서 검증 후에 `Authentication` 를 받습니다.
- `AuthenticationManager`
    - 스프링 시큐리티에서 실제로 인증이 이루어지는 곳입니다.
    - `authenticate` 메소드 하나만 정의되어 있는 인터페이스며 위 코드에서는 Builder 에서  `UserDetails` 의 유저 정보가 서로 일치하는지 검사합니다.
    - 그런데 코드상으로는 전혀 구현된게 없는데 어떻게 된 걸까요?
    - 내부적으로 수행되는 검증 과정은 아래의 `CustomUserDetailsService` 클래스에서 다루겠습니다.
- 인증이 완료된 `authentication` 에는 Member ID 가 들어있습니다.
- 인증 객체를 바탕으로 Access Token + Refresh Token 을 생성합니다.
- Refresh Token 은 저장하고, 생성된 토큰 정보를 클라이언트에게 전달합니다.

<br>

#**재발급 (reissue)**

- Access Token + Refresh Token 을 Request Body 에 받아서 검증합니다.
- Refresh Token 의 만료 여부를 먼저 검사합니다.
- Access Token 을 복호화하여 유저 정보 (Member ID) 를 가져오고 저장소에 있는 Refresh Token 과 클라이언트가 전달한 Refresh Token 의 일치 여부를 검사합니다.
- 만약 일치한다면 로그인 했을 때와 동일하게 새로운 토큰을 생성해서 클라이언트에게 전달합니다.
- Refresh Token 은 재사용하지 못하게 저장소에서 값을 갱신해줍니다.

<br>

## 6.3. CustomUserDetailsService

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final MemberRepository memberRepository;

    @Override
    @Transactional
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return memberRepository.findByEmail(username)
                .map(this::createUserDetails)
                .orElseThrow(() -> new UsernameNotFoundException(username + " -> 데이터베이스에서 찾을 수 없습니다."));
    }

    // DB 에 User 값이 존재한다면 UserDetails 객체로 만들어서 리턴
    private UserDetails createUserDetails(Member member) {
        GrantedAuthority grantedAuthority = new SimpleGrantedAuthority(member.getAuthority().toString());

        return new User(
                String.valueOf(member.getId()),
                member.getPassword(),
                Collections.singleton(grantedAuthority)
        );
    }
}
```

- `UserDetailsService` 인터페이스를 구현한 클래스입니다.
- `loadUserByUsername` 메소드를 오버라이드 하는데 여기서 넘겨받은 `UserDetails` 와 `Authentication` 의 패스워드를 비교하고 검증하는 로직을 처리합니다.
- 물론 DB 에서 username 을 기반으로 값을 가져오기 때문에 아이디 존재 여부도 자동으로 검증 됩니다.
- `loadUserByUsername` 메소드를 어디서 호출하는지 내부를 타고 들어가봅니다.

<br>

### 6.3.1. CustomUserDetailsService

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/security-2.png?raw=true" width="80%">

`loadUserByUsername` 는 여러 곳에서 호출하고 있는데 이 중에서 `DaoAuthenticationProvider` 내부를 확인해봅니다.

<br>

### 6.3.2. DaoAuthenticationProvider

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/security-3.png?raw=true" width="80%">

- `username` 을 받아서 넘겨주는 `retrieveUser` 메소드 내부에서 호출합니다.
- 그럼 이 `retrieveUser` 는 어디서 호출할까요?

<br>

### 6.3.3. AbstractUserDetailsAuthenticationProvider

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/security-4.png?raw=true" width="80%">

- `DaoAuthenticationProvider` 의 부모 클래스인 `AbstractUserDetailsAuthenticationProvider` 에서 호출합니다.
- 코드를 쭉 보니 받아온 user 변수로 `additionalAuthenticationChecks` 메소드를 호출합니다.
- 메소드를 확인해보니 추상 클래스였고, `DaoAuthenticationProvider` 를 다시 확인해보니 오버라이드 해서 구현이 되어 있었습니다.

<br>

## 6.3.4. 다시 DaoAuthenticationProvider

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/security-5.png?raw=true" width="80%">

- **실제로 비밀번호 검증이 이루어지는 부분입니다 !**
- Request 로 받아서 만든 `authentication` 와 DB 에서 꺼낸 값인 `userDetails` 의 비밀번호를 비교합니다.
- DB 에 있는 값은 암호화된 값이고 사용자가 입력한 값은 raw  값이지만 `passwordEncoder` 가 알아서 비교해줍니다.
- 그래서 결국 비밀번호 검증이 시큐리티가 제공하는 클래스에서 이루어지는 것을 확인했는데 로그인 시에 사용되는 `AuthenticationManager` 와는 무슨 관계일까요?
- `AbstractUserDetailsAuthenticationProvider` 의 `authenticate` 를 어디에서 호출하는지 확인해봅니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/security-6.png?raw=true" width="80%">

- `AbstractUserDetailsAuthenticationProvider` 의 `authenticate` 는 단 한곳에서 호출합니다.

<br>

### 6.3.5. ProviderManager

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/security-7.png?raw=true" width="80%">

- 여기서도 `authenticate` 라는 메소드네요.
- `AuthenticationProvider` 라는 인터페이스에서 호출하는데요.
- 이름으로 짐작할 수 있듯이 `AbstractUserDetailsAuthenticationProvider` 의 상위 인터페이스입니다.
- 그리고  `ProviderManager.authenticate` 를 호출하는 곳을 확인해보니 드디어 찾을 수 있었습니다.

<br>

### 6.3.6. AuthService

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/security-8.png?raw=true" width="80%">

- `ProviderManager` 는 `AuthenticationManager` 의 구현체입니다.
- 지금까지의 탐구 과정을 역으로 다시 가보면 어떤 순서로 비밀번호 검증이 이루어지는 지 알 수 있습니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/security-9.png?raw=true">

1. `AuthService` (그림에서는 오타) 에서 `AuthenticationManagerBuilder` 주입 받음
2. AuthenticationManagerBuilder 에서 `AuthenticationManager` 를 구현한 `ProviderManager` 생성
3. ProviderManager 는 `AbstractUserDetailsAuthenticationProvider` 의 자식 클래스인 `DaoAuthenticationProvider` 를 주입받아서 호출
4. DaoAuthenticationProvider 의 `authenticate` 에서는 `retrieveUser` 로 DB 에 있는 사용자 정보를 가져오고 `additionalAuthenticationChecks` 로 비밀번호 비교
5. retrieveUser 내부에서 `UserDetailsService` 인터페이스를 직접 구현한 `CustomUserDetailsService` 클래스의 오버라이드 메소드인 `loadUserByUsername` 가 호출됨

<br>

# 7. API 호출 테스트

이제 서버를 띄우고 실제로 API 호출을 해봅니다.

API 요청은 인텔리제이에 있는 http Tool 을 사용했습니다.

<br>

## 7.1. 가입

```sh
# Request
POST http://localhost:8080/auth/signup
Content-Type: application/json

{
  "email": "test@test.net",
  "password": "1q2w3e4r"
}

# Response
{
  "email": "test@test.net"
}
```

<br>

## 7.2. 로그인

```sh
# Request
POST http://localhost:8080/auth/login
Content-Type: application/json

{
  "email": "test@test.net",
  "password": "1q2w3e4r"
}

# Response
{
  "grantType": "bearer",
  "accessToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIyIiwiYXV0aCI6IlJPTEVfVVNFUiIsImV4cCI6MTYxNTExNDI4MH0.43LvabP41Awhicy6YYAYHtDPnxNYpEygtE-DjLaDjNpAxZf01Nx4xE_dGk0V4jBpjwCgKVGKZIMyEeIppwzARQ",
  "refreshToken": "eyJhbGciOiJIUzUxMiJ9.eyJleHAiOjE2MTU3MTcyODB9.DKqk-EZVT0TJAvvHpSN8nClIHKq-k4KYMHpx-Ltf7V8OB6Og4D_dsYnr3Z4Rw7iR7ckv-ZWMyi5SkheESw-T0g",
  "accessTokenExpiresIn": 1615114280584
}
```

<br>

## 7.3. 일반 API 요청

```sh
# Request
GET http://localhost:8080/api/member/me
Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIyIiwiYXV0aCI6IlJPTEVfVVNFUiIsImV4cCI6MTYxNTExNDI4MH0.43LvabP41Awhicy6YYAYHtDPnxNYpEygtE-DjLaDjNpAxZf01Nx4xE_dGk0V4jBpjwCgKVGKZIMyEeIppwzARQ

# Response
{
  "email": "test@test.net"
}
```

- 사용자 요청 -> JwtFiletr (SecurityContext 세팅) -> Controller -> Service

<br>

## 7.4. 재발급

```sh
# Request
POST http://localhost:8080/auth/reissue
Content-Type: application/json

{
  "accessToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIyIiwiYXV0aCI6IlJPTEVfVVNFUiIsImV4cCI6MTYxNTExNDI4MH0.43LvabP41Awhicy6YYAYHtDPnxNYpEygtE-DjLaDjNpAxZf01Nx4xE_dGk0V4jBpjwCgKVGKZIMyEeIppwzARQ",
  "refreshToken": "eyJhbGciOiJIUzUxMiJ9.eyJleHAiOjE2MTU3MTcyODB9.DKqk-EZVT0TJAvvHpSN8nClIHKq-k4KYMHpx-Ltf7V8OB6Og4D_dsYnr3Z4Rw7iR7ckv-ZWMyi5SkheESw-T0g"
}

# Response
{
  "grantType": "bearer",
  "accessToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIyIiwiYXV0aCI6IlJPTEVfVVNFUiIsImV4cCI6MTYxNTExNDM2NX0.5VXa6Cht_DPEEGe7-BrElvsrs7qRXmVnkDdi4Lm3PxZ0vAgqFdirhe5RlE1D-Wc1zaUepBmGhhw-u-oP_-rbKQ",
  "refreshToken": "eyJhbGciOiJIUzUxMiJ9.eyJleHAiOjE2MTU3MTczNjV9.tZytWyCWkWIYitvT3pa8FSnxilBDMtSevUzKRFK21TGLITf2eLXEwNNS_Q7rylD9uUe3Rx9ZR2NVqE_ZNWxTqg",
  "accessTokenExpiresIn": 1615114365284
}
```

<br>

# Reference

- [Github 전체 코드](https://github.com/ParkJiwoon/practice-codes/tree/master/spring-security-jwt)
- [인프런 Spring Boot JWT Tutorial (정은구)](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-jwt#)

