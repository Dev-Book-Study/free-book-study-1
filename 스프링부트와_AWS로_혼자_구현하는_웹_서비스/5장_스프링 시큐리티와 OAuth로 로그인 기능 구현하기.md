# 📘 Spring Security 학습 문서 (2025, Java 21 + Spring Boot 3.4 기준)

## 🔗 참고 사항

- 원본 참고 도서는 **2019년 발행** → 기술 스택 구버전 가능성 있음
- 따라서 **최신 개발 환경(Java 21, Spring Boot 3.4)** 기준 병행 학습 결정

---

## 🔐 인증(Authentication) vs 인가(Authorization)

### 1)인증 (Authentication)

- **"너 누구야?"** 에 대한 질문에 답하는 과정
- 사용자가 **정말 그 사람인지**를 확인하는 절차
- 예를 들어:
    - 로그인을 하면서 **아이디/비밀번호**를 입력하거나
    - 구글/카카오 등의 **소셜 로그인**으로 본인임을 증명하거나
    - **토큰(JWT)**으로 이미 인증된 사용자인지 확인하는 것도 포함됨

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
String username = authentication.getName(); // 현재 로그인한 사용자의 이름
```

### 2)인가 (Authorization)

- **"이 리소스에 접근 가능한가?"** 에 대한 질문에 답하는 과정
- 인증된 사용자가 **어떤 권한(ROLE)** 을 가지고 있고, 그에 따라 **요청한 기능을 사용할 수 있는지** 판단
- 예를 들어:
    - 일반 사용자는 게시글 작성만 가능하지만
    - 관리자는 게시글 삭제, 사용자 관리 등 추가 권한이 있음
- Spring Security에서는 어노테이션이나 설정 파일을 통해 제어 가능

```java

