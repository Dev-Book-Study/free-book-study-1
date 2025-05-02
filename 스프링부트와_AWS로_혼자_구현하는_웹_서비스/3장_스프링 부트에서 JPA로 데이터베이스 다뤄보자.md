# 📘 인텔리제이 학습 자료 요약

## 🔗 참고 사항

- 원본 참고 도서는 **2019년 발행** → 기술 스택 구버전 가능성 있음
- 따라서 **최신 개발 환경(Java 21, Spring Boot 3.4)** 기준 병행 학습 결정

---

# 🧪 JPA 개요

## 🔍 자주 사용하는 DB (웹서버 점유율 기준 Top 5)

| 순위 | DBMS              | 특징                           |
|----|-------------------|------------------------------|
| 1  | **MySQL**         | 오픈소스, 대중적, Spring Boot 기본 선택 |
| 2  | **PostgreSQL**    | 트랜잭션 강력, JSON 지원, 클라우드에 적합   |
| 3  | **SQLite**        | 경량, 모바일/테스트 용도               |
| 4  | **Oracle DB**     | 상용, 대기업/금융권에서 사용             |
| 5  | **MS SQL Server** | .NET/Windows 환경에서 사용         |

---

## 📦 ORM이란?

- **ORM (Object-Relational Mapping)**: 객체와 테이블을 자동으로 매핑해주는 기술
- **JPA**는 자바 진영의 ORM 표준
- **MyBatis/iBatis**는 ORM이 아니고 SQL Mapper로서 직접 SQL을 작성할 수 있게 도와줌

| 항목    | ORM (JPA)      | SQL Mapper (MyBatis 등) |
|-------|----------------|------------------------|
| 쿼리 작성 | 생략하거나 JPQL 사용  | 직접 작성 (XML 또는 어노테이션)   |
| 구조 중심 | 객체 중심 (도메인 모델) | SQL 중심                 |
| 유지보수  | 도메인 설계와 일관성    | 쿼리 따로 관리 → 유지보수 부담     |
| 성능 제어 | fetch 전략 등 필요  | SQL 최적화 자유로움           |

---

## 📘 JPA란?

- Java Persistence API
- 자바 ORM을 위한 **표준 인터페이스 명세**
- Hibernate, EclipseLink 등 구현체 존재
- 엔티티 중심으로 DB와의 통신 추상화

---

## 👍 JPA 장점 vs 👎 단점

### 장점

- 도메인 모델 기반 설계 가능
- 반복적인 CRUD 코드 제거
- 트랜잭션, 캐시, 지연 로딩, 변경 감지 등 다양한 기능
- Spring Data JPA와 결합 시 생산성 극대화

### 단점

- 러닝 커브 존재
- 복잡한 쿼리에 대한 한계 → QueryDSL 보완 필요
- 잘못 사용 시 퍼포먼스 이슈 (N+1 문제 등)

---

## 🧭 DDD 개발 방법론

- 도메인 중심으로 소프트웨어 설계
- 도메인 엔티티가 스스로 로직을 갖고 상태를 변화시킴
- 트랜잭션 단위도 도메인 기준으로 설계

```java
public class Order {
    public void completePayment() {
        if (this.status == OrderStatus.PENDING) {
            this.status = OrderStatus.PAID;
        }
    }
}
```

## 🌿 Spring Data JPA 계층 구조

- JPA: ORM 표준 인터페이스
- Hibernate: 가장 널리 쓰이는 JPA 구현체
- Spring Data JPA: JPA 사용을 간편화한 추상화 계층

## 🔄 VO(Value Object)

- ID 없음, 상태가 동일하면 동일 객체
- 변경 불가능 (불변성 보장)
- equals/hashCode 오버라이딩 필요

```java

@Embeddable
public class Email {
    @Column(name = "email", nullable = false)
    private String value;

    protected Email() {
    }

    public Email(String value) {
        this.value = validate(value);
    }

    public String getValue() {
        return value;
    }
}

@Entity
public class User {
    @Embedded
    private Email email;
}
```

