
# Refresh Token 저장시 Repository 구성
## **전체 인터페이스 요약**

```
public interface RefreshTokenRepository {
    void save(RefreshToken refreshToken);
    Optional<RefreshToken> findByUserId(String userId);
    Optional<RefreshToken> findByToken(String token);
    void deleteByUserId(String userId);
    void deleteByToken(String token);
    int deleteAllExpiredBefore(Instant now);
}
```

---

## **1.**  **void save(RefreshToken refreshToken)**

  

### **목적**

- **신규 발급 시 저장** 또는 **기존 토큰 갱신(update)**
    

  

### **언제 쓰나?**

- 로그인 시 refreshToken을 새로 생성했을 때
    
- 기존 userId에 대한 토큰이 이미 있다면 덮어쓰기 (merge)
    
- 없으면 새로 삽입 (persist)
    

  

### **예시**

```
RefreshToken token = new RefreshToken(...);
refreshTokenRepository.save(token);
```

---

## **2.**  **Optional<RefreshToken> findByUserId(String userId)**

  

### **목적**

- 특정 사용자에 대한 refreshToken 조회
    
- userId가 엔티티의 @Id인 경우 **기본 키 조회**
    

  

### **언제 쓰나?**

- 로그인 시 기존 토큰이 있는지 확인하고 갱신할지 결정할 때
    
- 사용자별로 토큰 유효 여부 검증할 때
    

  

### **예시**

```
Optional<RefreshToken> tokenOpt = refreshTokenRepository.findByUserId("user123");
```

---

## **3.**  **Optional<RefreshToken> findByToken(String token)**

  

### **목적**

- 클라이언트가 보낸 토큰이 유효한지 DB에서 직접 찾아서 검증
    

  

### **언제 쓰나?**

- refresh 요청 시, 클라이언트가 보낸 토큰 문자열로 찾기
    
- 토큰 탈취 여부 확인용
    
- 서버-DB 관점의 블랙리스트 체크 등
    

  

### **예시**

```
Optional<RefreshToken> tokenOpt = refreshTokenRepository.findByToken(requestToken);
```

---

## **4.**  **void deleteByUserId(String userId)**

  

### **목적**

- **로그아웃 시**, 해당 사용자의 토큰을 제거
    

  

### **언제 쓰나?**

- 유저 로그아웃, 강제 로그아웃, 계정 삭제 등
    
- 하나의 사용자당 하나의 refreshToken만 관리한다면 필수
    

  

### **예시**

```
refreshTokenRepository.deleteByUserId("user123");
```

---

## **5.**  **void deleteByToken(String token)**

  

### **목적**

- 토큰 문자열로 직접 삭제
    
- 토큰 블랙리스트 처리나 위조된 토큰 삭제용
    

  

### **언제 쓰나?**

- refreshToken이 특정한 이유로 폐기돼야 할 때 (예: 유출 의심)
    
- 토큰 값만 있을 경우
    

  

### **예시**

```
refreshTokenRepository.deleteByToken(requestToken);
```

---

## **6.**  **int deleteAllExpiredBefore(Instant now)**

  

### **목적**

- **스케줄러에서 정기적으로 만료된 토큰 정리**
    

  

### **언제 쓰나?**

- 하루에 한 번 만료된 토큰들 DB에서 제거
    
- 오래된 토큰 쌓이는 것 방지 (성능, 보안 차원)
    

  

### **예시**

```
int deleted = refreshTokenRepository.deleteAllExpiredBefore(Instant.now());
System.out.println("Expired tokens removed: " + deleted);
```

---

## **요약 표**

|**메서드 이름**|**목적**|**사용 시점**|
|---|---|---|
|save()|토큰 생성/갱신|로그인, 재발급|
|findByUserId()|유저 기준 조회|로그인, 검증|
|findByToken()|토큰 기준 조회|재발급 요청 시 검증|
|deleteByUserId()|유저 토큰 삭제|로그아웃|
|deleteByToken()|특정 토큰만 삭제|보안 위협 차단, 블랙리스트|
|deleteAllExpiredBefore()|만료된 토큰 전체 삭제|배치/스케줄러|
