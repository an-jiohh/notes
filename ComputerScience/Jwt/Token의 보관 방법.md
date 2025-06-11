
## **✅ JWT 보관 방법 상세 정리**

|**보관 위치**|**JS 접근**|**자동 전송**|**보안 수준**|**주의할 점**|**주 사용 대상**|
|---|---|---|---|---|---|
|**브라우저 메모리 (RAM)**|✅ 가능|❌ 수동 필요|🔐 높음 (XSS 안전)|새로고침 시 삭제됨|SPA, 보안 우선 앱|
|**localStorage**|✅ 가능|❌ 수동 필요|⚠️ 낮음 (XSS에 취약)|스크립트로 접근 가능|데모/개발용, 간단 앱|
|**sessionStorage**|✅ 가능|❌ 수동 필요|⚠️ 낮음 (XSS에 취약)|탭 닫으면 삭제|보안 + 짧은 세션|
|**Cookie (HttpOnly ❌)**|✅ 가능|✅ 자동 전송|⚠️ CSRF 위험|JS 접근 가능, XSS + CSRF 취약|옛날 방식, 권장하지 않음|
|**Cookie (HttpOnly ✅)**|❌ 불가능|✅ 자동 전송|🔐 매우 높음|CSRF 방지 위한 SameSite 설정 필수|✅ Refresh Token 보관|
|**Secure Storage (앱)**|❌ (앱 전용)|❌|🔐 매우 높음|네이티브 앱에서 사용|모바일 앱 전용|

---

## **🧠 각각의 특징 설명**

  

### **1.** **브라우저 메모리 (RAM)**

- JS 변수나 React 상태(state), context 등으로 관리
    
- Access Token을 **페이지 내부에서만** 쓰고 새로고침 시 사라짐(router가 달라질시)
    
- **가장 안전한 방식** (XSS 방어 가능)
    
- 새로고침 시 로그아웃됨 → UX 불편
    

  **추천:** 민감한 서비스, SSO 연동, 보안 우선 웹앱

---

### **2.** **localStorage**

- 브라우저에 저장되며 새로고침/재방문해도 유지됨
    
- JS에서 쉽게 접근 가능
    
- **XSS 공격에 매우 취약** → Access Token 탈취 위험
    

  
**권장되지 않음**. 단, 학습용, 데모 프로젝트, 보안 중요치 않은 토이 앱에서는 사용 가능

---

### **3.** 

### **sessionStorage**

- localStorage와 유사하나 **탭을 닫으면 자동 삭제**
    
- 그 외 특징은 동일 → 역시 JS 접근 가능
    

보안은 localStorage보다 살짝 나은 정도. 여전히 XSS 위험 있음.

---

### **4.** **쿠키 (HttpOnly ❌)**

- 서버가 Set-Cookie로 설정 가능
    
- JS에서도 읽고 쓸 수 있음 → XSS + CSRF 모두 취약
    
- 브라우저는 API 요청 시 자동 전송 (same origin 기준)
    

과거 방식. **보안 측면에서 매우 권장되지 않음**

---

### **5.**  **쿠키 (HttpOnly ✅)**

- 서버가 Set-Cookie로 설정하며, HttpOnly, Secure, SameSite 옵션 포함
    
- JS에서 읽지 못하므로 XSS 방어
    
- 자동 전송되므로 API 요청 시 별도 처리 불필요
    
- 단점: **CSRF 방어 전략** 필요 (SameSite=Strict 또는 CSRF Token 발급)
    

  
✅ **Refresh Token 보관용으로 실무에서 가장 많이 쓰는 방법**

```
Set-Cookie: refreshToken=JWT...; HttpOnly; Secure; SameSite=Strict; Path=/api/token
```

---

### **6.**  **Secure Storage (모바일 앱 전용)**

- React Native, Flutter 등에서 사용하는 **Keychain / Keystore API**
    
- 앱 외부에서는 접근 불가 → 탈취 위험 낮음
    
- JS 웹앱에서는 불가, **모바일 앱 전용**
    

  

> 모바일에서 Refresh Token 보관 시 최적

---

## **정리**

|**목적**|**Access Token**|**Refresh Token**|
|---|---|---|
|SPA (웹앱)|브라우저 메모리 / localStorage (보안 주의)|✅ HttpOnly Cookie|
|보안 우선 앱|브라우저 메모리만 사용|✅ HttpOnly Cookie + Redis 관리|
|모바일 앱|Secure Storage|Secure Storage (or 서버 저장)|

---

## **🔐 보안 설정 팁**

- HttpOnly: JS 접근 차단 → **XSS 방어**
    
- Secure: HTTPS에서만 전송
    
- SameSite: **CSRF 방어**
    
    - Strict: 가장 안전, 다른 도메인 전송 차단
        
    - Lax: 일반적 안전 수준
        
    - None: CORS 허용 시 필요 (단, 반드시 Secure=true)
        
    