@PreAuthorize("hasRole('ADMIN')")
public ResponseEntity<?> adminPage() {
    // 관리자만 접근 가능한 페이지
}
```

---

### 실무예시

- ADMIN 권한은 사용자 삭제, 시스템 설정 변경 등 민감한 기능을 수행할 수 있음
- USER 권한은 게시글 등록, 댓글 작성 등 일반적인 기능만 사용 가능
- REST API 서버에서는 ROLE 기반 접근 제어가 필수이며, 잘못된 권한 요청 시 403 Forbidden 반환
- ex)“이 사용자가 관리자 페이지에 접근하려고 합니다. 이 사람은 관리자인가요?” → 인가

---

## 🚧 인증 & 인가를 자체 구현시

### ✅ 구현 필요 기능

1. 비밀번호 암호화 저장 (`BCryptPasswordEncoder`)
2. 로그인 / 로그아웃 처리
3. 세션 기반 또는 JWT 기반 인증 구조 설계
4. 이메일 / 전화번호 인증 (회원가입, 비밀번호 찾기)
5. 비밀번호 재설정 및 변경
6. 사용자 상태 관리 (예: 탈퇴, 정지, 휴면 등)
7. 중복 로그인 방지 (기기 제한, 기존 세션 만료 처리 등)

---

### ⚖️ 예상 이슈 / 테스트 필요 내용

| 기능        | 예상 이슈                                   | 테스트 포인트                                                   |
|-----------|-----------------------------------------|-----------------------------------------------------------|
| 로그인       | 로그인 실패 시 에러 메시지가 너무 구체적일 경우 보안 위험       | 아이디/비밀번호 오류 시 **공통된 에러 메시지** 출력 확인 (`"로그인 정보를 확인하세요"` 등)  |
| 세션 저장     | 분산 서버 환경에서 세션이 공유되지 않음                  | Redis 기반 세션 저장 설정 후 **멀티 서버 환경에서 로그인 상태 유지 여부 확인**        |
| 이메일 인증    | 인증 메일이 수신되지 않거나 인증 링크가 만료됨              | Mock SMTP 환경에서 **메일 전송 확인**, 만료된 링크 접근 시 오류 처리 확인         |
| 비밀번호 재설정  | 재설정 토큰이 탈취될 경우 타인이 비밀번호를 변경할 수 있음       | 토큰의 **단일 사용 여부 확인**, **만료 시간 초과 시 재사용 불가 여부 테스트**         |
| 중복 로그인 방지 | 동일 계정으로 여러 기기에서 로그인 시 세션 충돌 또는 정책 누락 가능 | 기존 세션 만료 설정 테스트 (예: 마지막 로그인만 유지), **기기별 세션 구분 여부 확인**     |
| 사용자 상태 관리 | 탈퇴/정지된 사용자가 로그인을 시도할 수 있음               | 정지/탈퇴 계정으로 로그인 시 **접근 거부 처리** (`403`, `401`, 혹은 메시지 반환 등) |

---

## 🌐 OAuth2

### ✅ 개요

- **OAuth2**는 제3의 서비스(구글, 카카오, 네이버 등)가 사용자 인증을 대신 수행하고, 해당 사용자에 대한 최소한의 정보(이메일, 이름 등)를 애플리케이션에 제공하도록 허용하는 **권한 위임 인증
  프로토콜**
- 애플리케이션이 사용자의 **자격 증명(ID/PW)** 을 직접 처리하지 않아도 되기 때문에 보안성이 높음
- 대표적인 소셜 로그인(Google, Kakao, Naver 등)은 대부분 OAuth2 기반

### ✅ OAuth2의 장점

- ✨ **보안 강화**: 직접 비밀번호를 다루지 않음 (비밀번호 탈취 위험 최소화)
- ✨ **구현 간소화**: 자체 회원가입, 비밀번호 찾기, 이메일 인증 등을 생략 가능
- ✨ **접근성 향상**: 사용자가 기존 계정(Google 등)으로 빠르게 로그인 가능
- ✨ **추가 정보 활용 가능**: 프로필 사진, 이름, 이메일 등 추가 정보 획득 가능

### ⚠️ OAuth2의 단점

- ❗ **외부 서비스에 의존**: 소셜 API 변경 또는 장애가 로그인 기능에 직접 영향
- ❗ **회원 DB 분리 필요**: OAuth 로그인 사용자를 내부 시스템 사용자로 관리하려면 별도 매핑이 필요 (email 또는 providerId 기반)
- ❗ **권한 처리 한계**: 대부분 기본적인 사용자 정보만 제공 (직무, 부서 등은 따로 관리 필요)

---

### ⚙️ OAuth2 사용법 (Spring Boot 기준)

1. **의존성 추가**

```groovy
implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
```

2. **application.yml 설정 예시**

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: your-client-id
            client-secret: your-client-secret
            scope:
              - email
              - profile
        provider:
          google:
            authorization-uri: https://accounts.google.com/o/oauth2/auth
            token-uri: https://oauth2.googleapis.com/token
            user-info-uri: https://www.googleapis.com/oauth2/v3/userinfo
```

3. **CustomOAuth2UserService 구현 예시**

```java

@Service
public class CustomOAuth2UserService extends DefaultOAuth2UserService {

    @Override
    public OAuth2User loadUser(OAuth2UserRequest request) throws OAuth2AuthenticationException {
        OAuth2User oAuth2User = super.loadUser(request);

        // 예시: 구글에서 제공하는 이메일 정보 추출
        String email = oAuth2User.getAttribute("email");

        // 내부 DB 사용자와 매핑 (없으면 자동 가입 또는 가입 유도)
        // User user = userRepository.findByEmail(email).orElseGet(...)

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority("ROLE_USER")),
                oAuth2User.getAttributes(),
                "email" // 기본 식별자 key
        );
    }
}
```

> 🔍 실무에서는 OAuth2 로그인 이후 사용자 정보를 내부 계정과 매핑하여 **회원가입을 유도하거나 자동 생성 처리**하는 로직이 필요합니다.

