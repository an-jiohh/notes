# **UsernamePasswordAuthenticationToken**


UsernamePasswordAuthenticationToken은 Spring Security에서 **인증(Authentication) 객체를 나타내는 주요 구현 클래스** 중 하나로, **사용자의 인증 정보를 담기 위해 사용**된다.
  

Spring Security는 Authentication 인터페이스를 통해 사용자의 인증 정보를 관리하며,

UsernamePasswordAuthenticationToken은 이 인터페이스를 구현한 가장 일반적인 클래스이다.

```
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
    private final Object principal;
    private Object credentials;
}
```

-> Authentication  
	-> AbstractAuthenticationToken  
		-> UsernamePasswordAuthenticationToken  
		
Authentication의 구현체 중 하나이다.

---
## **사용 목적**

- 인증을 시도할 때(인증 전): 사용자 ID와 비밀번호를 담는 용도로 사용
    
- 인증이 완료된 후(인증 후): 사용자 정보(주로 UserDetails)와 권한을 담는 용도로 사용

인정 전과 인증 후 상태가 다르게 된다.

## **생성자**

  

### **1. 인증 전 생성자**

```
new UsernamePasswordAuthenticationToken(principal, credentials);
```

- principal: 사용자 ID (String userId 등)
    
- credentials: 비밀번호
    
- 이 경우 isAuthenticated()는 false 상태로 시작함
    

  

### **2. 인증 후 생성자**

```
new UsernamePasswordAuthenticationToken(principal, credentials, authorities);
```

- principal: 보통 UserDetails 또는 사용자 객체
    
- credentials: 비밀번호 (보통은 null 처리)
    
- authorities: 사용자의 권한 목록
    
- 이 경우 isAuthenticated()는 true로 설정됨
    

---

## **주로 사용되는 시점과 흐름**

|**시점**|**사용 위치**|**역할**|
|---|---|---|
|인증 시도 전|로그인 필터 내부 (예: UsernamePasswordAuthenticationFilter)|사용자 입력값을 바탕으로 인증 요청 생성|
|인증 완료 후|AuthenticationProvider 또는 Custom Filter에서|인증된 사용자 정보와 권한을 담아 SecurityContext에 저장|


예:

```
UsernamePasswordAuthenticationToken authRequest =
    new UsernamePasswordAuthenticationToken(username, password);
```

또는 JWT 인증에서:

```
UsernamePasswordAuthenticationToken authentication =
    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
```

---

## **필드 설명**

|**필드명**|**설명**|
|---|---|
|principal|사용자 식별 정보 (ID 또는 사용자 객체 등)|
|credentials|인증 수단 (비밀번호 등, 보통 인증 후에는 null 처리)|
|authorities|권한 정보 (GrantedAuthority 리스트)|
|authenticated|인증 여부 (내부적으로 설정됨)|

---

## **SecurityContext에 저장되는 구조**

```
SecurityContext context = SecurityContextHolder.createEmptyContext();
context.setAuthentication(authentication);
SecurityContextHolder.setContext(context);
```

UsernamePasswordAuthenticationToken이 위 코드에서 사용되며, 이 객체가 SecurityContextHolder에 저장되어 전역적으로 접근 가능하게 된다.

---

## **주의 사항**

- 인증 후에는 credentials는 **null로 설정하는 것이 안전** (보안상의 이유)
    
- 인증 전과 후에 **동일한 클래스**를 사용하지만 내부 상태가 다름
    
- isAuthenticated()를 임의로 true로 설정하면 AuthenticationManager가 이를 거부할 수 있음
    

---

## **인증 전, 인증 후 상세**

인증 전과 인증 후가 어떻게 다른지 의문이 들었다.  
### **인증 전 (Unauthenticated 상태)**

#### **사용 위치**

- 사용자가 로그인 시도할 때
    
- 사용자의 입력값(아이디, 비밀번호)을 담아서 **인증 요청** 객체로 사용됨
    

### **생성자**

```
new UsernamePasswordAuthenticationToken(principal, credentials);
```

- principal: 사용자 ID (예: "jiohh")
    
- credentials: 비밀번호 (예: "1234")
    
- authorities: 없음
    
- authenticated: false (기본값)
    

### **예시**

```
UsernamePasswordAuthenticationToken authRequest =
    new UsernamePasswordAuthenticationToken("jiohh", "1234");
```

이 객체는 AuthenticationManager (또는 AuthenticationProvider)에 전달되어 실제 인증을 수행함.

```
Authentication authenticate = authenticationManager.authenticate(authRequest);
```
AuthenticationManager을 통해 인증 메소드로 넘기게 되면 내부의 UserDetailsService를 통해 인증을 진행하게 된다.  
이후 인증 후 Authentication(UsernamePasswordAuthenticationToken)를 반환하게 된다.  

---

## **인증 후 (Authenticated 상태)**


### **사용 위치**

- 인증이 성공된 후
    
- 사용자 정보를 담아 **SecurityContext에 저장**할 때 사용됨
    
### **생성자**

```
new UsernamePasswordAuthenticationToken(principal, credentials, authorities);
```

- principal: 보통 UserDetails 객체 또는 사용자 객체
    
- credentials: 보안상 이유로 보통 null 처리
    
- authorities: 인증된 사용자의 권한 리스트
    
- authenticated: true (자동 설정)
    

### **예시**

```
UsernamePasswordAuthenticationToken authentication =
    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
```

이 객체는 SecurityContextHolder.getContext().setAuthentication(authentication) 으로 등록되어 전역 인증 정보가 됨.