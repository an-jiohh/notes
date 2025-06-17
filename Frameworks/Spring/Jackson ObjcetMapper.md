# **Jackson ObjectMapper 정리**
  

  

Jackson은 자바에서 JSON 데이터를 직렬화 및 역직렬화하기 위한 대표적인 라이브러리로, Spring Boot에서 기본 JSON 처리 도구로 채택되어 있다. REST API 기반의 백엔드 개발에서는 클라이언트와의 데이터 교환을 JSON 형식으로 처리하는 것이 일반적이며, 이를 효율적으로 처리하기 위해 ObjectMapper가 사용된다.

---

## **ObjectMapper의 기능 개요**

  

ObjectMapper는 다음과 같은 기능을 제공한다.

- 자바 객체 → JSON 문자열 변환 (Serialization)
    
- JSON 문자열 → 자바 객체 변환 (Deserialization)
    
- JSON ↔ Map/List 변환
    
- DTO ↔ Entity 변환
    
- JSON 구조에 따른 필드 명명 전략, null 처리 방식 등 유연한 설정 가능
    

---

## **주요 메소드와 예외 정리**

  

### **1.**  **writeValueAsString(Object value)**

  

자바 객체를 JSON 문자열로 직렬화한다.

```
User user = new User("홍길동", 30);
String json = objectMapper.writeValueAsString(user);
```

#### **발생 가능 예외**

- JsonProcessingException: 직렬화 중 문제가 발생할 경우
    
- InvalidDefinitionException: 직렬화 가능한 구조가 아닐 경우 (예: 순환 참조 등)
    

---

### **2.** 

### **readValue(String content, Class<T> valueType)**

  

JSON 문자열을 자바 객체로 역직렬화한다.

```
String json = "{\"name\":\"홍길동\", \"age\":30}";
User user = objectMapper.readValue(json, User.class);
```

#### **발생 가능 예외**

- JsonProcessingException: 전체적인 역직렬화 오류의 상위 예외
    
- JsonMappingException: JSON 필드 → 객체 필드 매핑 중 오류
    
- MismatchedInputException: 타입 불일치 등으로 매핑이 불가능한 경우
    
- IOException: 파일, 스트림 등에서 읽을 때 I/O 문제
    

---

### **3.**  **readValue(String content, TypeReference<T> valueTypeRef)**

  

제네릭 타입으로 변환할 때 사용하며, List, Map 등 구조에 적합하다.

```
String json = "{\"name\":\"홍길동\", \"age\":30}";
Map<String, Object> map = objectMapper.readValue(json, new TypeReference<>() {});
```

#### **발생 가능 예외**

- 위 readValue와 동일한 예외가 발생함
    

---

### **4.**  **convertValue(Object fromValue, Class<T> toValueType)**

  

자바 객체를 다른 클래스 구조로 변환할 때 유용하며 DTO ↔ Entity 변환 등에 사용된다.

```
UserDto dto = objectMapper.convertValue(userEntity, UserDto.class);
```

#### **발생 가능 예외**

- IllegalArgumentException: 구조적으로 변환할 수 없는 경우
    
- MismatchedInputException: 내부적으로 Jackson이 매핑 실패 시
    

---

### **5.**  **setSerializationInclusion(JsonInclude.Include.NON_NULL)**

  

JSON 직렬화 시 null 값 필드를 제외할 수 있다.

```
objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
```

```
User user = new User("홍길동", null);
String json = objectMapper.writeValueAsString(user);
// 결과: {"name":"홍길동"} (null 필드 제외)
```

예외 발생 없음 (단, 필드가 직렬화 불가능한 타입이면 위의 예외와 동일하게 처리)

---

### **6.**  **setPropertyNamingStrategy(...)**

  

카멜 케이스 → 스네이크 케이스 등 JSON 필드명 전략을 지정할 수 있다.

```
objectMapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);
```

```
class User {
    private String userName;
}
// → {"user_name": "홍길동"}
```

예외 발생 없음 (내부 처리 오류 시만 예외 발생)

---

### **7.**  **disable(...) / enable(...)**

  

예외 처리를 완화하거나 기능을 명시적으로 설정할 수 있다.

```
objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

```
String json = "{\"name\":\"홍길동\",\"unknown\":\"xxx\"}";
User user = objectMapper.readValue(json, User.class);
// unknown 필드 무시됨
```

#### **관련 예외**

- JsonMappingException: 위 설정을 하지 않으면 존재하지 않는 필드 매핑 시 발생
    

---

## **Jackson 관련 주요 예외 요약**

|**예외 클래스**|**발생 조건**|**설명**|
|---|---|---|
|JsonProcessingException|직렬화 또는 역직렬화 전반적인 오류|최상위 예외|
|JsonMappingException|필드 매핑 중 오류 발생|필드 타입 불일치 등|
|MismatchedInputException|타입 불일치|예: 문자열을 숫자 타입으로 매핑|
|InvalidDefinitionException|잘못 정의된 클래스 구조|순환 참조, Getter/Setter 없음 등|
|IOException|I/O 기반 입력 처리 중 오류|파일 또는 스트림 처리 시|

---

## **실무 적용 포인트**

- API 응답 시 null 값 제거를 통해 깔끔한 JSON 출력 가능
    
- DTO ↔ Entity 변환 시 convertValue()를 적극 활용
    
- 스네이크 케이스 등 JSON 명명 규칙에 대응 가능
    
- 테스트나 로그 확인용 JSON 직렬화에 유용
    
- 역직렬화 예외를 통해 입력값 유효성 검증 효과도 가능