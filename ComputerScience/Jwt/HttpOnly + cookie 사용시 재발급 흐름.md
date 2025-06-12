
# **HttpOnly 쿠키 기반 재발급 흐름**


## **1. 기본 흐름**

| **단계** | **설명**                                                          |
| ------ | --------------------------------------------------------------- |
| 1      | 서버에서 로그인 성공 시, Access Token을 **Set-Cookie 헤더로 HttpOnly 쿠키에 설정** |
| 2      | 클라이언트는 이후 요청마다 쿠키를 자동으로 포함하여 전송                                 |
| 3      | 서버는 쿠키에 담긴 Access Token을 추출하여 인증 수행                             |
| 4      | Access Token 만료 시, 클라이언트는 Refresh Token을 사용하여 **재발급 요청**        |
| 5      | 서버는 새로운 Access Token을 **쿠키로 다시 내려줌**                            |

---

## **2. 서버 측 설정 예 (Spring Boot 기준)**

```
ResponseCookie accessCookie = ResponseCookie.from("access_token", jwtToken)
    .httpOnly(true)
    .secure(true)
    .path("/")
    .maxAge(Duration.ofMinutes(15))
    .sameSite("Strict")
    .build();

response.addHeader("Set-Cookie", accessCookie.toString());
```

- httpOnly(true): JS로 접근 불가 (XSS 방어)
    
- secure(true): HTTPS 연결에서만 전송
    
- path("/"): 모든 경로에 전송
    
- sameSite("Strict" | "Lax" | "None"): CSRF 방지 수준 설정

---

## **3. 클라이언트 요청 구성**

```
// Axios 요청 시 자동으로 쿠키 포함
axios.defaults.withCredentials = true;

axios.get('/api/user/info');
```

- withCredentials: true는 CORS 허용 설정이 필요
    
- 서버에서도 Access-Control-Allow-Credentials: true 설정 필요
    

---

## **4. Access Token 재발급 흐름**

1. Access Token이 만료되어 서버가 401 응답을 반환
    
2. 클라이언트는 Refresh Token을 담아 /auth/refresh 요청
    
    (이때 Refresh Token도 쿠키에 포함되어 있음)
    
3. 서버는 새 Access Token을 발급하고 다시 Set-Cookie로 내려줌
    

---

## **5. 주의사항**

|**항목**|**설명**|
|---|---|
|CSRF 방어|쿠키는 자동 전송되므로 CSRF 방어 필수 → SameSite, CSRF 토큰 등 사용 필요|
|쿠키 관리 책임|서버가 모든 토큰 발급·삭제를 쿠키로 처리해야 함 (로그아웃 시 삭제 포함)|
|클라이언트 토큰 접근 불가|JS에서는 토큰을 읽을 수 없기 때문에, 상태 갱신을 위한 별도 사용자 정보 요청 필요|
|쿠키 유효시간 확인 불가|만료 시점은 클라이언트가 알 수 없음 → 401 실패 시 처리 방식 권장|

---

## **6. 쿠키 방식 vs 헤더 방식 비교**

|**항목**|**쿠키 방식 (**HttpOnly**)**|**Authorization 헤더**|
|---|---|---|
|JS 접근 가능 여부|불가|가능|
|XSS 방어|강함|JS 접근 허용 시 취약|
|CSRF 취약 여부|있음 → 설정 필수|없음|
|UX 제어 유연성|낮음|높음|
|실무 사용처|웹 브라우저 중심 SPA|토큰 기반 REST API, 모바일 등|
