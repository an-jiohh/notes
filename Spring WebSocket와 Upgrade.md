# Spring WebSocket와 Upgrade

---
## 요약정리

**Upgrade란?**

기존 HTTP 연결을 같은 TCP 연결 위에서 WebSocket 통신으로 전환하는 과정이다.

**왜 필요한가?**

기존 웹 환경과 호환되면서도 실시간 양방향 통신을 가능하게 하기 위해서다.

**TCP 관점에서는?**

TCP 연결은 그대로 두고, 그 위에서 사용하는 프로토콜만 HTTP에서 WebSocket으로 바꾼다.

**Spring에서는 어떻게 일어나는가?**

Spring이 핸드셰이크를 검사하고, 인터셉터와 핸드셰이크 핸들러를 거친 뒤, 실제 업그레이드는 컨테이너에 위임한다. 성공하면 `WebSocketSession`을 통해 메시지 송수신이 이루어진다.

**STOMP와의 관계는?**

Upgrade 과정은 동일하고, 이후의 메시지 처리 규칙만 달라진다.

---

## 1. WebSocket에서 `Upgrade`가 필요한 이유

웹 환경은 원래 HTTP를 기본으로 동작한다. 브라우저와 웹 서버, 프록시, 방화벽, 로드밸런서 같은 기존 웹 인프라는 모두 HTTP 요청/응답 구조를 전제로 설계되어 있다. 그래서 WebSocket도 처음부터 완전히 별도의 방식으로 시작하지 않고, 먼저 HTTP 요청으로 접속한 뒤 필요한 경우 WebSocket으로 전환하는 방식을 사용한다.

이 전환을 `Upgrade`라고 부른다.

핵심 이유는 다음과 같다.

- 기존 웹 인프라와 자연스럽게 호환되기 쉽다.
    
- 80, 443 같은 기존 웹 포트를 그대로 활용할 수 있다.
    
- 브라우저와 서버가 이미 가진 HTTP 기능(헤더, 쿠키, 세션, 인증 정보 등)을 핸드셰이크 단계에서 재사용할 수 있다.
    
- 서버가 요청을 검사한 뒤에만 WebSocket 연결을 허용할 수 있다.
    

즉, `Upgrade`는 새로운 TCP 연결을 다시 만드는 개념이 아니라, **기존 HTTP 연결을 같은 TCP 연결 위에서 WebSocket 통신으로 전환하는 과정**이다.

---

## 2. Upgrade 과정

WebSocket 연결은 처음부터 바로 시작되지 않는다. 먼저 클라이언트와 서버가 TCP 연결을 맺고, 그 위에서 HTTP 요청을 통해 WebSocket 전환을 요청한다. 서버가 이를 승인하면 같은 TCP 연결을 유지한 채 WebSocket 통신으로 넘어간다.

흐름은 다음과 같다.

1. 클라이언트와 서버가 TCP 연결을 맺는다.
    
2. 클라이언트는 HTTP 요청을 보낸다.
    
3. 이 요청 안에 `Upgrade: websocket` 같은 헤더를 포함해 WebSocket 전환을 요청한다.
    
4. 서버가 요청을 검사한다.
    
5. 서버가 이를 승인하면 `101 Switching Protocols`로 응답한다.
    
6. 그 뒤부터는 같은 TCP 연결 위에서 WebSocket 프레임 규칙으로 통신한다.
    

즉, Upgrade는 새로운 연결을 다시 만드는 과정이 아니라, **이미 맺어진 TCP 연결 위에서 HTTP 통신을 WebSocket 통신으로 전환하는 절차**라고 볼 수 있다.

---

## 3. Spring에서 Upgrade는 어떻게 일어나는가

Spring에서 WebSocket 통신도 시작은 HTTP 요청이다. 브라우저가 `ws://...` 또는 `wss://...`로 접속하면 내부적으로는 먼저 HTTP 핸드셰이크 요청이 들어온다.

Spring은 이 요청을 받고 다음과 같은 순서로 처리한다.

1. 클라이언트가 HTTP 기반 WebSocket 핸드셰이크 요청을 보낸다.
    
2. Spring이 이를 일반 HTTP 요청처럼 먼저 받는다.
    
3. 핸드셰이크 관련 인터셉터와 핸들러가 요청을 검사한다.
    
4. 문제가 없으면 실제 WebSocket 업그레이드를 수행한다.
    
5. 업그레이드가 성공하면 이후부터는 `WebSocketSession`을 통해 메시지를 주고받는다.
    

즉, Spring에서 Upgrade는 **HTTP 세계에서 WebSocket 세계로 넘어가는 경계**라고 볼 수 있다.

---

## 5. Spring 내부 흐름

Spring에서 WebSocket 경로를 등록하면 해당 경로는 일반 컨트롤러 응답용이 아니라 WebSocket 핸드셰이크 대상이 된다.

내부 개념 흐름은 대략 아래와 같다.

```text
브라우저
  ↓
HTTP GET + Upgrade 요청
  ↓
Spring 웹 인프라
  ↓
HandshakeInterceptor
  ↓
HandshakeHandler
  ↓
RequestUpgradeStrategy
  ↓
서블릿 컨테이너(Tomcat 등)의 실제 업그레이드 수행
  ↓
WebSocketSession 생성
  ↓
WebSocketHandler 동작 시작
```

즉, Spring이 모든 네트워크 레벨 처리를 직접 하는 것이 아니라,

- Spring은 핸드셰이크 검사와 연결 흐름을 관리하고
    
