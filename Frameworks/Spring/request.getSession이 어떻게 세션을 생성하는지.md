# request.getSession이 어떻게 세션을 생성하는지

 “HttpServletRequest는 요청 정보만 들고 있는 객체 아닌가? 
  어떻게 request.getSession()이 서버의 메모리나 세션 저장소와 상호작용해서 세션을 생성할까?

  
이건 **서블릿 컨테이너의 요청 처리 파이프라인과 DI 구조**를 이해하면 명확해집니다.

---

## **🔧 먼저 요약 정리**

  
HttpServletRequest는 단순한 요청 정보 객체가 아니라, **서블릿 컨테이너가 제공하는 “팩사드(facade)” 객체**

이 객체는 내부적으로 서버 컴포넌트들(SessionManager 등)과 연결되어 있어서,

getSession()을 호출하면 **실제로는 내부에서 세션 관리자(SessionManager)를 호출하여 동작**

---

## **🔍 내부 구조 깊이 보기 (Tomcat 예시)**

  

### **구조 요약 (Facade Pattern)**

```
HttpServletRequest  ← 인터페이스
└── RequestFacade   ← 외부에서 접근하는 public 객체
    └── Request     ← 실제 동작하는 Tomcat 내부 구현
        └── Context       ← 해당 웹 애플리케이션 컨텍스트
            └── Manager   ← 세션 저장소 / 세션 생성 담당
```

### **예를 들어,** 

### **request.getSession()**

###  **호출 시:**

```
public class RequestFacade implements HttpServletRequest {
    private Request request;

    public HttpSession getSession() {
        return request.getSession(true);
    }
}
```

→ 내부의 request.getSession(true)는 이렇게 진행됩니다:

```
public HttpSession getSession(boolean create) {
    // 1. 이미 세션이 있는지 확인
    if (session != null && session.isValid()) {
        return session;
    }

    // 2. 쿠키에서 JSESSIONID 가져오기 → 기존 세션 조회
    Session session = context.getManager().findSession(sessionId);

    // 3. 없으면 새로 생성 (여기서 서버 자원 사용!)
    if (create) {
        session = context.getManager().createSession();
    }
}
```

> 🔥 즉, request 객체는 단순히 HTTP 데이터를 들고 있는 게 아니라, **서버 내부의 컨텍스트와 세션 매니저에 접근할 수 있도록 연결된 객체**입니다.

---

## **💡 핵심 개념: 서블릿 컨테이너의 책임**

  

### **✅** **HttpServletRequest** **는 그냥 데이터 컨테이너가 아님**

- HTTP 요청 파싱 + 다양한 서블릿 기능을 위한 **인터페이스 겸 통로**
- HttpServletRequest.getSession()을 통해 **서버 자원을 추상적으로 사용할 수 있도록 설계**됨

---

## **📌 서블릿 컨테이너가 보장하는 것**

|**항목**|**설명**|
|---|---|
|컨텍스트 관리|Request 객체는 컨텍스트(Context), 세션 매니저(Manager) 등 서버 컴포넌트를 참조함|
|세션 매니저 위임|getSession() 호출은 내부적으로 Manager.createSession() 등으로 위임됨|
|완전한 추상화|개발자는 request 객체만 써도 서버 자원까지 안전하게 접근 가능함|

---

## **🔁 예시 비유**

  
 당신은 프런트 데스크(=request)에서 “좌석(=세션)을 주세요”라고 말합니다.

프런트 데스크는 뒤에 있는 매니저(=세션 매니저)를 호출해서 좌석을 가져오거나 새로 만들어서 내줍니다.

**직접 좌석 공간을 만들지 않아도, 요청만 하면 내부적으로 처리해주는 구조

---

## **✅ 결론**

|**질문**|**답변**|
|---|---|
|HttpServletRequest는 단순 요청 정보인가요?|아닙니다. 서블릿 컨테이너 내부 객체들과 연결된 **인터페이스이자 진입점**입니다.|
|세션 생성은 왜 가능하죠?|내부적으로 컨텍스트 → 세션 매니저를 참조하고 있어서, **서버 자원에 접근하고 세션을 생성할 수 있음**|
|이것이 안전한가요?|네. 서블릿 표준은 이를 예상하고 구조화했으며, 실제 자원 접근은 컨테이너가 관리합니다.|
request에 세션을 생성할 수 있는 컨텍스트가 DI되어있어 이를 이용하여 세션에 접근하여 세션을 생성할 수 있음

---
