
로그인 시 **기존 refreshToken을 어떻게 처리할 것인가**에 대해 정리한 문서


---

### **전략 1:** **기존 refreshToken 삭제 후 새로 저장 (1인 1세션 원칙)**

- 로그인할 때, **기존에 발급된 refreshToken이 있으면 삭제**
    
- 새로 만든 refreshToken만 저장
    

  

#### **예시 코드 (JPA 기반):**

```
public void reissueRefreshToken(String userId, String newRefreshToken) {
    refreshTokenRepository.deleteByUserId(userId); // 기존 삭제
    RefreshToken token = new RefreshToken(userId, newRefreshToken, issuedAt, expireAt);
    refreshTokenRepository.save(token); // 새로 저장
}
```

#### **장점:**

- **보안에 강함** (이전 refreshToken으로 접근 불가)
    
- 토큰 탈취 가능성 감소
    
- 구현 간단
    

  

#### **단점:**

- 여러 디바이스에서 로그인 불가 (ex. PC, 모바일 동시 로그인 불가)
    

---

### **전략 2:**  **기존 refreshToken 유지 + 병렬 저장 (멀티 디바이스)**

- 같은 userId에 대해 **여러 refreshToken을 저장**
    
- 디바이스, 브라우저, IP 등으로 구분
    

  

#### **테이블 구조 예시:**

|**userId**|**refreshToken**|**deviceInfo**|**createdAt**|
|---|---|---|---|
|user123|abc.def.ghi|Chrome_PC|2024-06-19|
|user123|jkl.mno.pqr|Safari_iOS|2024-06-19|

#### **예시 코드:**

```
public void storeNewToken(String userId, String newRefreshToken, String deviceInfo) {
    RefreshToken token = new RefreshToken(userId, newRefreshToken, deviceInfo, now, expireAt);
    refreshTokenRepository.save(token); // 덮지 않음, 새로 추가
}
```

#### **장점:**

- **멀티 디바이스 지원**
    
- 유연한 사용자 경험
    

  

#### **단점:**

- **보안 관리 복잡** (탈취된 token 삭제/블랙리스트 필요)
    
- 로그아웃 처리 시 **어떤 디바이스 토큰인지 식별 필요**
    

---

### **전략 3:** **기존 refreshToken 덮어쓰기 (업데이트)**

- 기존 토큰은 유지하되, **내용을 새 토큰 값으로 덮어씀**
    
- 주로 같은 디바이스에서 재로그인 시 사용
    

  

#### **예시 코드:**

```
public void updateRefreshToken(String userId, String newToken) {
    Optional<RefreshToken> existing = refreshTokenRepository.findByUserId(userId);
    if (existing.isPresent()) {
        RefreshToken token = existing.get();
        token.updateValue(newToken, newExpireTime);
    } else {
        refreshTokenRepository.save(new RefreshToken(userId, newToken));
    }
}
```

#### **장점:**

- 멀티 디바이스가 아니고, **하나의 디바이스 안에서는 부드럽게 사용**
    
- 기존 토큰 ID 재활용 가능
    

  

#### **단점:**

- 병렬 로그인은 제한됨
    
- 과도한 재로그인 시 **예상치 못한 충돌 가능성**
    

---

### **✅ 전략 4:** **Redis 등 캐시 저장소 사용 시 TTL 자동 만료 처리**

- userId를 키로 하고 refreshToken을 값으로 저장
    
- 중복 로그인 시 **덮어쓰기(OSET)** 또는 **deviceId로 분리된 key** 사용
    

```
redisTemplate.opsForValue().set("refresh:user123", token, Duration.ofDays(7));
```

- Redis는 TTL 지원 → 시간 지나면 자동 삭제
    

---

## **🔐 어떤 전략을 써야 하나요?**

|**요구사항**|**추천 전략**|
|---|---|
|보안 최우선, 단일 세션|삭제 후 재발급 (전략 1)|
|멀티 디바이스 허용|병렬 저장 (전략 2)|
|단일 디바이스, UX 중요|덮어쓰기 (전략 3)|
|캐시 기반 처리 필요|Redis TTL 활용 (전략 4)|

---

## **📌 참고: 전략 선택 시 고려 포인트**

- 사용자가 **동시에 여러 장치에서 로그인할 수 있어야 하는가?**
    
- 탈취된 refreshToken의 **실시간 차단**이 필요한가?
    
- 토큰 DB가 늘어나는 것을 **어디까지 허용할 것인가?**
    
- 로그아웃 시 **특정 토큰만 삭제해야 하는가?**
    
