
# @EnableWebSecurity

@EnableWebSecurity는 Spring Security 설정에서 핵심적인 역할을 하는 애노테이션입니다. 이 애노테이션은 Spring Security의 **기본 보안 설정을 활성화**하도록 스프링에게 알려주는 역할


- @Configuration과 함께 사용되어, Spring Security 설정을 포함하는 구성 클래스임을 명시
    
- WebSecurityConfigurerAdapter 또는 SecurityFilterChain을 통해 보안 설정을 커스터마이징할 수 있도록 함
    
- Spring Security의 **필터 체인(SecurityFilterChain)** 등록을 트리거함

- @EnableWebSecurity가 있어야 SecurityFilterChain 설정이 적용됨


---

## **작동 방식**

  

### **1. 기본 동작**

```
@EnableWebSecurity
public class SecurityConfig {
  // 설정 정의
}
```

이렇게 하면 Spring Boot가 제공하는 **기본 보안 설정(모든 요청에 인증 요구)**이 활성화됩니다.

  

### **2. 사용자 정의 설정**

  

Spring Security 5.7 이후에는 SecurityFilterChain 방식으로 설정을 구성하는 것이 권장됩니다.

```
@Configuration
@EnableWebSecurity
public class SecurityConfig {

  @Bean
  public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
      .authorizeHttpRequests()
        .requestMatchers("/public/**").permitAll()
        .anyRequest().authenticated()
      .and()
      .formLogin();

    return http.build();
  }
}
```

---

## **내부적으로 하는 일**

- @EnableWebSecurity는 내부적으로 WebSecurityConfiguration을 import하여 **SecurityFilterChain을 구성하는 Bean**을 등록합니다.
    
- 또한 AuthenticationManager, UserDetailsService 등의 보안 관련 Bean들도 생성될 수 있는 기반을 마련합니다.
    

---

## **요약 정리**

|**항목**|**설명**|
|---|---|
|역할|Spring Security 설정을 활성화|
|위치|@Configuration 클래스에 선언|
|기반 클래스|이전에는 WebSecurityConfigurerAdapter (5.7 이상에서는 SecurityFilterChain 사용 권장)|
|주요 기능|필터 체인 등록, 사용자 인증/인가 설정의 진입점 역할 수행|
