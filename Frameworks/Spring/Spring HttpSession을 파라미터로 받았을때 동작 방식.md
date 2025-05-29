
#  Spring HttpSession을 파라미터로 받았을때 동작 방식



---

## **❓ HttpSession은 누가 만들고 언제 만들어질까?**


Spring MVC 컨트롤러에서 다음과 같이 HttpSession을 파라미터로 선언할 수 있습니다.

```
@PostMapping("/login")
public ResponseEntity<ApiResponseDto<LoginResponseDto>> login(
        @RequestBody LoginRequestDto loginRequestDto,
        HttpSession session) {
    ...
}
```

이때 우리는 new HttpSession()을 호출한 적이 없는데도 session 객체가 있습니다.  
(또는 request.getSession()). 
  
### **✔️ 이유는?**

HttpSession 파라미터가 있으면 Spring 내부적으로 다음 코드가 실행

```
HttpSession session = request.getSession();
```

즉, 컨트롤러 진입 시점에 **request.getSession(true)가 호출되며 자동으로 세션이 생성**됩니다.

---

## **❗ 만약 로그인 실패 시 예외가 나면 세션은 어떻게 될까?**

  

예시 코드를 보면 다음과 같은 흐름입니다:

```
@PostMapping("/login")
public ResponseEntity<ApiResponseDto<LoginResponseDto>> login(
        @RequestBody LoginRequestDto loginRequestDto,
        HttpSession session) {

    Optional<LoginResponseDto> loginResult = userService.login(loginRequestDto);
    if (loginResult.isPresent()) {
        session.setAttribute("userId", dto.getUserId());
        return ResponseEntity.ok(ApiResponseDto.success(dto));
    } else {
        throw new InvalidCredentialsException();
    }
}
```

이 코드에서 loginResult가 비어 있을 경우 예외가 발생하게 되죠.
**그런데 이미 세션은 생성된 상태**입니다.

---

### **🔍 결론**

  

❗ 예외가 발생하더라도 세션은 남아 있습니다.

  

- 세션은 컨트롤러에 진입하는 순간 생성되며
- 이후 throw가 발생해도 세션은 무효화되지 않음
- 응답 시 클라이언트에게 **JSESSIONID 쿠키가 전달됨**
- 즉, **로그인 실패했는데도 세션이 생성되어 남게 되는 현상**
    

---

## **🚨 이게 왜 문제일까?**

1. 불필요한 세션 객체가 서버에 남게 됨
2. 클라이언트가 이후 요청마다 JSESSIONID를 계속 보냄
3. 서버 리소스 낭비 + 불필요한 인증 흐름 복잡도 증가
4. 보안적으로도 혼란을 줄 수 있음 (로그인 실패 상태인데 세션 존재)
    
---

## **✅ 해결 방안: 세션은 “성공했을 때만” 만들자**

```
@PostMapping("/login")
public ResponseEntity<ApiResponseDto<LoginResponseDto>> login(
        @RequestBody LoginRequestDto loginRequestDto,
        HttpServletRequest request) {

    Optional<LoginResponseDto> loginResult = userService.login(loginRequestDto);
    if (loginResult.isPresent()) {
        HttpSession session = request.getSession(true); // 명시적 생성
        LoginResponseDto dto = loginResult.get();
        session.setAttribute("userId", dto.getUserId());
        session.setAttribute("role", dto.getRole());
        session.setAttribute("name", dto.getName());
        session.setMaxInactiveInterval(60 * 30);
        return ResponseEntity.ok(ApiResponseDto.success(dto));
    } else {
        throw new InvalidCredentialsException();
    }
}
```

### **🔑 핵심 포인트**

- HttpSession을 파라미터로 선언하지 않음
- HttpServletRequest에서 getSession(true)을 호출하여 **로그인 성공시에만 세션 생성**

---

## **🔚 마무리**


💬 “나는 세션을 생성한 적이 없는데 왜 생기지?  
👉 그건 Spring이 자동으로 request.getSession()을 호출했기 때문입니다.

  
 💬 “로그인 실패했는데도 세션이 남아있어?”  
👉 컨트롤러에 진입한 순간 이미 생성된 것이고, 예외가 나도 삭제되지는 않습니다.
