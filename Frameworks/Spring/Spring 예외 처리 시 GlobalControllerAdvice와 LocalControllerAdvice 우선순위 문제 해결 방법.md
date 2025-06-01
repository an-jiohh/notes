
# Spring 예외 처리 시 GlobalControllerAdvice와 LocalControllerAdvice 우선순위 문제 해결 방법

  

## **문제 상황**

  

Spring 프로젝트에서 다음과 같이 예외 처리 핸들러를 구성하였다:

- GlobalExceptionHandler: 전역 예외를 처리 (예: RuntimeException)
    
- LoginExceptionHandler: 로그인 도메인에서 발생하는 예외 처리 (InvalidCredentialsException, SessionInvalidationException 등)
    

```
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<ErrorResponseDTO> handleRuntimeException(RuntimeException e) {
        // ... 처리
    }
}

@RestControllerAdvice(basePackages = "jiohh.springlogin.user")
public class LoginExceptionHandler {
    @ExceptionHandler(InvalidCredentialsException.class)
    public ResponseEntity<ErrorResponseDTO> handleInvalidCredentials(InvalidCredentialsException e) {
        // ... 처리
    }
}
```

### **예상했던 동작**

  

LoginController에서 InvalidCredentialsException이 발생하면,

LoginExceptionHandler가 먼저 잡아서 처리되길 기대했음.

  

### **실제 동작**

  

하지만 실제로는 GlobalExceptionHandler의 RuntimeException 핸들러가 먼저 실행되어,

세부 예외 처리 로직이 적용되지 않고 전역 메시지만 내려감.

---

## **원인 분석**

Spring은 여러 @RestControllerAdvice 클래스가 존재할 경우 내부적으로 정렬 기준에 따라 처리 순서를 결정한다.
기본적으로 @ExceptionHandler는 **가장 먼저 매칭되는 핸들러가 실행되고 나면 이후는 무시**된다.

따라서 RuntimeException을 잡는 핸들러가 먼저 적용되면,
그보다 구체적인 예외 (InvalidCredentialsException 등)는 호출되지 않는다.

---

## **해결 방안**

### **✅ 1.** @Order 어노테이션을 활용해 우선순위 명시

```
@RestControllerAdvice
@Order(Ordered.LOWEST_PRECEDENCE) // 전역 예외는 가장 마지막
public class GlobalExceptionHandler { ... }

@RestControllerAdvice(basePackages = "jiohh.springlogin.user")
@Order(Ordered.HIGHEST_PRECEDENCE) // 도메인 예외 먼저 처리
public class LoginExceptionHandler { ... }
```

- Ordered.HIGHEST_PRECEDENCE (0) : 가장 먼저 적용
    
- Ordered.LOWEST_PRECEDENCE (Integer.MAX_VALUE) : 가장 나중에 적용
    

  

### **❗️** @Order의 의미와 주의할 점*
  

처음에는 @Order가 @RestControllerAdvice에 사용되는 것이 명확하지 않을 수 있다.
  

원래 @Order는 Spring Bean의 정렬 순서를 지정하는 어노테이션이지만,
 @RestControllerAdvice 역시 Bean으로 등록되기 때문에 내부적으로 **ExceptionResolver**에서 우선순위 정렬에 사용된다.

  

즉, 공식 문서에 명시된 용도는 아니지만, **실제로는 Advice 간의 예외 핸들러 적용 우선순위를 조정할 수 있다.**
실무에서도 이 방식이 널리 쓰이고 있으며, Spring 내부에서도 AnnotationAwareOrderComparator를 통해 이를 지원한다.

  

### **✅ 2. 패키지 범위를 이용한 적용 범위 좁히기**

  

LoginExceptionHandler에 basePackages를 지정해 **적용 범위를 명확하게 설정**.

```
@RestControllerAdvice(basePackages = "jiohh.springlogin.user")
public class LoginExceptionHandler { ... }
```

이렇게 하면 LoginExceptionHandler는 로그인 관련 컨트롤러에서만 동작하며,

GlobalExceptionHandler는 그 외 컨트롤러에 대해 fallback 역할만 수행.

---

## **결론**

  

여러 개의 @RestControllerAdvice를 사용할 경우 다음 두 가지 전략을 병행해야 예외 처리 충돌을 방지할 수 있음:

1. **@Order로 우선순위 명시**: 도메인 별 핸들러를 먼저 처리하고, 전역 핸들러는 fallback 용도로
    
2. **basePackages 등으로 범위 제한**: 도메인 별 핸들러가 명확한 범위 내에서만 동작하도록
    

  

이러한 구조는 실제 기업 프로젝트에서도 자주 쓰이며, 예외 처리의 **모듈화**, **의도한 흐름 유지**, **가독성 향상** 측면에서 매우 유용함.

---

## **@Order는 Bean 우선순위 설정에 사용하는 것 아닌가?**

  

@Order는 본래 Spring에서 **여러 Bean의 우선순위를 지정하는 데 사용하는 어노테이션**이다.

예를 들어, 여러 개의 필터, 인터셉터, AOP Advice 등이 존재할 경우 우선순위를 지정해 순서를 제어할 수 있다.

그렇다면 @RestControllerAdvice에 @Order를 붙이는 것은 과연 정식 사용 방법일까?

  

### **✔ 내부 동작 원리로 보면 정당한 사용이다**


다음과 같은 이유로 @Order는 예외 처리 핸들러에도 영향을 줄 수 있다:

- @ControllerAdvice 또는 @RestControllerAdvice는 내부적으로 **ExceptionHandlerExceptionResolver**에 등록된 Bean이다.
- Spring은 여러 Advice를 처리할 때 AnnotationAwareOrderComparator를 사용해 우선순위를 정렬한다.
- ExceptionHandlerExceptionResolver는 예외 처리 시 우선순위가 높은 Bean부터 예외를 위임한다.

즉, @Order가 붙은 @RestControllerAdvice는 실제로 **예외 처리 순서에 영향을 준다**.

이 부분은 추후 @Order을 조금 더 찾아본 후 새로운 글로 작성해보고자 한다.  