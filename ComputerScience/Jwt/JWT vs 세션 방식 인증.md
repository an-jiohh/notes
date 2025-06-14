
## JWT vs 세션 방식 인증

|**구분**|**세션 방식 인증 (Session-based Auth)**|**JWT 인증 (Token-based Auth)**|
|---|---|---|
|**인증 정보 저장 위치**|서버 (메모리/DB/Redis 등)|클라이언트 (토큰 자체에 정보 포함)|
|**서버 상태 (State)**|**Stateful** (상태 저장)|**Stateless** (상태 없음)|
|**스케일링(서버 확장)**|세션 공유 필요 (Sticky Session 또는 중앙 저장소 필요)|별도 공유 필요 없음 (토큰만 전달하면 됨)|
|**요청 처리 흐름**|세션 ID → 서버 조회 → 사용자 정보|토큰 → 토큰 검증 → 사용자 정보 추출|
|**저장 및 전송 방법**|세션 ID를 쿠키에 저장 → 자동 전송|JWT를 Authorization: Bearer 헤더로 전송|
|**보안**|세션 탈취 (Session Hijacking)에 취약|토큰 탈취 시 주의 필요, 주로 HTTPS + 짧은 만료|
|**서버 부하**|사용자 수 많아질수록 세션 저장 공간 증가|토큰 자체가 인증정보를 가지므로 서버 부하 ↓|
|**만료 처리**|세션 만료 시 서버에서 삭제|JWT는 유효기간(exp) 지나면 자동 만료 (로그아웃 직접 반영 어려움)|
|**로그아웃 처리**|세션 제거로 바로 처리 가능|JWT는 만료되기 전까지 유효 → 로그아웃 처리가 복잡함 (Blacklist 등 필요)|

## 어떤 경우에 JWT가 더 적합한가?

- REST API 기반 백엔드    
- 서버 간 인증 공유가 필요한 마이크로서비스 구조
- 모바일 앱, SPA(React/Vue) 등 클라이언트 중심 아키텍처
- 무상태 인증이 필요한 경우 (Stateless)

  

## 어떤 경우에 세션이 더 적합한가?

- 전통적인 웹 사이트 (HTML 서버 렌더링 기반)
- 인증 처리에 복잡한 보안 정책이 필요한 경우
- 토큰 저장소를 따로 두기 어려운 상황

---

### **🧠 요약 문장**

  
**세션은 서버에 저장하고 관리하는 인증 방식, JWT는 클라이언트가 토큰을 들고 다니는 자가 포함(Self-contained) 인증 방식이다.**

각각의 특성과 보안 고려사항에 따라 **시스템 구조와 목적에 맞게 선택**해야 한다.

---
