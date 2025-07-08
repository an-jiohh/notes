
# **SecurityContext를 세션에 사용하는 이유**

```
Authentication authentication = loginResult.get();
SecurityContext context = SecurityContextHolder.createEmptyContext();
context.setAuthentication(auth);
session.setAttribute("SPRING_SECURITY_CONTEXT", context);
```

해당 코드를 보며, “왜 단순히 User 객체를 세션에 저장하지 않고, 굳이 SecurityContext라는 구조로 감싸서 넣는 걸까?” 라는 의문이 들었다. 이에 대해 Spring Security의 설계 의도를 이해하고 정리한 내용은 다음과 같다.

---

## **1. 인증 상태와 권한을 포함한 보안 정보 캡슐화**

  

SecurityContext는 단순 사용자 정보가 아니라, 인증 여부, 사용자 권한, 인증 방식 등 **보안과 관련된 모든 상태를 포함한 컨텍스트**이다.

이 정보는 다음과 같은 객체 구조로 구성된다:

- Authentication 객체
    
    - principal (사용자 정보)
        
    - authorities (권한 목록)
        
    - authenticated (인증 여부)
        
    - credentials 등
        
    

단순히 User 객체만 저장한다면, 인증의 성공 여부나 권한 같은 **보안과 직결된 정보**를 프레임워크 차원에서 일관되게 처리하기 어렵다.

---

## **2. 보안 처리를 위한 일관된 접근 지점 제공**

  

Spring Security는 인증 정보를 다음과 같이 다양한 계층에서 활용한다:

- 서블릿 필터 (SecurityFilterChain)
    
- 서비스 계층 (@PreAuthorize, @AuthenticationPrincipal)
    
- 컨트롤러, 인터셉터 등
    

이때 SecurityContextHolder.getContext().getAuthentication() 한 줄로 현재 인증 정보를 어디서든 접근 가능하도록 하여, **인증 주체에 대한 전역적이고 일관된 인터페이스**를 제공한다.

### **저장소의 차이를 추상화**


Spring Security는 환경이나 설정에 따라 인증 정보를 저장하는 방식이 달라질 수 있습니다

|**실행 환경**|**인증 정보 저장 방식**|
|---|---|
|서블릿 (Spring MVC)|세션 기반 (HttpSession)|
|리액티브 (WebFlux)|비동기 Context (ServerWebExchange)|
|테스트 코드|ThreadLocal 혹은 Mock 객체|
|비동기 작업|InheritableThreadLocal, 전달 전략 변경 가능|

→ 저장 방식이 바뀌든, 환경이 바뀌든 **코드 자체는 변하지 않음**

즉 코드로만 보았을때는 SecurityContextHolder.getContext().getAuthentication() 한줄이지만 저장소에 따라 추상화된 객체가 처리해서 전달해줌

---

## **3. 테스트 및 실행 환경에서의 제어 용이**

  

보안 흐름을 테스트하거나 임의로 인증 정보를 설정할 때 다음과 같은 방식으로 직접 주입할 수 있다:

```
SecurityContext context = SecurityContextHolder.createEmptyContext();
context.setAuthentication(mockAuthentication);
SecurityContextHolder.setContext(context);
```

이는 단순히 세션에 User 객체를 저장했다면 불가능한 방식이며, **프레임워크 전반의 인증 흐름을 프로그래밍적으로 제어**할 수 있게 한다.

---

## **4. 실행 환경 및 인증 방식의 다양성을 포괄하는 추상화**

  

SecurityContext는 Servlet 기반 환경뿐 아니라 다음과 같은 다양한 인증 구조에서도 사용된다:

- JWT 기반 인증 (세션 없음)
    
- OAuth2 인증 흐름
    
- Reactive 환경 (WebFlux)
    

이처럼 보안 컨텍스트를 하나의 추상 구조로 정립해두면, 환경이 바뀌거나 인증 방식이 달라져도 **일관된 API와 구조**를 유지할 수 있다.
