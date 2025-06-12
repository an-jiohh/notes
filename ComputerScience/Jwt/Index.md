아직 부족한 점이 많습니다...
내용 추가로 학습하는데로 추가하겠습니다...
### **Jwt 주요 항목 목록**

#### 1. JWT 기본 개념
- [[JWT란 무엇인가?]]
- [[JWT 구조]]
- [[JWT가 필요한 이유]]
- [[JWT vs 세션 방식 인증]]

#### 2. [[JWT 구조]]
- JWT의 구성 요소: Header, Payload, Signature
- 각 요소의 역할과 예시
- [[JWT의 인코딩 방식 (Base64Url)]]

#### 3. JWT 생성 및 서명
- [[토큰 발급 시 필요한 정보]]
- [[대칭키(HMAC) vs 비대칭키(RSA, ECDSA) 방식]]
- [[JWT 서명 방식 (alg 종류)]]


#### 4. JWT의 사용 흐름

- [[JWT 전체 흐름]]
- [[클라이언트가 JWT를 저장하는 위치 (LocalStorage vs Cookie)]]
- [[Authorization 헤더 사용 방식 (Bearer token)]]


#### 5. JWT의 유효성 검증
- [[유효성 검증이 필요한 이유]]
- [[토큰 유효 기간 확인 (exp)]]
- [[서명 검증]]
- [[토큰 무효화 방법의 한계 (ex 로그아웃 처리)]]
    
#### 6. Access Token vs Refresh Token
- [[Access Token, Refresh Token이 왜 필요한가]]
- [[Token의 보관 방법]]
- [[재발급 흐름]]
- [[HttpOnly + cookie 사용시 재발급 흐름]]
- [[왜 공식문서에는 Authorization헤더로 전송하는 것을 기반으로 되어 있는가?]]
- [[Refresh Token 구성 방식]]

#### 7. 보안 고려사항
- [[토큰 탈취 방지 - XSS]]
- [[토큰 탈취 방지 - CSRF]]
- JWT 키 로테이션 전략

#### 8. OAuth
- [[OAuth의 등장 배경과 탄생 이유]]
- [[OAuth 기반 로그인 - OpenID Connect(OIDC)]]

#### 9. 기타
- JWT의 단점과 대안 (PASETO, OAuth2 등)
- 토큰 블랙리스트 처리 방식
- 중복 로그인 문제 (여러 기기에서의 로그인)
- 기기에 따른 로그인 관리 문제
- 토큰 탈취 문제
