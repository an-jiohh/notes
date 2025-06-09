
# **클라이언트가 JWT를 저장하는 위치 (LocalStorage vs Cookie)**

  

JWT는 클라이언트에서 저장하고, 이후 인증 요청마다 포함시켜야 하는 구조.

대표적인 저장 방식으로는 localStorage와 cookie가 있으며, 보안과 편의성 측면에서 서로 장단점이 존재.

저장 방식의 선택은 단순 구현 여부가 아닌 **보안 요건, 서비스 구조, 인증 흐름**에 따라 결정됨.

결론 선입력  : **HTTP-Only + cookie**를 사용하자

---

## **1. localStorage**

  

브라우저의 로컬 저장소.

도메인 단위로 데이터를 저장하며, JavaScript에서 자유롭게 접근 가능.

  

### **특징**

- 자바스크립트에서 localStorage.getItem() 또는 setItem()으로 접근 가능
    
- 브라우저를 닫아도 데이터 유지
    
- Authorization 헤더에 토큰을 수동으로 포함해야 함
    

  

### **장점**

- 구현이 단순하고 제어가 직관적
    
- 요청마다 Authorization 헤더를 구성하므로 CSRF에 안전
    
- 브라우저의 도메인 간 정책과 잘 호환됨
    

  

### **단점**

- XSS(교차 스크립팅) 공격에 매우 취약
    
- JavaScript를 통한 접근 가능성으로 인한 보안 위험
    
- 클라이언트 개발자가 직접 토큰 만료 및 제거 시점 관리 필요
    

---

## **2. cookie**

  

브라우저가 자동으로 요청에 포함시켜주는 저장 방식.

서버가 응답 시 Set-Cookie를 사용하여 쿠키를 설정하면, 이후 해당 도메인으로의 요청마다 쿠키가 함께 전송됨.

  

### **특징**

- 자동 전송 특성으로 인해 인증 처리 자동화 가능
    
- HttpOnly, Secure, SameSite 옵션을 통한 보안 정책 설정 가능
    
- 서버에서 명시적으로 쿠키를 설정하는 방식으로 제어 가능
    

  

### **장점**

- HttpOnly 설정 시 JavaScript에서 접근 불가 → XSS 방어
    
- 자동 전송으로 인증 흐름 단순화 가능
    
- 보안 속성을 조합하여 다양한 공격 대응 가능
    

  

### **단점**

- CSRF에 취약 → 반드시 SameSite 또는 CSRF 토큰과 병행 필요
    
- 설정 실수가 발생할 경우 전체 도메인 또는 하위 경로에 노출 위험
    
- 일부 클라이언트 환경에서는 쿠키 기반 인증이 제한적 (예: 앱 내 WebView)
    

### **쿠키 설정 예시**

  

서버에서 응답 시 다음과 같이 쿠키를 설정

```
Set-Cookie: accessToken=JWT_STRING; 
             HttpOnly; 
             Secure; 
             SameSite=Strict; 
             Path=/; 
             Domain=example.com; 
             Max-Age=3600;
```

|**속성**|**설명**|
|---|---|
|HttpOnly|JS 접근 불가 설정|
|Secure|HTTPS 통신에서만 쿠키 전송|
|SameSite=Strict|외부 사이트 요청 시 쿠키 포함 방지 (CSRF 방어용)|
|Path=/|쿠키가 전송될 경로 범위 지정|
|Domain=example.com|쿠키 적용 도메인 명시|
|Max-Age=3600|쿠키 유효시간 (초 단위)|

---

## **비교 요약**

|**항목**|**localStorage**|**cookie**|
|---|---|---|
|접근 방식|JavaScript 직접 접근|브라우저 자동 전송|
|전송 방식|Authorization 헤더 수동 포함|HTTP 요청 시 자동 포함|
|지속성|브라우저 닫아도 유지|만료 시각 지정 가능|
|XSS 방어|취약|HttpOnly로 보호 가능|
|CSRF 방어|안전 (자동 전송 아님)|SameSite 필요|
|제어 유연성|클라이언트에서 자유롭게 관리|서버 또는 클라이언트에서 명시 설정|
|보안 설정 조합|없음|Secure, HttpOnly, SameSite 조합 가능|

---

## **권장 저장 방식

|**조건**|**저장 위치 권장**|
|---|---|
|서버가 인증 상태를 직접 제어해야 하는 경우|쿠키 기반 저장 (HttpOnly, Secure 설정 포함)|
|보안 위협에 강한 설계가 필요한 경우|쿠키 + SameSite + CSRF 토큰 병행|
|프론트엔드와 백엔드가 분리된 SPA 구조|localStorage 또는 cookie 중 보안 정책에 따라 선택|
|XSS 방어가 어려운 구조|localStorage 사용 비권장|
|SSR(서버 렌더링) 기반 앱 또는 백엔드 중심 인증 처리|쿠키 저장 선호|

---

## **마무리**

결론 : **HttpOnly Cookie** 를 사용하자

JWT 저장 위치 선택은 인증 흐름 전반에 영향을 주는 핵심 설계 요소.

localStorage는 구현이 쉽지만 XSS에 매우 취약하며, 쿠키는 다양한 보안 설정이 가능하지만 CSRF 방어 전략을 반드시 병행해야 함.


장단점을 파악하였을때 사실 모바일 앱을 제외한 웹 환경에서는 localStorage 사용하지 않는 것이 좋아보임
