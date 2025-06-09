# **Authorization 헤더 사용 방식 (Bearer token)**

  

JWT 기반 인증에서 가장 일반적인 사용 방식은 HTTP 요청의 Authorization 헤더에 토큰을 포함시키는 방법.

이 방식은 토큰의 전송 경로를 명확히 하며, 서버가 인증 처리를 일관성 있게 수행할 수 있도록 해줌.

특히 REST API, GraphQL API, 모바일 앱, SPA 등 클라이언트-서버 구조에서 많이 사용됨.

---

## **Authorization 헤더란**

  

HTTP 표준에 정의된 인증 관련 헤더 필드.

클라이언트가 자신의 인증 정보를 서버에 제출할 때 사용하는 방식.

기본 인증(Basic Auth), 토큰 인증(Bearer), API Key 등 다양한 스킴이 존재함.

```
Authorization: <type> <credentials>
```

- <type>: 인증 방식 (예: Basic, Bearer, Digest)
    
- <credentials>: 인증 정보 또는 토큰
    

---

## **Bearer 스킴이란**

  

Bearer는 “소지자(bearer)가 이 자격을 가졌다고 간주”하는 방식.

즉, 토큰을 소지하고 있다는 사실만으로 인증을 수행함.

일반적으로 JWT는 Bearer 방식으로 전달됨.

  

### **기본 구조**

```
Authorization: Bearer <JWT_STRING>
```

- Bearer: 스킴 타입
    
- <JWT_STRING>: JWT 문자열 (헤더.페이로드.서명 형식)
    

---

## **사용 예시**

  

### **클라이언트 요청**

```
GET /api/user/me HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### **서버 처리 흐름**

1. Authorization 헤더 추출
    
2. Bearer 타입인지 확인
    
3. JWT 파싱 및 서명 검증
    
4. Payload에서 사용자 정보 추출
    
5. 인증 성공 시 요청 처리
    

---

## **프론트엔드 예시 (JavaScript)**

```
const token = localStorage.getItem("accessToken");

axios.get("/api/user/me", {
  headers: {
    Authorization: `Bearer ${token}`
  }
});
```

- 토큰이 존재할 경우 헤더에 수동 포함
    
- 서버는 이 헤더를 기반으로 인증 상태 판단
    

---

## **장점**

- 요청에 포함되므로 별도의 쿠키, 세션 의존 없음
    
- 서버와 클라이언트 간 인증 흐름이 명확
    
- RESTful API와 잘 어울리는 설계
    
- Cross-Origin 요청에서 쿠키 없이도 인증 가능
    

---

## **보안 고려사항**

|**위협 유형**|**방지 방법**|
|---|---|
|토큰 탈취 (XSS)|토큰을 localStorage 대신 HttpOnly Cookie에 저장하거나, 클라이언트에서 안전하게 관리|
|중간자 공격 (MITM)|HTTPS 필수 사용|
|토큰 유효성 위조|서명 검증 철저히 수행|
|재사용 공격 (Replay)|Access Token 만료시간 짧게 설정 + Refresh Token 병행 사용|

> Bearer token은 누구든지 토큰을 가지고 있다면 인증이 된다고 간주하므로, **토큰 유출에 매우 민감**

> 서버는 항상 서명 검증, 만료 확인 등 엄격한 검증 로직을 구현해야 함

---

## **인증 실패 예시 응답**

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer error="invalid_token", error_description="The token is expired"
```

- RFC 6750에서 정의된 Bearer 인증 실패 응답 형식
    
- WWW-Authenticate 헤더를 통해 추가 설명 제공 가능
    

---

## **마무리**

  

Authorization: Bearer 방식은 JWT를 사용하는 대부분의 API 인증에서 기본으로 채택되는 구조.

이 방식은 간단하면서도 명확한 인증 전달 방식을 제공하며, 서버와 클라이언트 간 인증 로직 분리에 적합함.

  

실무에서는 다음 사항을 함께 고려하는 것이 바람직함.

1. HTTPS를 통한 안전한 전송
    
2. 만료 시간(exp) 및 재발급 전략 설정
    
3. 프론트엔드에서 토큰을 안전하게 보관
    
4. 서버에서 서명 및 토큰 유효성 철저히 검증
    
5. 인터셉터 또는 필터를 통해 전역 인증 처리 구성
    

---

다음 글에서는 Spring Security, Express, FastAPI 등에서 Authorization 헤더 기반 JWT 인증을 실제로 구현하는 흐름을 다룰 수 있음.