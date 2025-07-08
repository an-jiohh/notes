
# 로그인 식별자로 String을 사용하는 이유

사실 지금까지 로그인 식별자로 Id값을 사용하여서 이를 개선하고자 작성한 문제이다.


---

  

### getUsername() 의 리턴 타입은 무조건 String**

- Spring Security는 로그인 식별자(username)를 String으로 처리하도록 설계됨
    
- UsernamePasswordAuthenticationToken(username, password) 생성자에서 username 타입이 String
    
- 따라서 UserDetails.getUsername()도 String 반환만 허용됨
    

> 즉, Long 타입의 id는 **인증에 사용할 수 없음**

---

### **로그인 식별자로**  **String** **을 사용하는 이유**

|**이유**|**설명**|
|---|---|
|**사람 중심 식별자**|사용자 ID, 이메일, 닉네임 등은 사람이 이해하고 입력하기 좋음|
|**보안상 안전**|Long id는 예측 가능하고 공격자가 쉽게 추측 가능함 (ex: IDOR)|
|**UI/UX 관점에서도 자연스러움**|로그인창에 123 같은 숫자보단 "johndoe"나 "user@example.com"이 자연스럽고 익숙함|
|**Spring Security의 인증 시스템과 일치**|내부적으로 String username으로 인증 처리되기 때문에 설계적으로 맞춰야 함|

---

### **id(Long)를 쓰고 싶다면?**

- 로그인 인증에는 **사용하지 않고**,
    
- CustomUserDetails에 포함시켜서 **인증 이후 비즈니스 로직 처리에만 사용**
    

```
public class CustomUserDetails implements UserDetails {
    private final Long id;
    private final String userId;
    ...
}
```

```
@AuthenticationPrincipal CustomUserDetails user
→ user.getId() // 인증 이후 로직에서 Long id 사용 가능
```

---

## **정리 표**

|**항목**|**타입**|**사용 목적**|**왜 String?**|
|---|---|---|---|
|username|String|인증 시 사용되는 로그인 식별자|Spring Security는 String만 허용|
|password|String|비밀번호 인증|비밀번호는 문자열 비교 기반|
|GrantedAuthority|String|권한 표현 ("ROLE_USER")|권한도 문자열로 표현됨|
|Long id|Long|DB 내부 식별자, 비즈니스 로직|인증에는 부적합 (예측 가능하고 노출 위험)|
