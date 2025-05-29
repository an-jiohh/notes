# HttpSession

HttpSession은 Java EE (또는 Jakarta EE) 기반 웹 애플리케이션에서 **클라이언트의 상태 정보를 서버에서 유지하기 위한 방법**입니다. 주로 로그인 정보, 장바구니 정보 등 **세션 기반 사용자 데이터**를 저장하고 관리하는 데 사용됩니다.

---

## **✅ 1. HttpSession 개요**

- HttpSession은 javax.servlet.http.HttpSession 인터페이스에 정의되어 있으며, 서블릿 컨테이너가 제공함
- 클라이언트(브라우저)마다 고유한 세션 ID(JSESSIONID)를 부여하여 서버에 상태 정보를 유지   

---

## **✅ 2. 생성과 사용**

```
HttpSession session = request.getSession(); // 기존 세션이 없으면 새로 생성
HttpSession session = request.getSession(true); // 명시적으로 새로 생성
HttpSession session = request.getSession(false); // 세션이 없으면 null 반환
```

---

## **✅ 3. 주요 메서드**

|**메서드**|**설명**|
|---|---|
|getId()|세션 고유 ID (JSESSIONID) 반환|
|getAttribute(String name)|세션에 저장된 값 조회|
|setAttribute(String name, Object value)|세션에 데이터 저장|
|removeAttribute(String name)|세션 데이터 삭제|
|invalidate()|세션 무효화 (로그아웃 등)|
|getCreationTime()|세션 생성 시간 반환|
|getLastAccessedTime()|마지막 요청 시간 반환|
|getMaxInactiveInterval()|세션 유지 시간 설정 (초 단위)|
|setMaxInactiveInterval(int interval)|세션 유지 시간 변경|
|isNew()|클라이언트와의 첫 번째 세션인지 여부 확인|

---

## **✅ 4. 동작 원리**

1. 사용자가 처음 요청 → 서버는 세션을 생성하고 JSESSIONID라는 쿠키로 클라이언트에 전달
2. 이후 사용자의 요청에 JSESSIONID 쿠키가 포함됨 → 서버는 해당 세션을 조회
3. 서버는 세션 ID를 키로, 데이터(객체)를 메모리에 저장
4. 지정된 시간이 지나거나, 명시적으로 invalidate()하면 세션이 제거됨

---

## **✅ 5. 세션 저장소**

- 기본적으로는 **서버 메모리 (JVM Heap)**에 저장
- 대규모 서비스에서는 **세션 클러스터링** 또는 **Spring Session + Redis**를 사용하여 세션을 분산 저장
    

---

## **✅ 6. 실무에서의 사용 예시**

  

### **✅ 로그인 처리**

```
session.setAttribute("loginUser", userDto);
```

### **✅ 로그인 여부 확인**

```
UserDto user = (UserDto) session.getAttribute("loginUser");
if (user == null) {
    response.sendRedirect("/login");
}
```

### **✅ 로그아웃 처리**

```
session.invalidate(); // 세션 무효화
```

---

## **✅ 7. Spring MVC에서 사용하기**

  

Spring에서는 다음과 같이 사용 가능:

```
@GetMapping("/home")
public String home(HttpSession session) {
    User user = (User) session.getAttribute("user");
    return "home";
}
```

또는 @SessionAttributes / @SessionAttribute로 선언적으로 사용:

```
@SessionAttribute(name = "user", required = false)
public String home(User user) {
    ...
}
```

---

## **✅ 8. 주의사항**

|**항목**|**설명**|
|---|---|
|세션 과다 사용 주의|많은 데이터를 세션에 저장하면 서버 메모리 사용량 급증|
|브라우저 닫기와 무관|세션은 브라우저 닫아도 유지됨 (무작정 로그아웃 X)|
|CSRF 공격 대비 필요|세션을 기반으로 한 로그인 시스템은 CSRF 방어 필요|
|보안 설정 필수|JSESSIONID는 HTTPS에서 Secure, HttpOnly 설정 권장|
|세션 타임아웃|기본 30분 (서버 설정이나 코드에서 변경 가능)|

---

## 추가 Spring Session (확장)

- Spring의 spring-session 모듈을 통해 세션을 **Redis, JDBC, Hazelcast 등 외부 저장소로 분리 가능**
- 수평 확장이 필요한 시스템에서 필수

```
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

---

## **📌 마무리 요약**

|**핵심 개념**|**요약**|
|---|---|
|목적|클라이언트 상태를 서버에서 유지|
|주요 메서드|get/setAttribute, invalidate 등|
|동작 방식|JSESSIONID 쿠키로 세션 식별|
|주의점|메모리 관리, 보안 설정 필수|
|확장 방법|Spring Session으로 외부 저장 가능|
