# Controller에서 Cookie를 꺼내보자

## **1.** **@CookieValue (컨트롤러 메서드 파라미터)**

  

가장 간단한 방법

```
@GetMapping("/some-endpoint")
public ResponseEntity<?> someEndpoint(@CookieValue("refreshToken") String refreshToken) {
    // 쿠키 값 사용
    return ResponseEntity.ok("refreshToken = " + refreshToken);
}
```

### **옵션**

```
@CookieValue(value = "refreshToken", required = false, defaultValue = "") String token
```

**속성 상세 설명**

| **속성 이름**    | **설명**                                                               |
| ------------ | -------------------------------------------------------------------- |
| value        | 찾을 쿠키의 이름 (생략 시 파라미터 이름이 사용됨)                                        |
| required     | 쿠키가 필수인지 여부 (기본값은 true)<br>-  **MissingCookieValueException** 예외가 발생 |
| defaultValue | 쿠키가 없을 때 사용할 기본값 (이 값을 설정하면 required=false처럼 동작)                     |

---

## **2.** **HttpServletRequest 사용 (로우 레벨 접근)**

  

모든 쿠키를 순회하면서 직접 찾는 방식입니다.

```
@GetMapping("/check")
public ResponseEntity<?> check(HttpServletRequest request) {
    Cookie[] cookies = request.getCookies();
    if (cookies != null) {
        for (Cookie cookie : cookies) {
            if ("refreshToken".equals(cookie.getName())) {
                String token = cookie.getValue();
                // 사용
                return ResponseEntity.ok("refreshToken = " + token);
            }
        }
    }
    return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("No refresh token");
}
```

---

## **3.** **HandlerMethodArgumentResolver** **또는**  **Filter** **/** **Interceptor** **에서 사용**


### **예: JWT 토큰 인터셉터에서 쿠키 꺼내기**

```
Cookie[] cookies = request.getCookies();
if (cookies != null) {
    for (Cookie cookie : cookies) {
        if ("refreshToken".equals(cookie.getName())) {
            String refreshToken = cookie.getValue();
            // 토큰 검증 로직
        }
    }
}
```