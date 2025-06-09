## **✅ JWT란 무엇인가?**

  

### **1. 정의**

**JWT**(JSON Web Token)는

**서버와 클라이언트 간에 인증 정보를 JSON 형식의 토큰으로 안전하게 주고받기 위한 방식**입니다.

JWT는 **디지털 서명이 포함된 문자열**이기 때문에, 위조를 막고 **서버가 별도 상태 저장 없이 사용자를 식별할 수 있는** 장점이 있습니다.

---

### **2. 핵심 특징**

- **Self-contained**: 토큰 하나에 필요한 정보를 모두 포함 → 서버가 별도 DB 조회 없이 인증 처리 가능
    
- **Stateless**: 서버가 세션을 저장하지 않음 (상태 없음 → 수평 확장에 유리)
    
- **서명 기반 무결성 보장**: 중간에 토큰이 위·변조되지 않았음을 서명을 통해 검증 가능
    
- **Base64Url 인코딩**: 텍스트 기반이므로 URL, HTTP Header에 안전하게 포함 가능
    

---

### **3. JWT는 언제 사용하나?**

- 사용자 로그인 후 **토큰 기반 인증** 방식으로 인증 상태 유지
    
- **REST API** 기반 백엔드 서비스에서 인증 상태를 유지하는 데 적합
    
- **마이크로서비스 간 통신** 시 사용자 정보를 공유하는 데 사용
    

---

### **4. 구조 (예시)**

  

JWT는 세 부분으로 구성됩니다:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9       ← Header
.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ   ← Payload
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c ← Signature
```

**Header**: 알고리즘 정보

**Payload**: 사용자 정보 (ex. userId, role 등)

**Signature**: 비밀키 또는 공개키를 이용한 서명값 (위변조 검증용)

---

### **5. 예시 상황**

  사용자가 로그인하면 서버는 해당 사용자 정보로 JWT를 생성하고 응답에 담아 클라이언트에 전달
이후 클라이언트는 이 JWT를 요청 헤더에 포함시켜 서버에 보내며, 서버는 토큰만으로 사용자를 식별하고 요청을 처리

