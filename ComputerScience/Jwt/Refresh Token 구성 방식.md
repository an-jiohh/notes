# **Refresh Token 구성 방식

Access Token은 일반적으로 짧은 만료 시간(수 분~수십 분)을 가지며, 토큰 탈취 시 피해를 줄이기 위한 전략으로 사용된다. 그러나 이 짧은 만료 시간으로 인해 사용자 경험이 저하될 수 있다. 이를 보완하기 위해 등장한 것이 Refresh Token이다. Refresh Token은 만료 시간이 길며, 사용자가 다시 로그인하지 않고도 새로운 Access Token을 발급받을 수 있게 한다.

Refresh Token을 어떻게 설계하고 저장할지에 따라 보안성과 시스템 구조가 달라진다. 대표적인 두 가지 방식은 JWT 기반과 Opaque(불투명) 토큰 방식이다.

---

## **JWT 기반 Refresh Token**

  

### **구성 방식**

  

JWT 형식의 Refresh Token은 Header.Payload.Signature 형태를 가진다. 구조적으로 Access Token과 동일하며, 자체적으로 정보를 포함하고 서명 검증을 통해 위조 여부를 판단할 수 있다.

  

#### **예시**

```
// JWT Payload 예시
{
  "sub": "user123",
  "iat": 1718172000,
  "exp": 1720764000,
  "jti": "refresh-abc123"
}
```

### **장점**

- **Stateless 구조 가능**: DB 없이 자체 서명 검증만으로 토큰 유효성 판단 가능
    
- **확장성 유리**: 인증 서버 확장 시에도 별도 세션 저장소 불필요
    
- **빠른 처리**: DB 조회 없이 토큰 하나만으로 처리 가능
    

  

### **단점**

- **토큰 탈취 시 정보 노출 위험**: Payload에 사용자 정보 포함 시 공격자에게 유용한 정보 제공 가능
    
- **블랙리스트 관리 어려움**: 로그아웃 등 토큰 강제 무효화를 위해서는 별도 저장소(Blacklist DB 등) 필요
    
- **길이 증가**: URL 등에 포함시키기에는 크기가 큼
    

  

### **실무 사용 예시**

- **모바일 앱**: 로컬 저장소에 Refresh Token을 저장하고, 주기적으로 Access Token을 재발급
    
- **내부 시스템 또는 분산 마이크로서비스**: 인증 서버와 리소스 서버가 분리되어 있고, 별도 세션 서버 없이 JWT만으로 인증 유지
    

---

## **Opaque(불투명) Refresh Token**

  

### **구성 방식**

Opaque Token은 의미 없는 무작위 문자열로 구성되며, 자체적으로는 정보를 담지 않는다. 토큰 자체는 단순히 식별자 역할만 하고, 서버는 이를 기반으로 DB에서 관련 정보를 조회한다.

#### **예시**

```
refresh_5f8f8c44eeb5b8a3c3f98f6e36d8b9a5
```

서버 저장소 예시 (Redis, DB 등):

|**Token 값**|**userId**|**expiresAt**|
|---|---|---|
|refresh_5f8f8…|123|2025-07-01T12:00:00Z|

### **장점**

- **보안성 우수**: 토큰 탈취 시에도 토큰 자체에는 정보가 없어 노출 피해 감소
    
- **무효화 용이**: 토큰 삭제 또는 만료 시간 변경을 통해 유연한 관리 가능
    
- **세밀한 제어**: 특정 기기, 세션별로 개별 토큰 관리 가능
    

  

### **단점**

- **Stateful 필요**: 토큰 검증을 위해 서버 측 저장소 필요 (DB, Redis 등)
    
- **확장성 문제**: 대규모 트래픽 처리 시 저장소 부하 고려 필요
    
- **Latency 증가 가능성**: 저장소 조회로 인한 응답 지연 발생 가능
    

  

### **실무 사용 예시**

- **웹 기반 서비스**: Refresh Token을 HttpOnly 쿠키로 저장하고 서버에서 DB 검증
    
- **보안 민감 서비스**: 금융, 헬스케어 등 정보 유출 위험이 높은 분야에서 사용
    

---

## **두 방식 비교**

|**항목**|**JWT 기반 Refresh Token**|**Opaque Refresh Token**|
|---|---|---|
|유효성 검사|서명 검증으로 자체 확인|서버 저장소에서 조회|
|서버 저장소|불필요 (Blacklist 제외)|필수 (DB/Redis 등)|
|토큰 탈취 시 피해|정보 포함 가능성 있음|정보 없음|
|로그아웃/무효화 처리|블랙리스트 필요|저장소에서 삭제 가능|
|확장성|우수 (Stateless)|저장소 부하 고려 필요|
|구현 복잡도|상대적으로 낮음|세션 관리 필요|
|실무 활용 분야|모바일, 마이크로서비스|웹, 보안 민감 서비스|

---

## **실무 적용 시 고려사항**

- **보안 우선 시**: Opaque 토큰이 적합. 특히 사용자 인증, 권한 제어가 민감한 환경에서는 DB 기반 관리가 권장됨
    
- **성능과 확장성 우선 시**: JWT 방식이 적합. 인증 서버와 리소스 서버 분리 구조에서 부하 최소화 가능
    
- **하이브리드 방식도 고려 가능**:
    
    - Access Token은 JWT
        
    - Refresh Token은 Opaque 방식 (DB 저장 및 관리)
        
    