- 실제 저수준 업그레이드는 톰캣 같은 컨테이너가 수행한다.
    

---

## 5. Spring에서 중요한 구성 요소

아래 요소들은 Spring WebSocket 동작에서 중요한 구성 요소들이다. 이 중에는 개발자가 직접 구현하거나 설정을 통해 수정할 수 있는 부분도 있고, 내부 동작으로 이해만 하면 되는 부분도 있다.

### 5-1. `WebSocketHandler`

업그레이드가 끝난 뒤 실제 메시지를 처리하는 핸들러다. 연결이 성공하면 세션이 전달되고, 이후 텍스트 메시지나 바이너리 메시지를 주고받는다.

예를 들어 `TextWebSocketHandler`를 상속하면 텍스트 기반 채팅 같은 기능을 구현할 수 있다.

수정 가능 여부: 가능

개발자가 직접 구현하는 영역이다. 메시지 처리 방식, 연결 후 동작, 종료 시 동작 등을 원하는 방식으로 작성할 수 있다.

---

### 5-2. `HandshakeInterceptor`

업그레이드 전후에 동작하는 인터셉터다. 주로 다음과 같은 일을 한다.

- 로그인 사용자 정보 확인
    
- JWT나 세션 검사
    
- 헤더 확인
    
- 사용자 이름 같은 값을 attributes에 저장
    
- 허용되지 않은 요청 차단
    

즉, `beforeHandshake()`는 “이 요청을 WebSocket으로 업그레이드해도 되는가?”를 판단하는 단계다.

수정 가능 여부: 가능

개발자가 직접 인터셉터를 구현해서 인증, 권한 확인, 속성 저장, 요청 차단 등의 로직을 넣을 수 있다.

---

### 5-3. `HandshakeHandler`

핸드셰이크 전체를 총괄하는 역할이다. 기본적으로 요청이 올바른 WebSocket 업그레이드 요청인지 확인하고, 서브프로토콜이나 확장 협상 등을 처리한다.

즉, 핸드셰이크의 논리적 중심이라고 볼 수 있다.

수정 가능 여부: 가능

기본 구현을 그대로 사용할 수도 있고, 필요하면 커스텀 핸드셰이크 핸들러를 지정해서 동작을 조정할 수 있다. 다만 보통은 기본 구현을 사용하고, 특별한 요구사항이 있을 때만 수정한다.

---

### 5-4. `RequestUpgradeStrategy`

실제 업그레이드를 수행하는 전략 객체다. 이 부분은 서버 런타임에 따라 달라질 수 있다.

예를 들어 Spring은 톰캣, 제티 등 각 컨테이너에 맞는 방식으로 실제 업그레이드를 위임한다.

즉,

- Spring = 업그레이드 절차를 관리하는 총감독
    
- 컨테이너 = 실제 소켓 업그레이드를 수행하는 실행 담당
    

이라고 이해하면 쉽다.

수정 가능 여부: 일반적으로는 수정하지 않음

이 부분은 Spring과 서블릿 컨테이너가 내부적으로 처리하는 영역이다. 특별한 저수준 커스터마이징이 필요한 경우가 아니라면 개발자가 직접 건드리는 경우는 드물다.

---

## 6. Spring에서 Upgrade 전과 후의 차이

### Upgrade 전: 아직 HTTP 단계

이때는 일반 HTTP 요청처럼 다룰 수 있다.

가능한 것:

- 헤더 읽기
    
- 쿠키 읽기
    
- 세션 확인
    
- 인증 정보 확인
    
- URI 파라미터 확인
    
- Origin 검사
    

### Upgrade 후: 이제 WebSocket 단계

이제는 일반 HTTP 요청/응답 흐름이 아니다.

이후 사용하는 것:

- `WebSocketSession`
    
- `TextMessage`
    
- `BinaryMessage`
    
- `handleTextMessage()`
    
- `sendMessage()`
    

즉, Upgrade는 단순한 설정이 아니라 **프로그래밍 모델 자체가 바뀌는 시점**이다.

---

## 7. `afterConnectionEstablished()`의 의미

Spring에서 `afterConnectionEstablished()`가 호출된다는 것은 이미 업그레이드가 성공했다는 뜻이다.

즉,

- 아직 HTTP 핸드셰이크 단계라면 이 메서드는 호출되지 않는다.
    
- 업그레이드가 완료되고 `WebSocketSession`이 만들어진 뒤에만 호출된다.
    

따라서 이 메서드는 “이제부터 WebSocket 통신을 시작할 수 있다”는 신호로 이해하면 된다.

---

## 8. STOMP를 써도 Upgrade는 같은가

그렇다. STOMP를 사용하든 순수 WebSocket을 사용하든 시작은 동일하다.

먼저 HTTP 요청으로 핸드셰이크를 수행하고, 업그레이드가 성공하면 WebSocket 연결이 만들어진다. 차이는 그 다음부터다.

### 순수 WebSocket

- 메시지를 직접 해석해야 한다.
    
- `WebSocketHandler`에서 직접 처리한다.
    

### STOMP over WebSocket

- WebSocket 위에 STOMP라는 메시지 규칙을 얹는다.
    
- 발행/구독 모델을 쉽게 사용할 수 있다.
    
- `@MessageMapping`, 브로커, 구독 관리 등을 활용할 수 있다.
    

즉, Upgrade 자체는 같고, **업그레이드 이후의 메시지 처리 방식만 달라진다**.