---

### 🧪 OAuth2 로그인 테스트 예시 (MockMvc + Spring Security Test)

> `spring-security-test`를 사용하면 OAuth2 로그인 흐름을 MockMvc 환경에서 일부 시뮬레이션할 수 있습니다.

```java

@SpringBootTest
@AutoConfigureMockMvc
public class OAuth2LoginTest {

    @Autowired
    private MockMvc mockMvc;

    /**
     * 실제 로그인 엔드포인트로 접근하면 구글 OAuth 인증 URL로 리다이렉트되는지 테스트
     */
    @Test
    void shouldRedirectToGoogleOAuth2Authorization() throws Exception {
        mockMvc.perform(get("/oauth2/authorization/google"))
                .andExpect(status().is3xxRedirection())
                .andExpect(header().string("Location", startsWith("https://accounts.google.com/")));
    }

    /**
     * OAuth2 인증이 완료된 사용자가 로그인 상태로 특정 페이지를 접근하는 테스트
     * => oauth2Login()은 spring-security-test 에서 제공
     */
    @Test
    void shouldAccessWithAuthenticatedOAuth2User() throws Exception {
        mockMvc.perform(get("/secured/page").with(oauth2Login().attributes(attrs -> {
                    attrs.put("email", "user@example.com");
                    attrs.put("name", "홍길동");
                })))
                .andExpect(status().isOk())
                .andExpect(authenticated().withUsername("user@example.com"));
    }
}

```

---

## 💾 세션 저장 방식 비교

### 🏠 1. 톰캣 메모리 세션 (기본 방식)

- **설명**: 톰캣 또는 내장 서블릿 컨테이너의 메모리에 사용자 세션을 저장
- **장점**
    - 설정이 필요 없이 기본 동작
    - 가장 빠른 응답 속도 (네트워크 없이 로컬 메모리 접근)
- **단점**
    - 서버 재시작 시 세션 정보 유실
    - 서버 간 세션 공유 불가능 → 로드밸런서가 동일 서버로만 연결해야 함 (Sticky Session 필요)
- **사용 예시**: 단일 WAS(개발/테스트 환경 등), 세션 공유 필요 없는 경우

### 🗄️ 2. 데이터베이스 저장 (JDBC 기반)

- **설명**: 세션을 RDB에 저장 (예: MySQL, PostgreSQL 등)
- **장점**
    - 세션을 영구 저장 가능 (재시작에도 유지)
    - 서버 수평 확장 시에도 공유 가능
- **단점**
    - I/O 비용이 커서 성능 저하 가능
    - 데이터 정합성 관리 필요 (만료 세션 정리 등)
- **사용 예시**: 로그인 유지가 장시간 필요한 서비스, 트랜잭션 통합 관리가 필요한 경우

### ⚡ 3. 캐시 저장 (Redis 기반)

- **설명**: 세션을 Redis와 같은 인메모리 데이터 저장소에 저장
- **장점**
    - 빠른 속도 (메모리 기반 + 네트워크로도 빠름)
    - 분산 환경에서도 세션 공유 가능
    - TTL 설정으로 자동 만료 관리
- **단점**
    - Redis 서버 운영 필요 (추가 비용 및 장애 대응 필요)
    - 완전 영속성은 아님 (RDB/AOF 설정 필요)
- **사용 예시**: 대부분의 실무 서비스, 수평 확장 필요 환경, Kubernetes, AWS 환경 등

### 🏁 비교 요약

| 저장 방식      | 속도    | 세션 공유 | 복구 가능성       | 실무 적합성       |
|------------|-------|-------|--------------|--------------|
| 톰캣 메모리     | 매우 빠름 | ❌ 불가능 | ❌ 재시작 시 유실   | ❌ 단일 서버만 가능  |
| 데이터베이스     | 느림    | ✅ 가능  | ✅ 영속 가능      | △ (성능 이슈 주의) |
| Redis (캐시) | 빠름    | ✅ 가능  | ⭕ 설정 시 복구 가능 | ✅ 실무 최고 권장   |

