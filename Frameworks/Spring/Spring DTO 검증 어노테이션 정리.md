# ✅ Spring DTO 검증 어노테이션 정리

Spring에서 `@Valid` 또는 `@Validated`와 함께 사용할 수 있는 **검증 어노테이션**은 JSR-380 (Jakarta Bean Validation 3.0)을 기반으로 하며, 입력값의 유효성 검사를 편리하게 도와줍니다.

---

## 📌 기본 검증 어노테이션 목록

| 어노테이션 | 설명 |
|------------|------|
| `@NotNull` | `null` 값 허용하지 않음 (공백은 허용됨) |
| `@NotEmpty` | 문자열, 배열, 컬렉션이 `null`이거나 비어 있으면 안됨 |
| `@NotBlank` | 문자열이 `null`, 빈 문자열, 공백 문자만 포함할 경우 불허 |
| `@Size(min, max)` | 문자열, 배열, 컬렉션의 길이나 크기 제한 |
| `@Min(value)` | 최소값 제한 (`int`, `long` 등 숫자형에 사용) |
| `@Max(value)` | 최대값 제한 |
| `@Positive` | 양수만 허용 (`> 0`) |
| `@PositiveOrZero` | 0 이상만 허용 (`>= 0`) |
| `@Negative` | 음수만 허용 (`< 0`) |
| `@NegativeOrZero` | 0 이하만 허용 (`<= 0`) |
| `@Email` | 이메일 형식 검사 (`abc@example.com`) |
| `@Pattern(regexp = "...")` | 정규 표현식을 통한 형식 검사 |
| `@Past` | 과거 날짜만 허용 |
| `@PastOrPresent` | 과거 또는 현재 날짜 허용 |
| `@Future` | 미래 날짜만 허용 |
| `@FutureOrPresent` | 현재 또는 미래 날짜만 허용 |
| `@AssertTrue` | `boolean` 필드가 `true`여야 함 |
| `@AssertFalse` | `boolean` 필드가 `false`여야 함 |
| `@Digits(integer, fraction)` | 정수/소수 자릿수 제한 |
| `@DecimalMin(value)` | 최소값 제한 (실수 포함) |
| `@DecimalMax(value)` | 최대값 제한 (실수 포함) |

---

## 🧩 타입별 추천 어노테이션 조합

| 타입 | 추천 어노테이션 |
|------|----------------|
| `String` | `@NotBlank`, `@Size`, `@Email`, `@Pattern` |
| `int`, `Integer`, `Long` 등 숫자 | `@NotNull`, `@Min`, `@Max`, `@Positive` |
| `List`, `Set`, `Map` | `@NotEmpty`, `@Size` |
| `LocalDate`, `Date` | `@Past`, `@Future`, `@PastOrPresent` |
| `boolean` | `@AssertTrue`, `@AssertFalse` |

---

## 🧱 중첩 객체 및 리스트의 검증 처리

```java
public class ParentDto {
    
    @Valid
    private ChildDto child;

    @Valid
    private List<@Valid ChildDto> children;
}
````

- 중첩 객체나 리스트의 각 요소에도 재귀적으로 @Valid를 붙여줘야 검증됨
    

---

## **🎯 실용 예시: DTO 클래스**

```
public class SignUpRequestDto {

    @NotBlank(message = "아이디는 필수입니다.")
    private String userId;

    @Size(min = 8, message = "비밀번호는 8자 이상이어야 합니다.")
    private String password;

    @Email(message = "이메일 형식이 아닙니다.")
    private String email;

    @Min(value = 14, message = "14세 이상만 가입할 수 있습니다.")
    private int age;
}
```

---

## **🔧 의존성 설정 (Spring Boot)**

```
<!-- Spring Boot 3 이상에서는 자동 포함됨 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

---

## **⚙️ 검증 예외 처리 예시 (@RestControllerAdvice)**

```
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponseDto<Object>> handleValidationException(MethodArgumentNotValidException ex) {
        String errorMsg = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .findFirst()
            .orElse("잘못된 요청입니다.");

        return ResponseEntity.badRequest().body(ApiResponseDto.error("VALIDATION_ERROR", errorMsg));
    }
}
```

---

## **🧩 커스텀 검증 어노테이션도 만들 수 있다**

- 예: @ValidPassword, @UniqueEmail, @MatchPassword 등
- ConstraintValidator<A, T>를 구현하여 커스터마이징 가능

---

## **📚 참고**

- [Jakarta Bean Validation](https://jakarta.ee/specifications/bean-validation/)
- [Spring Boot Validation 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.validation)
