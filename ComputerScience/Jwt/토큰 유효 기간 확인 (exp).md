
# **JWT 토큰 유효 기간 확인 (exp)**

  

JWT는 자체적으로 만료 시간을 포함할 수 있는 구조를 가지고 있음.

이를 통해 서버는 토큰의 유효성을 판단하고, 인증 처리 여부를 결정할 수 있음.

유효 기간 확인에 사용되는 클레임은 exp(expiration time)이며, 이는 JWT의 핵심 보안 기능 중 하나로 간주됨.

---

## **exp 클레임의 정의**

  

exp(expiration)는 토큰의 만료 시점을 나타내는 표준 클레임

Unix timestamp(초 단위)로 표현되며, 해당 시각을 초과하면 토큰은 만료된 것으로 간주됨

  

### **예시 (JWT Payload)**

```
{
  "sub": "user-1234",
  "iat": 1718000000,
  "exp": 1718003600
}
```

- iat: 발급 시간 (issued at)
    
- exp: 만료 시간 (expiration time)
    
- 위 예시에서 유효 시간은 발급 시점으로부터 3600초 (1시간)

---

## **서버에서 exp 검증 방식**

  

JWT는 클라이언트가 만료된 토큰을 포함해 요청을 보내더라도 자동으로 막히지 않음

**서버가 요청을 받을 때마다 exp 값을 검증해야 함**

  

### **검증 절차**

1. 토큰 파싱 후 Payload 추출
    
2. 현재 시각과 exp 비교
    
3. 현재 시각 > exp → 인증 실패

---

## **클라이언트에서 만료 시점 관리**

  

서버가 아닌 클라이언트에서도 만료 시점을 인지하고 사용자 경험 개선 가능

예: 자동 로그아웃, 토큰 갱신 스케줄링 등

  

### **관리 방식**

- 서버 응답 시 expiresIn 값을 함께 전달
    
- 또는 JWT 디코딩 후 exp 값 직접 추출
    

  

### **예시 (JavaScript)**

```
const payload = JSON.parse(atob(token.split('.')[1]));
const expiresAt = payload.exp * 1000;
const isExpired = Date.now() > expiresAt;
```

> 클라이언트에서는 만료 여부를 판단할 수는 있지만, 인증 거부 여부는 서버가 최종 판단함

---

## **만료 응답 예시**

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer error="invalid_token", error_description="The token is expired"
```

- 표준 형식은 RFC 6750 기반
    
- WWW-Authenticate 헤더를 통해 만료 원인 명시 가능

---

## **실무 고려사항**

|**항목**|**권장 설정**|
|---|---|
|Access Token 유효시간|짧게 (5분~30분 권장)|
|Refresh Token 유효시간|길게 (7일~30일 또는 영속적)|
|토큰 재발급 방식|만료 감지 후 Refresh Token 사용|
|클라이언트 관리 방식|expiresIn 또는 exp 기반 시간 계산|
Access Token은 탈취되더라도 위험을 줄이기 위해 만료 시간을 짧게 설정하는 것이 일반적
Refresh Token은 장기 인증을 위해 사용되며, 별도 저장소 또는 쿠키로 관리됨

---

## **정리**

  

exp 클레임은 JWT가 단순 인증 수단을 넘어 보안성을 확보하는 핵심 장치

서버는 매 요청마다 exp를 검증해야 하며, 클라이언트도 만료 시점을 활용하여 사용자 경험을 개선할 수 있음