---

## ⛨ 보안 고려 요소

- 웹 애플리케이션을 개발할 때는 다음과 같은 **기본 보안 요소들**을 반드시 고려해야 함
- Spring Security는 대부분의 보안 리스크에 대해 방어 장치를 기본 제공하지만, 올바른 설정과 인식이 중요

---

### 🔐 CSRF (Cross-Site Request Forgery) 방지

- CSRF는 사용자가 **의도하지 않은 요청을 공격자가 대신 보내는 공격** 방식
- ex)로그인된 사용자의 세션을 이용해 게시글 삭제 요청을 몰래 보내는 경우

#### ✅ Spring Security 기본 정책

- `POST`, `PUT`, `PATCH`, `DELETE` 요청은 CSRF 토큰이 없으면 **403 Forbidden** 처리됨
- `GET`은 안전하므로 기본적으로 토큰 검사 대상이 아님

#### ✅ Thymeleaf + HTMX에서의 대응

- Spring Security가 자동으로 뷰 템플릿에 `_csrf` 속성을 주입해주므로 아래와 같이 사용할 수 있음

```html

<button
        hx-post="/api"
        th:attr="hx-headers='{'X-CSRF-TOKEN':'[[${_csrf.token}]]'}'">
    저장
</button>
```

---

### 🔑 비밀번호 암호화

- 평문 비밀번호를 저장하거나 비교하는 것은 **심각한 보안 위험**
- Spring Security는 기본적으로 `BCryptPasswordEncoder`를 제공
- 내부적으로 **salt 처리 + 반복 해싱**이 적용되어 안전

```java
PasswordEncoder encoder = new BCryptPasswordEncoder();
String encoded = encoder.encode("raw_password");

// 로그인 시 비교
boolean matched = encoder.matches("raw_password", encoded);
```

---

### 🌐 CORS (Cross-Origin Resource Sharing)

- **서버와 프론트가 서로 다른 도메인**일 경우 브라우저가 자동으로 요청을 차단
- CORS 정책을 설정해 브라우저에게 "이 출처(origin)는 허용됨"을 알려야 함

```java

@Bean
public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/api/**")
                    .allowedOrigins("https://myfrontend.com")
                    .allowedMethods("GET", "POST", "PUT", "DELETE")
                    .allowCredentials(true); // 쿠키 전달 시 필요
        }
    };
}
```

> 특히 **JWT, 세션 로그인 시 쿠키를 주고받는 구조**일 경우 `allowCredentials(true)`가 필수입니다.

---

### 🚑 Session Fixation 방지

- 공격자가 미리 세션 ID를 만들어 피해자에게 할당시키고, 이후 탈취하는 방식
- Spring Security는 기본적으로 로그인 성공 시 **세션을 새로 생성**하여 이 공격을 방지

```java
void example() {
    http.sessionManagement()
            .sessionFixation()
            .migrateSession(); // 기본값
}
```

> 다른 옵션으로는 `.newSession()`, `.none()` 등이 있으나 일반적으로 `migrateSession()`을 유지합니다.

---

### 🧼 HTML Escape / XSS 방지

- 사용자의 입력이 HTML로 렌더링될 경우, `<script>` 태그 등을 그대로 출력하면 **XSS(스크립트 삽입)** 공격 가능
- Thymeleaf의 `th:text`는 기본적으로 HTML을 escape 처리하여 안전

```html
<!-- 안전: <script>가 escape됨 -->
<span th:text="${userInput}"></span>

<!-- 위험: 직접 HTML 렌더링 -->
<span th:utext="${userInput}"></span>
```

- Markdown 렌더링 등 사용자의 입력을 HTML로 변환해야 할 경우, 반드시 `Jsoup` 또는 HTML Sanitizer 등을 통해 필터링 필요
