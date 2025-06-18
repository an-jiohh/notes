  

Java의 어노테이션은 메타데이터(metadata)를 코드에 부여하는 방식이다.

이 중 “커스텀 어노테이션”은 개발자가 직접 정의하여, 특정 로직이나 의미를 코드에 주석처럼 표시하고,

이를 프레임워크나 런타임에서 해석하여 동작을 유도할 수 있도록 한다.

자주 반복되는 로직이나, 의미를 부여해야 할 메타데이터를 명확하게 표현하고, 이를 처리하기 위한 전처리기(예: AOP, 리플렉션 기반 로직 등)와 함께 사용한다.

---

## **어노테이션 처리 흐름**

  

어노테이션 자체는 아무 기능이 없으며, 반드시 이를 **해석하는 처리 로직**이 있어야 한다.

대표적인 처리 주체는 다음과 같다:

|**처리 주체**|**설명**|**동작 시점**|**예시 어노테이션**|
|---|---|---|---|
|컴파일러|컴파일 시점에 어노테이션을 기반으로 오류 확인, 코드 생성|컴파일 시|@Override, @Generated|
|프레임워크|스프링, JPA 등에서 어노테이션을 분석하고 Bean 등록, 매핑 등을 처리|초기 실행 시|@Component, @Entity, @Service|
|런타임 해석기|리플렉션이나 프록시, AOP 등을 활용하여 동작 시점에 처리|요청/실행 시|@Transactional, @LoginUser, @Valid|

---

## **커스텀 어노테이션 정의 예시**

```
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```

### **구성 요소**

|**요소**|**설명**|
|---|---|
|@Target|어노테이션 적용 위치 지정 (TYPE, METHOD, FIELD, PARAMETER 등)|
|@Retention|어노테이션 유지 기간 설정 (SOURCE, CLASS, RUNTIME)|
|@Documented|Javadoc 문서에 포함 여부 설정|
|@Inherited|상속 여부 설정 (클래스에만 적용 가능)|

---
## **어떻게 사용해서 동작을 바꾸는지**

#### **1. 컴파일러 기반 처리**

- **컴파일 시점**에 어노테이션을 인식해서 경고, 오류, 코드 생성 등에 활용
    
- 주로 @Override, @Deprecated, @SuppressWarnings 같은 어노테이션이 여기에 해당
    
- 또는 **Annotation Processor**를 활용한 **소스 코드 생성** (예: @Getter → Lombok)
    
  
**예시:**  **@Override**

```
@Override
public void run() {
    // 부모 클래스에 같은 메서드가 없다면 컴파일 에러 발생
}
```

→ 이 어노테이션이 없으면 오타가 있어도 에러가 안 날 수 있음

**예시: Annotation Processor (APT)**

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
public @interface AutoFactory {
}
```

→ 컴파일 시 AutoFactoryProcessor가 동작해 자동으로 .java 파일을 생성

---

#### **2. 프레임워크 기반 처리 (Spring, JPA 등)**

- 프레임워크가 어노테이션을 **리플렉션**이나 **스프링 컨텍스트 로딩 과정**에서 해석
    
- 주로 DI, 트랜잭션, 유효성 검증, REST 매핑 등에서 사용


**예시 1: Spring의**  **@Component** **,** **@Service**

```
@Component
public class MyBean {}
```

→ Spring 부트가 클래스패스를 스캔할 때, 해당 클래스를 Bean으로 자동 등록

**예시 2: JPA의** **@Entity**

```
@Entity
public class User {
    @Id
    private Long id;
}
```

→ JPA가 해당 클래스를 데이터베이스 테이블과 매핑해서 SQL을 자동 생성

---

#### **3. 런타임 기반 처리 (리플렉션, AOP 등)**

### **특징**

- 실행 중에 어노테이션 정보를 읽고 동작을 바꿈
    
- 주로 **리플렉션 API**나 **프록시 기반 AOP**에서 사용
    
- @AspectJ, @LoginUser, @Transactional, @Valid 등이 여기에 속함
    

  
**예시 1: 커스텀 유효성 검증** **@ValidName**

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidName {
}
```

```
// 리플렉션을 통해 해당 필드에 어노테이션이 붙었는지 확인
Field field = ...;
if (field.isAnnotationPresent(ValidName.class)) {
    // 검증 로직 수행
}
```

**예시 2: Spring AOP** **@LogExecutionTime**

