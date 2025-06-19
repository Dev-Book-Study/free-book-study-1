# 📘 학습 자료 요약

## 🔗 참고 사항

- 원본 참고 도서는 **2019년 발행** → 기술 스택 구버전 가능성 있음
- 따라서 **최신 개발 환경(Java 21, Spring Boot 3.4)** 기준 병행 학습 결정

---

## ☁️ AWS RDS

### ✅ RDS란 무엇인가?

- **RDS(Relational Database Service)** 는 AWS에서 제공하는 **관리형 관계형 데이터베이스 서비스**
- 지원 DB 엔진: MySQL, MariaDB, PostgreSQL, Oracle, SQL Server, Amazon Aurora
- 인프라 설정, 운영, 백업, 복구, 패치 등을 **자동화**하여 개발자가 비즈니스 로직에 집중할 수 있도록 함

### ✅ 왜 사용해야 하는가?

- **운영 관리 자동화**: 백업, 장애 조치, 소프트웨어 패치 등을 자동 수행
- **보안성**: IAM 연동, 암호화, VPC 설정, 보안 그룹 등 제공
- **확장성**: 수직/수평 스케일링 지원
- **가용성**: Multi-AZ 배포로 고가용성 구성 가능
- **비용 효율성**: 온디맨드 또는 예약 인스턴스로 탄력적 과금

---

## 🧬 Amazon Aurora

### ✅ Aurora란 무엇인가?

- AWS가 자체 개발한 **고성능 클라우드 네이티브 관계형 데이터베이스**
- MySQL, PostgreSQL과 **호환 가능**
- RDS 기반에서 동작하며, 고가용성과 확장성 중심으로 설계됨

### ✅ 장점

- **MySQL 대비 최대 5배, PostgreSQL 대비 3배 성능 개선** (AWS 발표 기준)
- 고속 자동 복제 및 장애 복구
- 자동 스토리지 확장 (최대 128TiB)
- Compute와 Storage 분리 운영
- DB Cluster 단위로 읽기 전용 인스턴스 구성 가능 (Read Replicas)

### ✅ 단점

- **비용이 높음** (MySQL/MariaDB보다 가격 부담 있음)
- Aurora 고유 기능으로 인해 **AWS 종속성 강화**
- 로컬 개발 환경과의 차이 존재

---

## 🔍 MySQL vs MariaDB vs Aurora

| 항목    | MySQL             | MariaDB              | Aurora (MySQL 호환) |
|-------|-------------------|----------------------|-------------------|
| 소유    | Oracle            | 커뮤니티 주도 (MariaDB 재단) | Amazon            |
| 라이선스  | GPL + Proprietary | GPL                  | 상용 (AWS 내부 전용)    |
| 성능    | 안정적               | 일부 기능 개선             | 고성능 (스토리지 최적화)    |
| 호환성   | 광범위               | MySQL과 호환            | MySQL 5.6/5.7 호환  |
| 사용처   | 범용적으로 널리 사용       | 오픈소스 지향 환경에서 활용      | AWS 기반 환경에 최적화    |
| 확장성   | 제한적               | MySQL과 유사            | 자동 스케일링 및 고가용성 제공 |
| 운영 비용 | 상대적 저렴            | 저렴                   | 높음                |

> ✅ **Aurora는 높은 성능과 안정성을 제공하지만, 비용과 AWS 종속성을 고려해야 함**

---

## 🔐 보안 그룹 지정

### ✅ 보안 그룹이란?

- AWS에서 제공하는 **가상 방화벽(Security Group)**
- 인바운드/아웃바운드 트래픽을 **인스턴스 단위로 제어**
- RDS 접근을 IP/포트 기반으로 제한 가능

### ✅ 보안 그룹 지정이 필요한 이유

- 외부의 불필요한 접근으로부터 **DB를 보호**
- DB 서버가 퍼블릭 IP를 가지는 경우, 필수 설정 요소
- 실수로 모든 포트/모든 IP 허용 시 보안 위협 발생 가능

### ✅ 설정 방법

1. AWS 콘솔 접속 → EC2 또는 RDS → "보안 그룹" 이동
2. 해당 RDS 인스턴스에 연결된 보안 그룹 클릭
3. "인바운드 규칙 편집" 클릭
4. 필요한 IP(예: `123.123.123.123/32`) 및 포트(예: `3306`) 허용

---

## ⚙️ RDS 필수 설정

### ⏱ 타임존(Time Zone)

- 기본값: UTC
- 권장값: `Asia/Seoul` (KST 기준)
- 설정 방법:
    - RDS → 파라미터 그룹 → `time_zone` 값 변경
    - 또는 MySQL 접속 후 `SET time_zone = 'Asia/Seoul';`

### ✍️ 문자셋(Character Set)

- 권장 설정:
    - `character_set_server = utf8mb4`
    - `collation_server = utf8mb4_general_ci` 또는 `utf8mb4_unicode_ci`
- `utf8mb4`: 이모지 포함 다국어 지원 가능

### 🔌 Max Connections

- 기본값은 인스턴스 타입마다 다름 (예: db.t3.medium = 약 150)
- 설정 위치:
    - RDS → 파라미터 그룹 → `max_connections` 값 수정
- 고트래픽 예상 시 적절한 인스턴스 스펙 조정 필요

---

## 🛠 GUI 클라이언트 - DataGrip

### ✅ 왜 사용해야 하는가?

- JetBrains 제품군의 **강력한 DB 관리 도구**
- 다양한 DB 지원 (MySQL, PostgreSQL, Oracle, SQLite 등)
- IntelliJ 사용자에게 익숙한 UI/UX
- 쿼리 자동 완성, ERD 뷰, 스키마 동기화, 히스토리 등 기능 제공

### ✅ 접근 가능한 대상

- **MySQL, MariaDB, PostgreSQL, Oracle, SQL Server 등 대부분 RDBMS**
- **Redis 등 NoSQL**도 별도 플러그인 설치 후 사용 가능

### ✅ 간단한 사용법

1. 좌측 상단 `Database` 탭 → `+` 버튼 클릭 → 데이터 소스 선택
2. 드라이버 설치 후, RDS Endpoint, 포트, 사용자, 비밀번호 입력
3. 테스트 연결 → 성공 시 저장 후 사용

### ✅ 활용 예시

- 로컬 DB ↔ RDS 쿼리 비교 및 데이터 이관
- RDS에 저장된 테이블/뷰/인덱스 등 구조 파악
- Redis 키-값 데이터 확인, TTL(만료시간) 체크
- 빠른 쿼리 실행 및 결과 내보내기 (.csv, .sql 등)

---

## 📌 요약

| 항목       | 주요 포인트 요약                                  |
|----------|--------------------------------------------|
| RDS      | 관리형 DB 서비스, 자동화, 보안 강화                     |
| Aurora   | 고성능 RDS, AWS 특화, 비용 ↑                      |
| DB 비교    | MySQL - 안정 / Maria - 오픈소스 / Aurora - 성능 최강 |
| 보안 그룹    | DB 접근 제한 필수 설정                             |
| 필수 설정    | 타임존, charset, max connection 등 조정 필요       |
| DataGrip | 범용 GUI 클라이언트, Redis 지원도 가능                 |

---
