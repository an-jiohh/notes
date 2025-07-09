# java.security.Key

java.security.Key는 Java에서 암호화에 사용되는 **모든 종류의 키**(공개키, 개인키, 대칭키 등)의 **최상위 인터페이스**입니다. 이 인터페이스는 PublicKey, PrivateKey, SecretKey 등 다양한 하위 인터페이스들의 공통 기반 역할을 합니다.

---

## **1.**  **Key 인터페이스 개요**

```
package java.security;

public interface Key extends Serializable {
    String getAlgorithm();     // 사용하는 알고리즘 (예: "RSA", "AES")
    String getFormat();        // 인코딩 포맷 (예: "X.509", "PKCS#8", "RAW")
    byte[] getEncoded();       // 키 데이터를 바이트 배열로 반환
}
```

### **주요 특징**

- Serializable을 상속함: 키 객체를 직렬화 가능
    
- 실제 구현체는 보통 SecretKeySpec, PrivateKeyImpl, PublicKeyImpl 등 라이브러리나 JDK 내부에서 제공됨
    

---

## **2. 주요 하위 인터페이스**

|**인터페이스**|**설명**|**예시 알고리즘**|
|---|---|---|
|PublicKey|공개키 (비대칭키 중 하나)|RSA, DSA, EC|
|PrivateKey|개인키 (비대칭키 중 하나)|RSA, DSA, EC|
|SecretKey|대칭키|AES, HMAC, DES|

---

## **3. 주요 사용 예시**

  

### **(1) 공개/개인 키 쌍 생성 (RSA)**

```
KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
keyGen.initialize(2048);
KeyPair keyPair = keyGen.generateKeyPair();

PublicKey publicKey = keyPair.getPublic();
PrivateKey privateKey = keyPair.getPrivate();
```

### **(2) 대칭키 생성 (AES)**

```
KeyGenerator keyGen = KeyGenerator.getInstance("AES");
keyGen.init(128); // 또는 256
SecretKey secretKey = keyGen.generateKey();
```

---

## **4. JWT에 적용 예시 (JJWT 등)**


JJWT에서 signWith()에 Key 타입을 인자로 받을 수 있음(기존의 방법은 Deprecated)

```
SecretKey key = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
String jwt = Jwts.builder()
    .setSubject("user")
    .signWith(key, SignatureAlgorithm.HS256)
    .compact();
```

JJWT에서 Keys.hmacShaKeyFor 메소드를 사용하면 쉽게 변환가능하다.