```
@Around("@annotation(LogExecutionTime)")
public Object log(ProceedingJoinPoint joinPoint) {
    long start = System.currentTimeMillis();
    Object result = joinPoint.proceed();
    long end = System.currentTimeMillis();
    System.out.println("실행 시간: " + (end - start) + "ms");
    return result;
}
```

---
## **커스텀 어노테이션 해석 방식**

  

### **1. HandlerMethodArgumentResolver**

- 컨트롤러의 파라미터에 붙은 어노테이션을 해석해 객체를 주입할 때 사용
    
- 대표 예: @RequestParam, @PathVariable, @LoginUser
    

```
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(LoginUser.class);
    }

    public Object resolveArgument(...) {
        return session.getAttribute("loginUser");
    }
}
```

```
// WebMvcConfigurer에 등록
@Override
public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
    resolvers.add(new LoginUserArgumentResolver());
}
```

---

### **2. AOP (Aspect-Oriented Programming)**

- 어노테이션이 붙은 메서드에 공통 동작을 주입할 때 사용
    
- 대표 예: @Transactional, @LogExecutionTime
    

```
@Around("@annotation(LogExecutionTime)")
public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = joinPoint.proceed();
    long end = System.currentTimeMillis();
    log.info("실행 시간: " + (end - start) + "ms");
    return result;
}
```

---

### **3. 인터셉터 (Interceptor)**

- HTTP 요청/응답 전후로 어노테이션을 활용하여 인증, 권한 검증 등을 처리
    
- @LoginRequired, @AdminOnly 같은 인증 관련 어노테이션 해석에 사용 가능
    

```
public class JwtAuthInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String token = request.getHeader("Authorization");
        if (!isValid(token)) {
            response.setStatus(401);
            return false;
        }
        return true;
    }
}
```
---
## 구현
#### 목표

```
@GetMapping("/me")
public UserDto myInfo(@LoginUser User user) {
    return new UserDto(user.getUsername());
}
```
아래와 같이 컨트롤러에서 로그인 유저 정보를 별도 조회 없이 자동 주입받고 싶다.


#### **전체 구조**
```
요청 → DispatcherServlet
       → HandlerMapping에서 컨트롤러 메서드 찾음
       → ArgumentResolver로 파라미터 해석
       → @LoginUser 붙은 파라미터 감지
       → 세션 또는 토큰에서 유저 꺼냄
       → 컨트롤러 메서드 호출 시 주입
```

#### **Step 1. 어노테이션 정의**
```
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```
- @Target(PARAMETER) : 메서드 파라미터에만 사용 가능
- @Retention(RUNTIME) : 런타임에 해석 가능해야 리플렉션으로 동작 가능

#### **Step 2. ArgumentResolver 구현**

HandlerMethodArgumentResolver를 구현해서 해당 어노테이션이 붙은 파라미터를 처리

```
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(LoginUser.class) &&
               parameter.getParameterType().equals(User.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) {
        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();

        // 로그인 유저를 세션에서 꺼낸다고 가정
        return request.getSession().getAttribute("loginUser");
    }
}
```
- supportsParameter() : 해당 파라미터를 처리할지 여부    
- resolveArgument() : 실제 객체 반환

#### **Step 3. WebMvcConfigurer에 등록**

이 ArgumentResolver는 수동으로 등록해줘야 한다.
```
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final LoginUserArgumentResolver loginUserArgumentResolver;

    public WebConfig(LoginUserArgumentResolver loginUserArgumentResolver) {
        this.loginUserArgumentResolver = loginUserArgumentResolver;
    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(loginUserArgumentResolver);
    }
}
```

---

## **커스텀 어노테이션 실전 예시**

|**어노테이션 이름**|**목적**|**처리 방식**|
|---|---|---|
|@LoginUser|로그인 유저 객체 주입|HandlerMethodArgumentResolver|
|@LogExecutionTime|메서드 실행 시간 로깅|AOP (@Around)|
|@EncryptField|민감 필드 암호화 처리|AOP 또는 Jackson 커스터마이저|
|@ValidName|입력 필드 유효성 검사|Bean Validator 커스터마이저|

---

## **마무리 요약**

- 커스텀 어노테이션은 **의미를 부여하는 표식**일 뿐, 반드시 해석 로직이 필요하다
    
- 해석 로직은 보통 스프링의 **리졸버, 프록시, AOP, 인터셉터**로 구성된다
    
- 실무에서는 인증, 로깅, 암호화, 트랜잭션, 캐싱, 검증 등에 널리 활용된다