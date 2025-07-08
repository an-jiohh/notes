
# **SecurityContext 스레드 바인딩



## **1. HTTP 요청은 스레드 기반으로 처리됨**

- Spring MVC 기반 웹 애플리케이션에서는 하나의 HTTP 요청이 하나의 스레드에서 처리됨
    
- 요청을 처리하는 동안, **현재 사용자(=인증 정보)**를 참조할 필요가 있음
    
- 따라서, 요청 처리에 사용하는 스레드에 인증 정보를 바인딩하면 자연스럽게 “현재 사용자 정보”에 접근할 수 있음
    

  

## **2. SecurityContextHolder와 ThreadLocal**

- SecurityContextHolder는 내부적으로 ThreadLocal<SecurityContext>를 사용
    
- 즉, 각 스레드는 **자신만의 SecurityContext**를 가짐
    
- 인증이 완료되면 해당 스레드에 SecurityContext가 저장되고,
    
    요청을 처리하는 도중에는 언제든지 SecurityContextHolder.getContext()로 접근 가능
    

  

## **3. 세션과의 역할 분리**

|**저장 위치**|**목적**|
|---|---|
|HttpSession|사용자 인증 정보를 **요청 간에 유지**|
|ThreadLocal|현재 요청을 처리하는 동안 인증 정보를 **빠르게 접근**하고 **전달**|

- 로그인 시 생성된 SecurityContext는 HttpSession에도 저장됨
    
- 다음 요청 시 필터(SecurityContextPersistenceFilter)가 세션에서 인증 정보를 꺼내와 스레드의 ThreadLocal에 저장
    
- 요청이 끝나면 ThreadLocal은 정리됨 (메모리 누수 방지를 위해)
    

  

## **4. 요청 처리 흐름 요약**

```
[클라이언트 요청] → [Spring Security FilterChain]
                           ↓
            SecurityContext 생성 및 스레드에 바인딩
                           ↓
               Controller / Service 계층
         → SecurityContextHolder.getContext()로 인증 정보 조회
                           ↓
                요청 완료 후 ThreadLocal 정리
```

## **5. 왜 굳이 ThreadLocal을 사용할까?**

|**이유**|**설명**|
|---|---|
|요청당 사용자 정보 분리|스레드마다 독립적인 컨텍스트 저장 가능|
|인증 정보 접근의 간편성|전역 메서드로 어디서든 현재 사용자 정보 조회 가능|
|스레드 기반 처리 모델과 잘 맞음|HTTP 요청이 하나의 스레드에서 처리되므로 자연스럽게 매칭됨|

---

## **결론**

  

Spring Security는 **각 HTTP 요청에 대해 생성된 스레드에 SecurityContext를 바인딩함으로써**,

요청 처리 도중 인증 정보에 안전하고 효율적으로 접근할 수 있도록 설계되어 있음.

이는 스레드 기반 처리 모델과 인증 보안 모델을 자연스럽게 연결해주는 방식임.