## 📨 DTO(Data Transfer Object)

### 🧱 계층별 DTO 설계 (클린 아키텍처 관점)

| 구분           | 위치            | 예시                    | 역할                       |
|--------------|---------------|-----------------------|--------------------------|
| Request DTO  | Controller 입력 | `UserRegisterRequest` | 클라이언트 요청 수신              |
| Response DTO | Controller 출력 | `UserResponse`        | 클라이언트에 응답 전달             |
| Command DTO  | Service 내부    | `RegisterUserCommand` | 비즈니스 로직에 전달할 값 구조화       |
| Query DTO    | Repository 결과 | `UserSummaryDto`      | 조회용 최적 DTO (JPQL/Native) |

## 🧱 DAO

- 전통적으로 SQL + JDBC 방식에서 데이터 접근 로직 분리
- 현재는 Repository 계층이 DAO 역할을 수행

## JpaRepository, JpaRepositorySupport 예시

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByEmail(String email);
}
```

```java

@Repository
public class UserRepositorySupport extends QuerydslRepositorySupport {
    public UserRepositorySupport() {
        super(User.class);
    }

    public List<User> findActiveUsers() {
        QUser user = QUser.user;
        return from(user)
                .where(user.status.eq(Status.ACTIVE))
                .fetch();
    }
}
```

```java
public interface UserViewPositionRepository extends JpaRepository<UserViewPosition, Integer>, UserViewPositionRepositorySupport {
    List<UserViewPosition> findByUserProfileAndPositionIdAndDeviceEnvironmentAndDp(UserProfile userProfile, Integer positionId, DeviceEnvironment deviceEnvironment, String dp);

    Optional<List<UserViewPosition>> findAllByUserProfile(UserProfile userProfile);

    void deleteAllByPositionId(Integer positionId);
}
```

## 🧪 JPA 테스트 코드

```java

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void 유저_저장_및_조회() {
        // given
        User user = new User(new Email("test@example.com"), "홍길동");
        userRepository.save(user);

        // when
        List<User> result = userRepository.findByEmail("test@example.com");

        // then
        assertThat(result).hasSize(1);
        assertThat(result.get(0).getName()).isEqualTo("홍길동");
    }
}

```

## @Transactional 주요 옵션 정리

| 옵션                | 설명                                                 |
|-------------------|----------------------------------------------------|
| `readOnly = true` | 변경 감지, flush 비활성화. 조회 전용 트랜잭션에 적합                  |
| `propagation`     | 트랜잭션 전파 방식: `REQUIRED`, `REQUIRES_NEW`, `NESTED` 등 |
| `rollbackFor`     | 지정된 예외가 발생하면 롤백 수행 (예: `IOException.class`)        |
| `noRollbackFor`   | 특정 예외는 롤백하지 않도록 설정                                 |
| `timeout`         | 트랜잭션 수행 제한 시간 (초 단위)                               |
| `isolation`       | 트랜잭션 격리 수준: `READ_COMMITTED`, `SERIALIZABLE` 등     |

## 🚨 JPA 사용 시 주의사항

### ❌ Entity를 Request/Response에 사용 금지

순환 참조, 직렬화 문제, 도메인 변경 영향이 클 수 있음

### 🧼 set 메서드보다 행위 메서드 사용

- user.setName("변경") ❌

- user.changeName("변경") ✅ → 캡슐화 + 명확한 책임 분리

### 🧩 @Transactional 남용 금지

- Service 계층에만 선언하도록 제한

- Controller나 Repository에 선언 시 트랜잭션 흐름이 불명확해짐

### 🔒 OSIV(Open Session In View)

- 기본값 true → View 렌더링 시 Lazy Loading 가능

- 운영 환경에서는 false 권장 → 모든 조회는 Service 계층 안에서 끝내야 함