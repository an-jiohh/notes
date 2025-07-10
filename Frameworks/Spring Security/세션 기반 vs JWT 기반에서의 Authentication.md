# 세션 기반 vs JWT 기반에서의 UsernamePasswordAuthenticationToken

세션 기반, JWT 기반에서의 UsernamePasswordAuthenticationToken이 헷갈려서 흐름별로 정리하고자 하였다.  

---

## **세션 기반 인증 과정에서 Authentication  리턴 흐름**

  

### **예시 코드 흐름 (로그인 요청 처리)**

```java
// 1. 인증 요청 객체 생성 (인증 전 상태)
UsernamePasswordAuthenticationToken authRequest =
    new UsernamePasswordAuthenticationToken(username, password);

// 2. 인증 시도 (AuthenticationManager가 내부적으로 AuthenticationProvider 호출)
Authentication authentication = authenticationManager.authenticate(authRequest);

// 3. 인증 성공 시 반환된 authentication 객체는 인증 완료된 상태
SecurityContext context = SecurityContextHolder.createEmptyContext();
context.setAuthentication(authentication);
SecurityContextHolder.setContext(context);

// 4. SecurityContext가 세션에 저장됨 → 이후 자동 로그인 처리
request.getSession().setAttribute(
    HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY,
    context
);
```
### **AuthenticationManager.authenticate(...)**


내부적으로 AuthenticationProvider를 호출하여 인증을 수행

#### **내부 구조 요약**

```
Authentication authenticate(Authentication authentication) {
    // 인증을 지원하는 provider 찾기
    for (AuthenticationProvider provider : providers) {
        if (provider.supports(authentication.getClass())) {
            return provider.authenticate(authentication); // 인증 수행
        }
    }
}
```

→ 보통은 DaoAuthenticationProvider가 호출되어 DB에서 사용자 정보를 확인하고

UsernamePasswordAuthenticationToken(인증 완료 상태)을 반환합니다.

### **그림으로 정리**

```
[로그인 요청]
   ↓
UsernamePasswordAuthenticationToken("jiohh", "1234")   ← 인증 전
   ↓
authenticationManager.authenticate(token)
   ↓
DaoAuthenticationProvider → 사용자 검증
   ↓
UsernamePasswordAuthenticationToken(userDetails, null, authorities) ← 인증 후
   ↓
SecurityContextHolder.getContext().setAuthentication(authentication)
   ↓
세션에 SecurityContext 저장
```


---
## JWT 기반 인증 과정에서 Authentication  리턴 흐름

### **인증 과정 요약 (로그인 시)**

1. 로그인 요청 → UsernamePasswordAuthenticationToken (인증 전) 생성
    
2. AuthenticationManager.authenticate() → 사용자 검증
    
3. 인증 성공 → JWT 생성 (토큰에 사용자 ID, 권한 등 포함)
    
4. JWT를 클라이언트에 전달 (Authorization: Bearer {token})

### **이후 요청 처리**

1. 클라이언트가 JWT를 포함하여 요청
    
2. 커스텀 필터(JwtAuthenticationFilter)에서 토큰 파싱
    
3. JwtPayload를 이용해 UsernamePasswordAuthenticationToken (인증 후) 생성
    
4. SecurityContextHolder에 저장 → 요청 처리 시 인증된 사용자로 인식
    
### **그림 요약**

```
로그인 요청
  ↓
[UsernamePasswordAuthenticationToken (id, pw)] 생성
  ↓
AuthenticationManager.authenticate()
  ↓
[인증된 사용자 정보] → JWT 생성 후 클라이언트에 전달
  ↓
요청 시 JWT 헤더 포함 → 필터에서 파싱
  ↓
[UsernamePasswordAuthenticationToken (인증 후)] 수동 생성 → SecurityContextHolder 저장
```


---

## **차이점 요약**

|**구분**|**인증 요청 시 역할**|**인증 완료 후 역할**|**재사용 방식**|
|---|---|---|---|
|세션|로그인 ID/PW 전달용|인증된 사용자 정보 보관|세션에서 자동 로딩|
|JWT|로그인 ID/PW 전달용|JWT 발급만 하고 SecurityContext 저장 안함|매 요청마다 토큰 파싱 후 직접 생성|


---
