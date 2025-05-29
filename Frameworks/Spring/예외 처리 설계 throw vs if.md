# 예외 처리 설계: `throw` vs `if`

Spring 기반 시스템에서 인증/권한/에러 처리를 할 때  
아래 두 가지 방식 중 어떤 것이 더 좋은지에 대한 고민

---

## ✅ 핵심 원칙

| 구분         | 설명                           | 적합한 방식         |
| ---------- | ---------------------------- | -------------- |
| **비정상 흐름** | 예상은 가능하지만, 정상적으로 처리할 수 없는 상황 | `throw` 예외 던지기 |
| **정상 흐름**  | 로직 내에서 자연스럽게 발생하는 분기         | `if` 조건문 사용    |

---

## ✅ 예외 (`throw`)는 언제 쓰는가?

### ✔ 예시 상황
- 로그인 실패 (비밀번호 틀림, 유저 없음)
- 존재하지 않는 리소스 조회
- 인증되지 않은 접근
- 비즈니스 도메인 규칙 위반

### ✔ 이유
- 컨트롤러에서 에러 처리 책임을 분리할 수 있음
- 전역 예외 처리기를 통해 일관된 에러 응답 포맷 유지 가능
- 예외 클래스를 통해 상황의 의미를 명확히 전달할 수 있음

### ✔ 코드 예시

```java
User user = userRepository.findByUserId(req.getUserId())
        .orElseThrow(() -> new InvalidCredentialsException());

if (!passwordEncoder.matches(req.getPassword(), user.getPassword())) {
    throw new InvalidCredentialsException();
}
````

---

## **✅ 조건문 (if)는 언제 쓰는가?**

### **✔ 예시 상황**
- 사용자 역할에 따라 분기 처리
- 입력값이 옵션일 때 분기 처리
- 내부 로직에서 데이터 흐름에 따라 다르게 처리할 때
    

  

### **✔ 코드 예시**

```
if (user.getRole().equals("ADMIN")) {
    // 관리자 전용 로직
} else {
    // 일반 사용자 로직
}
```

---

## **❌ 잘못된 패턴 예시 (조건문으로 예외 처리)**

```
User user = userRepository.findByUserId(req.getUserId());
if (user == null || !passwordEncoder.matches(req.getPassword(), user.getPassword())) {
    return ResponseEntity.status(401).body("잘못된 사용자입니다");
}
```

- 문제점:
    
    - 컨트롤러 로직이 복잡해지고 책임이 증가
        
    - 에러 응답 포맷의 일관성 저하
        
    - 예외 상황의 의미 전달 부족
        
    

---

## **✅ 정리**

- “정상적인 흐름”은 if로 분기
    
- “예외적인 상황”은 throw로 예외 던지기
    
- 전역 예외 처리기(@RestControllerAdvice)와 커스텀 예외 클래스를 활용하면 객체지향적이고 유지보수에 강한 구조를 만들 수 있음
    

---

## **✅ 보너스: 커스텀 예외 + 전역 처리기 구성 예시**

```
// 예외 클래스
public class InvalidCredentialsException extends RuntimeException {
    public InvalidCredentialsException() {
        super("이메일 또는 비밀번호가 올바르지 않습니다");
    }
}

// 에러 응답 DTO
public class ErrorResponse {
    private String status = "error";
    private String code;
    private String message;

    public ErrorResponse(String code, String message) {
        this.code = code;
        this.message = message;
    }
}

// 전역 처리기
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(InvalidCredentialsException.class)
    public ResponseEntity<ErrorResponse> handleInvalidCredentials(InvalidCredentialsException ex) {
        return ResponseEntity
            .status(HttpStatus.UNAUTHORIZED)
            .body(new ErrorResponse("INVALID_CREDENTIALS", ex.getMessage()));
    }
}
```