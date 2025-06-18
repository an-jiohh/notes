
# **SecurityContextHolder와 Spring Security 인증 흐름 정리**

  

## **1. SecurityContextHolder란?**

- **Spring Security에서 현재 인증된 사용자 정보를 저장하는 전역 저장소**
    
- 내부적으로 ThreadLocal<SecurityContext> 구조로 동작
    
- 어디에서든 인증 정보 (Authentication)에 접근 가능
    

```
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
Object principal = auth.getPrincipal(); // 사용자 정보
```

---

## **2. 왜 사용하는가?**

|**이유**|**설명**|
|---|---|
|사용자 정보 일관성 유지|컨트롤러, 서비스, AOP 등 모든 계층에서 동일 방식으로 인증 정보 접근 가능|
|Spring Security 연동|@PreAuthorize, 권한 체크, 필터 체인 등 모든 보안 기능이 이 객체를 기준으로 동작|
|상태 유지와 무관하게 사용 가능|JWT 같은 무상태 인증에서도 사용할 수 있도록 설계됨|
|스레드 안전성 확보|요청별 스레드에 독립된 보안 정보 저장 (ThreadLocal 기반)|

---

## **3. request와의 차이점**

|**항목**|HttpServletRequest**로 직접 전달**|SecurityContextHolder **사용**|
|---|---|---|
|사용자 정보 전달|request.setAttribute() 등으로 수동 처리|자동으로 저장 및 관리|
|유지보수|불안정, 실수로 누락 가능|일관된 접근 방식 보장|
|스레드 안전성|명시적 제어 필요|ThreadLocal 기반으로 안전|
|Spring Security 기능 연동|안됨|가능 (인가, 권한, 필터, AOP 등 전부 연동됨)|

---

## **4. JWT와 세션 기반 인증에서의 사용 방식**

|**항목**|**세션 기반 인증**|**JWT 기반 인증**|
|---|---|---|
|인증 정보 저장 위치|HttpSession (서버)|JWT 토큰 (클라이언트)|
|SecurityContextHolder 사용 시점|Spring이 자동으로 세션에서 꺼내서 등록|커스텀 필터가 토큰을 파싱 후 등록|
|인증 정보 설정 방식|세션에서 Authentication 객체 자동 로딩|JWT 검증 후 Authentication 직접 생성 및 저장|

### **JWT 흐름 예시**

1. 요청 헤더에 JWT 포함 (Authorization: Bearer <token>)
    
2. 필터에서 토큰 검증
    
3. Authentication 객체 생성
    
4. SecurityContextHolder.getContext().setAuthentication(auth) 호출
    

---

## **5. 어디에 저장되는가?**

- 기본 저장소는 ThreadLocal
    
- 요청이 들어올 때 인증 정보를 SecurityContextHolder의 ThreadLocal에 저장
    
- 요청이 끝나면 자동으로 제거됨
    

  

### **저장 전략 종류**

|**전략 이름**|**설명**|
|---|---|
|MODE_THREADLOCAL|기본값. 요청 스레드별로 고유한 보안 정보 저장|
|MODE_INHERITABLETHREADLOCAL|부모 스레드의 보안 정보를 자식 스레드에 전달 (비동기 환경에서 사용)|
|MODE_GLOBAL|전역 공유 (권장되지 않음)|

---

## **6. @AuthenticationPrincipal와의 관계**

- SecurityContextHolder에 저장된 Authentication의 principal 값을 꺼내기 쉽게 해주는 어노테이션
    
- 컨트롤러에서 간결한 코드 작성 가능
    

```
@GetMapping("/me")
public ResponseEntity<?> getMyInfo(@AuthenticationPrincipal CustomUserDetails user) {
    return ResponseEntity.ok(user.getUsername());
}
```

---

## **7. 실무 요약**

|**상황**|**처리 방식**|
|---|---|
|인증 정보 저장|세션 또는 JWT → Authentication 생성 → SecurityContextHolder에 저장|
|사용자 정보 접근|SecurityContextHolder.getContext().getAuthentication() 또는 @AuthenticationPrincipal|
|인증 정보 소멸|요청 완료 시 자동 클리어|
|보안 일관성 유지|Spring Security 전 기능(@Secured, @PreAuthorize, 필터 등)과 완전 연동됨|

---

### **핵심 정리**

SecurityContextHolder는 **Spring Security의 인증 상태를 전역에서 안전하고 일관되게 유지하기 위한 핵심 메커니즘**으로, 세션 기반이든 JWT 기반이든 모든 인증 방식에서 **통합된 사용자 정보 관리**를 가능하게 한다.

인증 과정에서 사용자 정보 저장, 이후 꺼내오는 것까지 쉽게 할 수 있도록 해줌