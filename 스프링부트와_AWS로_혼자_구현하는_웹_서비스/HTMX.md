# ⚡ HTMX

## 📘 1. 개요

**HTMX**는 HTML 속성만으로 동적인 서버 통신을 가능하게 하는 경량 JavaScript 라이브러리입니다.

> "You can access modern browser features directly from HTML."

- JavaScript 없이도 **AJAX, partial rendering, client-side event handling** 가능
- SSR(Server Side Rendering)과 **궁합이 뛰어남**
- **점진적 개선(PJAX)** 방식으로 전통적인 MPA를 SPA처럼 다룰 수 있음
- 파일 크기 약 14KB로 매우 가볍고 의존성 없음

📦 공식 사이트: [https://htmx.org](https://htmx.org)

---

## 🔤 2. 기본 문법

HTMX는 `hx-*` 속성을 통해 동작합니다. 주요 속성은 다음과 같습니다:

| 속성            | 설명 |
|-----------------|------|
| `hx-get`        | 지정한 URL로 **GET 요청**을 보냅니다 |
| `hx-post`       | **POST 요청**을 보냅니다 |
| `hx-target`     | 응답 내용을 삽입할 대상 요소 선택자 |
| `hx-swap`       | 응답 삽입 방식 (innerHTML, outerHTML 등) |
| `hx-trigger`    | 이벤트 트리거 지정 (예: `click`, `change`) |
| `hx-include`    | 함께 전송할 추가 폼 요소 지정 |
| `hx-vals`       | JavaScript 객체를 JSON으로 전송 |

📝 예:
```html
<button hx-get="/hello" hx-target="#result">불러오기</button>
<div id="result"></div>
```

---

## 🧪 3. 예시 코드

### ✅ 사용자 정보를 비동기로 로딩하는 예

**📄 HTML (Thymeleaf 템플릿)**
```html
<button hx-get="/user/info" hx-target="#user-info" hx-swap="innerHTML">
  사용자 정보 보기
</button>
<div id="user-info"></div>
```

**📦 Spring Boot 컨트롤러**
```java
@GetMapping("/user/info")
public String getUserInfo(Model model) {
    model.addAttribute("user", userService.getCurrentUser());
    return "fragments/user-info :: info"; // Thymeleaf 조각
}
```

**📄 Thymeleaf Fragment (`fragments/user-info.html`)**
```html
<div th:fragment="info">
  <p th:text="${user.name}">홍길동</p>
  <p th:text="${user.email}">user@example.com</p>
</div>
```

---

## 🤝 4. Spring Boot + Thymeleaf 궁합

### 왜 잘 맞는가?

- HTMX는 **서버에서 HTML 조각을 응답받아 삽입**하는 구조 → SSR을 지향하는 Thymeleaf와 자연스럽게 결합됨
- Thymeleaf의 fragment(`th:fragment`)와 조합하여 **부분 렌더링(PJAX)** 구현이 용이
- Spring Boot의 **MVC 구조와 REST 컨트롤러** 방식 그대로 사용 가능

### 예시 구조

```
templates/
 ├─ fragments/
 │   └─ user-info.html
 └─ main.html (전체 레이아웃)
```

### 장점

✅ 기존 Spring MVC 프로젝트에 **JS 프레임워크 없이 동적 UI** 구현 가능  
✅ HTMX는 SEO에도 유리함 (전체 페이지 렌더링 포함 가능)  
✅ JS 의존도 줄이고 **간결한 유지보수** 가능

---

## 📌 참고

- 공식 문서: [https://htmx.org/docs/](https://htmx.org/docs/)
- Thymeleaf: [https://www.thymeleaf.org](https://www.thymeleaf.org)
- Spring Boot Docs: [https://docs.spring.io/spring-boot/docs/current/reference/html/](https://docs.spring.io/spring-boot/docs/current/reference/html/)
