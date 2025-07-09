# **@AuthenticationPrincipal**

  

Spring Security에서 인증된 사용자 정보를 컨트롤러에서 손쉽게 주입받기 위해 사용하는 애너테이션이 @AuthenticationPrincipal이다. 이 애너테이션은 스프링 시큐리티의 SecurityContext에 저장된 Authentication 객체로부터 principal 정보를 추출하여 메서드 파라미터로 전달해준다.

  

## **등장 배경**

  

기존에는 컨트롤러에서 인증된 사용자 정보를 얻기 위해 다음과 같은 방식으로 직접 SecurityContextHolder에서 꺼내야 했다.

```
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
UserDetails userDetails = (UserDetails) authentication.getPrincipal();
```

이 방식은 매번 보일러플레이트 코드가 반복되며, 테스트 코드 작성이 어려워지는 단점이 있었다. 이러한 문제를 해결하기 위해 @AuthenticationPrincipal이 도입되었다.

  

## **주요 기능**

  

### **인증된 사용자 정보 주입**

  

컨트롤러 메서드의 파라미터에 사용하면 현재 로그인한 사용자 객체를 자동으로 주입해준다.

```
@GetMapping("/profile")
public ResponseEntity<User> getProfile(@AuthenticationPrincipal User user) {
    return ResponseEntity.ok(user);
}
```

### **커스텀 UserDetails 클래스 사용 가능**

  

사용자가 구현한 커스텀 UserDetails 클래스가 있을 경우, 해당 클래스를 그대로 주입받을 수 있다.

```
public class CustomUserDetails implements UserDetails {
    private final Long userId;
    private final String email;
    // 생략

    public Long getUserId() {
        return userId;
    }
}
```

```
@GetMapping("/me")
public ResponseEntity<Long> getMyId(@AuthenticationPrincipal CustomUserDetails user) {
    return ResponseEntity.ok(user.getUserId());
}
```

### **특정 필드만 추출할 수도 있음**

  

Spring 5.0 이후부터는 SpEL(Spring Expression Language)을 통해 Authentication.getPrincipal() 객체에서 특정 필드만 주입할 수 있다.

```
@GetMapping("/email")
public ResponseEntity<String> getEmail(
    @AuthenticationPrincipal(expression = "email") String email) {
    return ResponseEntity.ok(email);
}
```

## **동작 원리**

  

@AuthenticationPrincipal은 HandlerMethodArgumentResolver를 통해 작동한다. 스프링 시큐리티는 이 애너테이션이 붙은 파라미터가 있을 경우, 현재 SecurityContext에 저장된 Authentication 객체에서 principal을 추출하여 주입한다.

```
SecurityContextHolder.getContext()
   → Authentication
      → getPrincipal() → 파라미터 주입
```

## **인증되지 않은 경우 처리**

  

인증되지 않은 사용자가 접근할 경우 principal은 익명 사용자("anonymousUser" 문자열 또는 null)가 되며, 이 상태에서 @AuthenticationPrincipal을 사용하면 예외가 발생할 수 있다.

  

예외 발생을 방지하려면 다음과 같이 required = false를 명시하여 null 주입을 허용할 수 있다.

```
@GetMapping("/optional")
public ResponseEntity<?> getOptionalUser(@AuthenticationPrincipal(required = false) CustomUserDetails user) {
    if (user == null) return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    return ResponseEntity.ok(user.getUserId());
}
```

## **실무 적용 시 유의 사항**

- @AuthenticationPrincipal을 통해 주입되는 객체는 UserDetailsService에서 반환한 principal이다.
    
- JWT 기반 인증을 사용하는 경우에도 UsernamePasswordAuthenticationToken에 주입된 principal 객체를 기준으로 작동하므로 커스텀 구현이 중요하다.
    
- REST API에서 익명 사용자를 허용하는 엔드포인트라면 required = false 설정이 필요하다.
    

  

## **요약**

|**항목**|**설명**|
|---|---|
|주요 기능|인증된 사용자 정보 자동 주입|
|장점|보일러플레이트 제거, 테스트 용이|
|사용 대상|커스텀 UserDetails 포함|
|비로그인 처리|required = false로 대응|
|동작 기반|SecurityContext → Authentication → principal|
