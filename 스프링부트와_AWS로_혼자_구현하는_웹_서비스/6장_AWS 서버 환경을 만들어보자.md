# 📘 학습 자료 요약

## 🔗 참고 사항

- 원본 참고 도서는 **2019년 발행** → 기술 스택 구버전 가능성 있음
- 따라서 **최신 개발 환경(Java 21, Spring Boot 3.4)** 기준 병행 학습 결정

---

# ☁️ AWS EC2 개요 및 실무 설정 가이드

## ✅ EC2란?

- **EC2 (Elastic Compute Cloud)** 는 AWS에서 제공하는 가상 서버
- 클라우드 환경에서 온디맨드 방식으로 리눅스/윈도우 서버를 생성하고 사용할 수 있음
- 사용자는 서버에 대한 **루트 권한**을 가지고 설치/설정/배포 가능

---

## 🌍 리전(Region)과 가용 영역(Availability Zone)

### 📌 리전(Region)이란?

- **지리적으로 분산된 AWS 데이터센터 그룹**
- 예: `ap-northeast-2`(서울), `us-west-1`(캘리포니아), `eu-central-1`(프랑크푸르트)
- 사용자와 가까운 리전을 선택하면 **지연(latency)** 이 줄어들고, 일부 서비스는 지역별로 가격이 다름

### 📌 가용 영역(AZ, Availability Zone)

- 각 리전은 **2개 이상**의 가용 영역으로 구성
- 하나의 AZ가 문제가 생겨도 다른 AZ로 자동 전환하여 **고가용성** 확보 가능

---

## 🖥️ 인스턴스 관련 개념

### ✅ AMI (Amazon Machine Image)

- EC2 인스턴스의 **운영체제 + 설정 + 애플리케이션**이 포함된 이미지
- 기본 제공: Amazon Linux, Ubuntu, Windows 등
- 사용자 정의 AMI로 백업, 클론 서버 생성도 가능

### ✅ 인스턴스 타입: T2, T3, M5 등

- **T2, T3 계열**: 일반적인 범용 인스턴스. 저렴하고 스파이크 트래픽에 대응
- **M, C 계열**: 메모리/CPU 성능 중심으로 최적화된 서버
- **R, X 계열**: 메모리 집중형 워크로드용

### ✅ CPU 크레딧 (T 계열만 해당)

- T2, T3 인스턴스는 **CPU 성능을 일정량만 제공**하고, 사용량에 따라 **크레딧을 소모**
- CPU 크레딧을 다 쓰면 성능이 떨어짐 → CPU 부하가 높을 경우 M 또는 C 계열을 고려

---

## 🔐 보안 설정

### ✅ 보안 그룹 (Security Group)

- **가상 방화벽** 역할
- EC2 인스턴스에 대한 **포트/프로토콜/IP 주소 기반 접근 제어**
- 기본적으로는 모든 트래픽 차단 상태 → 포트 22(SSH), 80(HTTP), 443(HTTPS) 등을 명시적으로 열어야 함

### ✅ 방화벽 규칙 예시

| 프로토콜 | 포트  | 설명           |
|------|-----|--------------|
| TCP  | 22  | SSH 접속용      |
| TCP  | 80  | 웹서버(HTTP) 접속 |
| TCP  | 443 | HTTPS 접속     |

### ✅ PEM 키

- EC2 인스턴스 생성 시 자동으로 발급되는 **비대칭 키**
- `.pem` 파일은 **절대 분실 금지**  
  → 분실 시 인스턴스에 다시 접근 불가 (재생성도 안 됨)

---

## 💡 팁: IP 관리

### ✅ 고정 IP가 필요한 이유

- EC2는 재시작할 때마다 **기본 공인 IP가 바뀜**
- 외부 시스템과 연동 시 IP 고정이 필요할 경우 **탄력적 IP(Elastic IP)** 사용

### ✅ 탄력적 IP (Elastic IP)

- AWS가 제공하는 **고정 공인 IP 주소**
- 하나의 EC2 인스턴스에 바인딩 가능
- EC2가 꺼져 있으면 과금됨 → 항상 연결 상태를 유지하거나 해제 필요

---

## 🛠 SSH 접속 방법 (Mac/Linux 기준)

1. `.pem` 키 파일 권한 변경
   ```bash
   chmod 400 my-key.pem
   ```

2. SSH 접속
   ```bash
   ssh -i my-key.pem ec2-user@<EC2-공인-IP>
   ```

3. Ubuntu AMI의 경우 사용자 이름은 `ubuntu`  
   Amazon Linux의 경우는 `ec2-user`

> ❗ Windows 사용자는 PuTTY 또는 WSL 기반 SSH 사용 필요

---

## 🧾 EC2 서버 생성 후 필수 설정

### ✅ Java 설치

- 대부분의 백엔드 서버(Spring 등)는 Java 기반이므로 기본 JDK 설치는 무조건 필요함
- `Amazon Corretto 17` 사용을 권장함
- 설치 전 `yum update`로 보안 패치까지 함께 적용하는 게 좋음

```bash
sudo yum update -y
sudo yum install java-17-amazon-corretto -y
```

---

### ✅ 타임존은 꼭 Asia/Seoul로 변경

- 기본이 UTC라서 로그 시간, 스케줄링이 꼬이기 쉬움
- 한국 서버라면 `Asia/Seoul`로 맞추는 게 표준임

```bash
sudo timedatectl set-timezone Asia/Seoul
```

---

### ✅ 호스트명은 명확하게 변경

- AWS는 기본 호스트명이 무작위 문자열임
- 운영/개발 서버 구분, 로그 추적을 위해 직관적인 이름 설정이 필요함

```bash
sudo hostnamectl set-hostname my-ec2-server
```

---

## 🔧 추가적으로 해두면 좋은 설정들

### ☑️ 자주 쓰는 도구 설치는 필수임

```bash
sudo yum install git unzip net-tools -y
```

- 배포/압축/네트워크 확인 등 자주 사용

### ☑️ SSH 키 권한 변경은 안 하면 접속 안 됨

```bash
chmod 400 my-key.pem
```

### ☑️ 탄력적 IP(EIP)는 재시작해도 IP 유지하고 싶을 때 필수임

### ☑️ systemd로 서비스 등록해두면 서버 재시작 후에도 자동 실행됨

### ☑️ 필요 시 시간 동기화도 해두는 게 좋음 (chrony 사용)

```bash
sudo yum install chrony -y
sudo systemctl enable chronyd
sudo systemctl start chronyd
```

---

## ✅ 추가로 추천되는 설정

- **Fail2Ban, ufw 등 보안 패키지 설치**
- **CloudWatch Agent 설정** → 로그 및 메트릭 수집
- **EC2 Auto Recovery 또는 Backup 스크립트 구성**
- **도커 / NGINX 등 런타임 설치 자동화 스크립트 (UserData 활용)**