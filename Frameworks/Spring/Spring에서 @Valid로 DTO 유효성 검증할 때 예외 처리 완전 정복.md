
# Spring에서 @Valid로 DTO 유효성 검증할 때 예외 처리

스프링(Spring)에서 REST API를 만들다 보면, 클라이언트의 요청 본문(JSON)을 DTO로 받고 `@Valid`로 검증하는 일이 자주 생긴다. 이때 **유효성 검증 실패를 어떻게 처리할지**에 따라 로직의 구조가 달라진다.

이 글에서는 다음 두 가지 핵심 질문에 대해 정리한다:

1. `@Valid` 사용 시 예외 처리 방법들, 각각의 장단점, 그리고 실무에서는 어떻게 쓰이는가?
2. `@Valid`에 `BindingResult`를 붙이면 왜 예외가 발생하지 않을까?

---

## ✅ @Valid 사용 시 예외 처리 방법

스프링에서 `@Valid`를 사용하면 해당 DTO에 대한 **Bean Validation(JSR-380)**이 수행된다. 이때 유효성 검사를 통과하지 못하면 **예외가 발생하거나**, 또는 **오류 정보를 수동으로 처리할 수 있다.**

이때 두가지 방법이 존재한다.
- `BindingResult`를 함께 사용하는 방식
- 예외를 이용하여 처리하는 방식 = 전역 예외 처리 (@ControllerAdvice) 방식

| 조합                        | 예외 발생 여부   | 흐름                   |
| ------------------------- | ---------- | -------------------- |
| @Valid만 사용                | ✅ 예외 발생    | 예외 핸들러로 감            |
| @Valid + BindingResult 사용 | ❌ 예외 발생 안함 | bindingResult로 수동 처리 |
두가지 방식은 예외에 따라 다르다고 볼 수 있다.  

검증 실패 시 MethodArgumentNotValidException 예외 발생하는데  
BindingResult가 인자로 있을 경우 예외정보를 넣어서 컨트롤러 단까지 실행되게 해준다.


### 방법 1. `BindingResult`를 함께 사용하는 방식

```java
@PostMapping("/api")
public ResponseEntity<?> create(@RequestBody @Valid MyDto dto, BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        List<String> errors = bindingResult.getFieldErrors().stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());
        return ResponseEntity.badRequest().body(errors);
    }
    return ResponseEntity.ok("Success");
}
````

- bindingResult.hasErrors()를 통해 수동으로 검증 실패 여부를 판단할 수 있다.
- 예외가 발생하지 않는다.

장점

- 검증 실패 이후에도 로직을 계속 수행할 수 있음
- 응답 메시지를 자유롭게 커스터마이징 가능

  
단점
- 컨트롤러 코드가 길어지고 중복될 수 있음
- 모든 메서드에 BindingResult를 추가해야 하는 불편함

---

### 방법 2. 전역 예외 처리 (@ControllerAdvice) 방식

```
@PostMapping("/api")
public ResponseEntity<?> create(@RequestBody @Valid MyDto dto) {
    // 검증 통과한 경우만 실행됨
    return ResponseEntity.ok("Success");
}
```

```
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidationExceptions(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors().stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());
        return ResponseEntity.badRequest().body(errors);
    }
}
```

- 유효성 검증 실패 시, MethodArgumentNotValidException이 자동 발생
- 예외 핸들러에서 공통적으로 처리 가능

  

장점
- 컨트롤러가 간결하고 깔끔함
- 예외 처리를 한 곳에서 관리할 수 있어 유지보수가 쉬움

단점
- 검증 실패 시 로직을 계속 수행할 수 없음
- 복잡한 조건의 커스터마이징이 어려울 수 있음

---

### ✅ 실무에서는?

|상황|추천 방식|
|---|---|
|일반적인 REST API 서버|@ControllerAdvice 전역 예외 처리 사용 ✅|
|검증 실패 후에도 로직 진행이 필요한 경우|BindingResult 방식 사용|
|웹 폼 처리 (JSP, Thymeleaf 등)|BindingResult 선호|

---
## 🔚 결론

- 간단한 REST API에서는 전역 예외 처리(@ControllerAdvice) 방식이 가장 많이 쓰인다.
- 복잡한 조건 제어, 로직 분기 등이 필요하면 BindingResult를 고려할 수 있다.
- 두 방법의 차이를 알고 상황에 맞게 사용하는 것이 중요하다.