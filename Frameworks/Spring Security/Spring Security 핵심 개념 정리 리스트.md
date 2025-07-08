
# **Spring Security 핵심 개념 정리 리스트**

Spring Security를 적용하기 전 핵심 키워드를 확인하기 위한 리스트 정리입니다.

---

## **1. Spring Security의 목적과 역할**

|**항목**|**설명**|
|---|---|
|인증 (Authentication)|사용자가 누구인지 확인 (로그인)|
|인가 (Authorization)|사용자가 어떤 권한을 가졌는지 판단 (접근 통제)|
|기타 보안 기능|CSRF 보호, 세션 고정 방지, CORS 설정, HTTPS 강제, HTTP 헤더 보안 설정 등|

---

## **2. 인증 흐름 핵심 컴포넌트**

  

> 로그인 요청부터 인증 완료까지 처리하는 구성 요소

|**컴포넌트**|**설명**|
|---|---|
|AuthenticationFilter|로그인 요청을 가로채 사용자 입력 추출 (UsernamePasswordAuthenticationFilter)|
|AuthenticationManager|인증 처리의 중심, authenticate() 메서드 호출|
|AuthenticationProvider|실제 인증 로직 담당 (DaoAuthenticationProvider 등)|
|UserDetailsService|사용자 정보를 DB에서 가져오는 로직 (직접 구현 필요)|
|UserDetails|사용자 정보를 담는 객체 (username, password, roles 등)|
|SecurityContextHolder|인증된 사용자 정보를 저장하고 유지 (ThreadLocal 기반)|

---

## **3. 인가 흐름 핵심 컴포넌트**

  

> 인증된 사용자가 요청한 리소스에 접근 가능한지 판단

|**컴포넌트**|**설명**|
|---|---|
|SecurityFilterChain|모든 요청에 대해 필터 체인 적용|
|FilterSecurityInterceptor|URL 접근 권한 검사|
|AccessDecisionManager|실제 권한 판단을 내리는 결정자|
|GrantedAuthority|권한을 나타내는 객체 (예: ROLE_USER)|
|@PreAuthorize, .hasRole(...)|메서드나 URL 수준의 접근 제어|

---

## **4. 주요 설정 클래스**

|**클래스**|**역할**|
|---|---|
|SecurityFilterChain|필터 체인을 Bean으로 등록하여 보안 정책 구성 (현행 방식)|
|WebSecurityConfigurerAdapter|예전 방식 (5.7부터 deprecated)|
|AuthenticationManagerBuilder|사용자 정보 및 비밀번호 인코딩 설정|
|HttpSecurity|인증, 인가, 필터, 세션 등 전체 보안 설정을 구성하는 핵심 객체|
|WebSecurityCustomizer|보안 필터 대상에서 제외할 경로 지정 (/static/** 등)|

---

## **5. 인증 방식 (전환 가능)**

| **방식**     | **설명**                                                   |
| ---------- | -------------------------------------------------------- |
| Form Login | 웹 페이지 기반 로그인 (formLogin())                               |
| HTTP Basic | 요청 헤더에 ID/PW 포함 (테스트용 또는 API 기본 방식)                      |
| Session 기반 | 로그인 성공 시 세션(JSESSIONID) 생성                               |
| JWT 기반     | 토큰 발급 및 stateless 인증, 직접 구현 필요                           |
| OAuth2     | 소셜 로그인 (구글, 네이버 등), spring-boot-starter-oauth2-client 필요 |

---

## **6. 필터 체인 구성**

  

> Spring Security의 핵심: 요청은 수십 개의 보안 필터를 통과함

  

대표 필터:

|**필터 이름**|**설명**|
|---|---|
|SecurityContextPersistenceFilter|이전 요청에서 저장된 인증 정보 불러오기|
|UsernamePasswordAuthenticationFilter|로그인 요청을 가로채 인증|
|ExceptionTranslationFilter|예외 발생 시 처리 (401, 403 등)|
|FilterSecurityInterceptor|최종적으로 권한 판단 수행|

---

## **7. 기본 컴포넌트 구현체**

|**인터페이스**|**기본 구현체**|**역할**|
|---|---|---|
|AuthenticationProvider|DaoAuthenticationProvider|DB 기반 인증|
|UserDetailsService|직접 구현 필요|사용자 조회|
|UserDetails|org.springframework.security.core.userdetails.User|사용자 정보 모델|
|PasswordEncoder|BCryptPasswordEncoder|비밀번호 암호화|

---

## **8. 세션과 상태 관리**

|**정책**|**설명**|
|---|---|
|SessionCreationPolicy.ALWAYS|요청마다 새 세션 생성|
|SessionCreationPolicy.IF_REQUIRED|필요한 경우만 생성 (기본)|
|SessionCreationPolicy.STATELESS|세션 사용 안 함 (JWT에 적합)|

---

## **9. 자주 쓰는 설정 메서드 (HttpSecurity기준)**

```
http
  .authorizeHttpRequests()
  .requestMatchers("/admin/**").hasRole("ADMIN")
  .anyRequest().authenticated()

  .and().formLogin().loginPage("/login")

  .and().logout().logoutUrl("/logout")

  .and().csrf().disable()

  .and().sessionManagement().sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
```

---

## **10. 커스터마이징 포인트**

|**목적**|**방법**|
|---|---|
|DB 인증 연결|UserDetailsService 직접 구현|
|사용자 정보 확장|UserDetails 구현체를 직접 작성|
|필터 확장|addFilterBefore(...)로 커스텀 필터 추가|
|JWT 로그인|필터 + Provider + Token 유틸 직접 구현|
|로그인 응답 커스터마이징|AuthenticationSuccessHandler, FailureHandler 구현|

---

## **11. 테스트 및 디버깅 팁**

|**항목**|**방법**|
|---|---|
|로그인된 사용자 확인|SecurityContextHolder.getContext().getAuthentication()|
|인증 여부 확인|Authentication.isAuthenticated()|
|사용자 이름|authentication.getName()|
|역할 확인|authentication.getAuthorities()|

---
## **기타. Spring Security 버전별 주의점**

|**버전**|**주요 변경 사항**|
|---|---|
|5.4|SecurityFilterChain 방식 도입 (람다 기반 설정 가능)|
|5.7|WebSecurityConfigurerAdapter deprecated (사용 중단)|
|6.0+|Spring Boot 3.x에서 기본 방식 → SecurityFilterChain + Bean 구성 방식만 지원|
