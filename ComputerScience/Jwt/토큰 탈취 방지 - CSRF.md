

CSRF는 사용자가 **로그인된 상태**라는 점을 악용해, 공격자가 **의도하지 않은 요청을 실행하도록 유도**하는 공격 방식이다. 사용자는 공격자가 만든 악성 웹페이지에 접속하고, 해당 페이지에서 자동으로 전송된 요청이 피해자의 인증 정보를 포함하게 된다.

### 예시
사용자가 은행 웹사이트에 로그인한 상태에서 공격자가 만든 악성 웹사이트를 방문한다고 가정한다.  
공격자의 사이트에는 다음과 같은 폼과 스크립트가 삽입되어 있다.

```html
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="to" value="attacker" />
  <input type="hidden" name="amount" value="1000000" />
</form>
<script>document.forms[0].submit();</script>
```

결과
bank.com에 로그인된 상태로 해당 요청을 보내게 되어 attacker에게 1000000을 보내게 된다.  

이 요청은 피해자의 브라우저가 `bank.com`에 로그인된 상태라면 쿠키를 자동 전송하게 되어 정상 요청처럼 보이게 된다.

---

## 방어 전략

### 1. SameSite 쿠키 설정

- 쿠키의 교차 사이트 전송을 제한하는 브라우저 기능
    
- `Strict`, `Lax`, `None` 중 선택 가능

```http
Set-Cookie: access_token=...; SameSite=Strict; HttpOnly; Secure
```

| 옵션       | 설명                       |
| -------- | ------------------------ |
| `Strict` | 외부 요청에서 쿠키 전송 차단 (보안 강함) |
| `Lax`    | 일부 GET 요청만 허용            |
| `None`   | 제한 없음 (`Secure` 필수)      |
#### SameSite와 사이트 판별 기준

SameSite는 요청이 발생한 페이지와 요청 대상의 '사이트(Site)'가 같은지를 기준으로 판단한다. 여기서 '사이트'란 브라우저가 판단하는 **등록 도메인(eTLD+1)** 기반으로, 다음 조건들을 고려한다:

- **프로토콜은 고려하지 않음** (HTTP/HTTPS 구분 없음)
- **서브도메인은 무시됨** (`example.com`과 `sub.example.com`은 같은 사이트로 간주)
- **포트는 무시됨**

예시:

- `https://example.com:443`와 `https://sub.example.com:3000` → SameSite에서는 같은 사이트
- `http://example.com`과 `https://example.com` → SameSite에서는 같은 사이트
- `example.com`과 `example.org` → 다른 사이트로 간주

예시:

- `https://example.com:443`와 `https://sub.example.com:3000` → SameSite에서는 동일한 사이트
- `https://example.com`과 `https://example.org` → 서로 다른 사이트로 간주

참고로, **CORS는 Origin 기준으로 스킴, 도메인, 포트가 모두 일치해야 동일 출처(Same-Origin)**로 인정한다. 이 때문에 SameSite와 CORS는 동일해 보이지만 실제로는 **다른 기준을 사용한다.**

---

### 2. CSRF 전용 토큰 사용

- 서버가 로그인 시 토큰을 발급하고, 클라이언트는 이를 헤더나 요청 본문에 포함
    
- 서버는 요청에 포함된 값과 세션 정보를 비교해 검증
    

```http
X-CSRF-Token: abcdef123456
```


장점: 쿠키 자동 전송 외에 추가적인 검증 수단 제공

---

### 3. Authorization 헤더 방식

- JWT를 Authorization 헤더로 보내는 방식은 CSRF 방어에 효과적
    
- 브라우저가 자동으로 헤더를 붙이지 않기 때문에 악성 사이트에서는 모방 불가능
    

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

주의: 이 방식은 보안상 `LocalStorage`나 메모리에서 토큰을 직접 다루므로, **XSS 방어 대책과 병행 필수**

---

## 쿠키 기반과 헤더 기반 비교

|구분|자동 전송|CSRF 위험|보안 전략 필요|
|---|---|---|---|
|쿠키 (`HttpOnly`)|O|O|SameSite, CSRF 토큰|
|Authorization 헤더|X|X|XSS 방어 필요|

---

## 실무 권장 조합

| 조건            | 권장 방식                                |
| ------------- | ------------------------------------ |
| 쿠키 기반 인증      | `HttpOnly + SameSite + CSRF 토큰`      |
| SPA 등 JS 제어 앱 | `Authorization 헤더 + 메모리 저장 + XSS 방어` |
