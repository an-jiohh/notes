# **OncePerRequestFilter**

  

OncePerRequestFilter는 Spring Framework에서 제공하는 **HTTP 요청당 한 번만 실행되는 필터**를 만들기 위한 추상 클래스이다.

Spring Security 설정 시 인증, 인가, 로깅, JWT 검증 등의 **공통 처리를 필터 체인 상에서 구현할 때** 자주 사용된다.

---

## **등장 배경**

서블릿 필터는 요청이 DispatcherServlet을 여러 번 통과하거나, 포워딩/에러 페이지 등으로 인해 **하나의 요청에 대해 여러 번 실행될 수 있다**.

이런 상황에서 **같은 로직이 중복 실행되는 것을 방지하기 위해** OncePerRequestFilter가 등장했다.

---

## **동작 방식**

  

### **핵심 메서드**

```
@Override
protected abstract void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException;
```

- 개발자는 doFilterInternal() 메서드만 구현하면 된다.
    
- OncePerRequestFilter는 내부적으로 요청 속성(request attribute)을 사용하여 **이미 필터가 실행되었는지 체크**한다.
    
- 실제 doFilter()는 오버라이드되어 있으며, **중복 호출을 차단하고 doFilterInternal()만 한 번 실행되도록 보장**한다.
    

  

### **중복 방지 로직**

```
String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();
if (request.getAttribute(alreadyFilteredAttributeName) != null) {
    // 이미 실행된 경우 필터 건너뜀
    filterChain.doFilter(request, response);
    return;
}
```

---

## **주요 특징**

|**특징**|**설명**|
|---|---|
|**단일 실행 보장**|한 요청(Request)당 한 번만 실행됨 (에러 페이지, 포워딩 포함해도 중복 실행 안됨)|
|**추상 클래스**|직접 구현체에서 doFilterInternal()만 오버라이딩하면 됨|
|**실행 순서 제어 가능**|Spring Security FilterChain 내에서 @Order 혹은 SecurityFilterChain 등록 순서로 조정 가능|

---

## **실무 사용 예시**

  

### **JWT 인증 필터**

```
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException {

        String token = resolveToken(request);
        if (token != null && jwtUtil.validate(token)) {
            Authentication auth = jwtUtil.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }

        filterChain.doFilter(request, response);
    }
}
```

---

## **주의사항**

- OncePerRequestFilter는 **HttpServletRequest/Response 기반**이므로 서블릿 환경에서만 동작함.
    
- 필터 순서를 조정할 경우 FilterRegistrationBean 또는 SecurityFilterChain의 필터 체인 위치를 명확히 이해해야 함.
    
- 동일한 요청을 내부적으로 여러 번 처리해야 하는 경우(FORWARD, INCLUDE 등)라면 로직이 한 번만 실행된다는 점을 주의해야 함.
    