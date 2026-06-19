# **`@ElementCollection`**

**정리**

## **1. 개념과 특징**

`@ElementCollection`은 엔티티가 아닌 값 타입 컬렉션을 매핑할 때 사용하는 JPA 어노테이션

예시:

```java
@ElementCollection(fetch = FetchType.EAGER)
@CollectionTable(name = "lab_prerequisites", joinColumns = @JoinColumn(name = "lab_id"))
@OrderColumn(name = "prerequisite_order")
@Column(name = "prerequisite", nullable = false, length = 1000)
private List<String> prerequisites = new ArrayList<>();
```

위 코드는 `Lab` 엔티티가 여러 개의 선행 조건 문자열을 가지도록 설정한 구조

  

`prerequisites`는 별도 엔티티가 아닌 `String` 값들의 목록

```java
private List<String> prerequisites;
```

각 항목은 독립적인 엔티티가 아니라 단순 문자열 값

```text
"Java 기본 문법 이해"
"Spring Boot 실행 가능"
"HTTP API 개념 이해"
```

따라서 `@OneToMany`가 아닌 `@ElementCollection` 사용

  

`@OneToMany`는 엔티티와 엔티티 사이의 관계 매핑  
`@ElementCollection`은 엔티티가 아닌 값 타입 목록 매핑

  

값 타입 컬렉션의 각 값에는 별도 ID 없음  
각 값은 주인 엔티티인 `Lab`에 종속됨

  

즉, `prerequisite` 값은 `Lab` 없이 독립적으로 존재하기 어려움`Lab`이 저장되면 함께 저장되고, `Lab`이 삭제되면 함께 삭제되는 구조

  

개념적으로 부모 엔티티의 생명주기에 종속된 데이터

```text
Lab 저장 → prerequisites도 함께 저장
Lab 삭제 → prerequisites도 함께 삭제
Lab 컬렉션 수정 → lab_prerequisites 테이블 반영
```

단, `cascade` 옵션을 직접 설정하는 구조는 아님  
`cascade`는 엔티티 간 연관관계에서 사용하는 옵션  
`@ElementCollection`의 대상은 엔티티가 아닌 값 타입이므로 별도 `cascade` 설정 대상 아님

---

## **2. 관계형 DB 저장 구조**

`List<String>` 값들은 `Lab` 테이블에 직접 저장되지 않고 별도 테이블에 저장

### 

### **`labs`**

**테이블**

|**id**|**title**|
|---|---|
|1|Redis 장애 복구 실습|

### 

### **`lab_prerequisites`**

**테이블**

|**lab_id**|**prerequisite_order**|**prerequisite**|
|---|---|---|
|1|0|Java 기본 문법 이해|
|1|1|Spring Boot 실행 가능|
|1|2|HTTP API 개념 이해|

`labs` 테이블에는 `Lab` 자체 정보 저장`lab_prerequisites` 테이블에는 선행 조건 목록 저장

  

관계 구조:

```text
labs
+----+----------------------+
| id | title                |
+----+----------------------+
| 1  | Redis 장애 복구 실습 |
+----+----------------------+

          1
          |
          | lab_id
          ↓

lab_prerequisites
+--------+--------------------+------------------------+
| lab_id | prerequisite_order | prerequisite           |
+--------+--------------------+------------------------+
| 1      | 0                  | Java 기본 문법 이해    |
| 1      | 1                  | Spring Boot 실행 가능  |
| 1      | 2                  | HTTP API 개념 이해     |
+--------+--------------------+------------------------+
```

즉, 자바에서는 하나의 `List<String>` 구조

```java
private List<String> prerequisites;
```

DB에서는 여러 개의 row로 저장

```text
Lab 하나
→ lab_prerequisites 테이블의 여러 행
```

---

## **3. 어노테이션별 역할**

### **`@ElementCollection`**

```java
@ElementCollection(fetch = FetchType.EAGER)
```

값 타입 컬렉션 매핑

  

`String`, `Integer`, 임베디드 타입 등 엔티티가 아닌 값들을 컬렉션으로 저장할 때 사용

  

`fetch = FetchType.EAGER`는 `Lab` 조회 시 `prerequisites`도 즉시 조회한다는 의미

  

예시:

```java
Lab lab = labRepository.findById(id).get();
```

위 코드 실행 시 `Lab`뿐만 아니라 `prerequisites` 목록도 함께 조회

  

단, 데이터가 많아질 경우 성능 부담 가능  
실무에서는 `LAZY` 사용 고려

```java
@ElementCollection(fetch = FetchType.LAZY)
```

---

### **`@CollectionTable`**

```java
@CollectionTable(
    name = "lab_prerequisites",
    joinColumns = @JoinColumn(name = "lab_id")
)
```

값 타입 컬렉션을 저장할 별도 테이블 지정

`name = "lab_prerequisites"`  
→ 컬렉션 저장 테이블 이름

`joinColumns = @JoinColumn(name = "lab_id")`→ 주인 엔티티인 `Lab`과 연결되는 외래 키 컬럼 이름

---

### **`@JoinColumn`**

```java
@JoinColumn(name = "lab_id")
```

컬렉션 테이블에서 `Lab`을 참조하는 컬럼 지정

  

관계 구조:

```text
labs.id  ←  lab_prerequisites.lab_id
```

`lab_id`를 통해 특정 선행 조건이 어떤 `Lab`에 속하는지 구분

---

### **`@OrderColumn`**

```java
@OrderColumn(name = "prerequisite_order")
```

`List`의 순서 저장

  

`List`는 순서가 중요한 컬렉션  
DB는 기본적으로 행의 순서를 보장하지 않음  
따라서 순서를 저장할 별도 컬럼 필요

  

예시:

|**lab_id**|**prerequisite_order**|**prerequisite**|
|---|---|---|
|1|0|Java 기본 문법 이해|
|1|1|Spring Boot 실행 가능|
|1|2|HTTP API 개념 이해|

`prerequisite_order` 값으로 리스트 순서 유지

---

### **`@Column`**

```java
@Column(name = "prerequisite", nullable = false, length = 1000)
```

실제 문자열 값이 저장될 컬럼 설정

`name = "prerequisite"`  
→ 컬럼 이름 지정

`nullable = false`→ `NULL` 저장 불가

`length = 1000`  
→ 최대 길이 1000자

---

## **4. 값 타입 컬렉션의 제약**

값 타입 컬렉션은 편리하지만, 엔티티가 아니기 때문에 몇 가지 제약 존재

### **1. 각 값에 독립 ID 없음**

값 타입 컬렉션의 각 항목은 단순 값

```java
List<String> prerequisites;
```

따라서 각 항목에 별도 ID 없음

```text
"Java 기본 문법 이해"
"Spring Boot 실행 가능"
"HTTP API 개념 이해"
```

엔티티라면 각 항목에 ID를 둘 수 있음

|**id**|**lab_id**|**content**|
|---|---|---|
|10|1|Java 기본 문법 이해|
|11|1|Spring Boot 실행 가능|
|12|1|HTTP API 개념 이해|

하지만 값 타입 컬렉션은 다음과 같은 구조

|**lab_id**|**prerequisite_order**|**prerequisite**|
|---|---|---|
|1|0|Java 기본 문법 이해|
|1|1|Spring Boot 실행 가능|
|1|2|HTTP API 개념 이해|

각 prerequisite 자체를 식별하는 별도 ID 없음

  

따라서 다음과 같은 개별 관리 구조 어려움

```java
labPrerequisiteRepository.findById(1L)
```

항상 주인 엔티티인 `Lab`을 통해 접근

```java
Lab lab = labRepository.findById(1L).get();
lab.getPrerequisites().set(0, "Java 기본 문법과 객체지향 이해");
```

---

### **2. 변경 추적 어려움**

값 타입 컬렉션은 각 값에 고유 ID가 없기 때문에 JPA가 특정 값 하나의 변경을 세밀하게 추적하기 어려움

예시:

```java
lab.getPrerequisites().set(1, "Spring Boot 프로젝트 실행 가능");
```

변경 전:

```text
[Java 기본 문법 이해, Spring Boot 실행 가능, HTTP API 개념 이해]
```

변경 후:

```text
[Java 기본 문법 이해, Spring Boot 프로젝트 실행 가능, HTTP API 개념 이해]
```

사람 입장에서는 두 번째 값만 변경된 상황

  

하지만 JPA 입장에서는 값들의 묶음이 변경된 상황  
엔티티처럼 `id = 11`인 항목만 수정한다고 판단하기 어려움

  

따라서 컬렉션 변경 시 기존 데이터를 삭제하고 현재 컬렉션 값을 다시 저장하는 방식 발생 가능

```sql
DELETE FROM lab_prerequisites
WHERE lab_id = 1;

INSERT INTO lab_prerequisites (lab_id, prerequisite_order, prerequisite)
VALUES (1, 0, 'Java 기본 문법 이해');

INSERT INTO lab_prerequisites (lab_id, prerequisite_order, prerequisite)
VALUES (1, 1, 'Spring Boot 프로젝트 실행 가능');

INSERT INTO lab_prerequisites (lab_id, prerequisite_order, prerequisite)
VALUES (1, 2, 'HTTP API 개념 이해');
```

데이터가 적으면 큰 문제 없음  
컬렉션 크기가 크거나 수정이 자주 발생하면 성능 부담 가능

---

### 

### 

### **3. 기본 키,**

**`null`**

**, 중복 제약**

값 타입 컬렉션 테이블에는 보통 별도 `id` 컬럼 없음  
따라서 여러 컬럼을 묶어 row 식별

예시:

```sql
PRIMARY KEY (lab_id, prerequisite_order)
```

또는 `Set<String>` 구조에서는 다음과 같은 형태 가능

```sql
PRIMARY KEY (lab_id, prerequisite)
```

이처럼 여러 컬럼을 묶어 기본 키를 구성하는 방식  
이를 복합 기본 키라고 부름

  

기본 키에 포함되는 컬럼은 `NULL` 불가

  

예시:

```sql
PRIMARY KEY (lab_id, prerequisite)
```

위 구조에서는 `lab_id`, `prerequisite` 모두 `NULL` 불가

  

네 코드에서도 아래 설정으로 `NULL` 저장 제한

```java
@Column(name = "prerequisite", nullable = false, length = 1000)
```

따라서 다음과 같은 값 저장 불가

```java
lab.getPrerequisites().add(null);
```

또한 기본 키는 중복 불가

  

예를 들어 `Set<String>` 구조에서 기본 키가 다음과 같다고 가정

```sql
PRIMARY KEY (lab_id, prerequisite)
```

이 경우 아래 데이터 저장 불가

|**lab_id**|**prerequisite**|
|---|---|
|1|Java 기본 문법 이해|
|1|Java 기본 문법 이해|

기본 키 조합이 동일하기 때문

```text
(1, "Java 기본 문법 이해")
(1, "Java 기본 문법 이해")
```

단, 현재 코드처럼 `List + @OrderColumn` 구조에서는 다르게 동작 가능

```sql
PRIMARY KEY (lab_id, prerequisite_order)
```

이 경우 같은 문자열이라도 순서 값이 다르면 저장 가능

|**lab_id**|**prerequisite_order**|**prerequisite**|
|---|---|---|
|1|0|Java 기본 문법 이해|
|1|1|Java 기본 문법 이해|

기본 키 조합이 다름

```text
(1, 0)
(1, 1)
```

따라서 `List + @OrderColumn` 구조에서는 중복 문자열 저장 가능  
다만 일반적인 값 타입 컬렉션 설명에서는 기본 키 구조에 따른 `null`, 중복 제약 주의 필요

---

### **4. 순서 변경 비용**

`@OrderColumn` 사용 시 리스트 순서를 DB에 저장

중간 삽입 또는 삭제 시 뒤쪽 항목들의 순서 값 변경 필요

예시:

```java
lab.getPrerequisites().add(1, "객체지향 기본 개념 이해");
```

기존 구조:

|**lab_id**|**prerequisite_order**|**prerequisite**|
|---|---|---|
|1|0|Java 기본 문법 이해|
|1|1|Spring Boot 실행 가능|
|1|2|HTTP API 개념 이해|

중간 삽입 후:

|**lab_id**|**prerequisite_order**|**prerequisite**|
|---|---|---|
|1|0|Java 기본 문법 이해|
|1|1|객체지향 기본 개념 이해|
|1|2|Spring Boot 실행 가능|
|1|3|HTTP API 개념 이해|

기존 `prerequisite_order` 값 변경 발생 가능  
중간 삽입, 삭제가 자주 발생하면 update 비용 증가 가능

---

### **5. 확장성 부족**

현재 구조는 문자열만 저장 가능

추후 다음과 같은 속성이 필요하면 한계 발생

```text
선행 조건별 난이도
선행 조건별 필수/선택 여부
선행 조건별 링크
선행 조건별 설명
선행 조건별 생성일/수정일
선행 조건별 체크 여부
```

이런 경우 별도 엔티티 분리 필요

---

## **5. 별도 엔티티로 분리해야 하는 경우**

단순 문자열 목록이면 `@ElementCollection` 적합

```java
@ElementCollection
private List<String> prerequisites;
```

하지만 각 항목에 의미나 속성이 붙으면 별도 엔티티로 분리하는 구조 권장

  

예시:

```java
@Entity
public class LabPrerequisite {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    private Lab lab;

    private String content;

    private boolean required;

    private int sortOrder;
}
```

DB 구조:

|**id**|**lab_id**|**content**|**required**|**sort_order**|
|---|---|---|---|---|
|1|1|Java 기본 문법 이해|true|0|
|2|1|Spring Boot 실행 가능|true|1|
|3|1|HTTP API 개념 이해|false|2|

이 구조에서는 각 prerequisite를 독립적으로 식별 가능  
개별 수정, 삭제, 조회 가능  
속성 확장에도 유리

---

## **6. 선택 기준**

### 

### **`@ElementCollection`**

**사용 적합**

```text
단순 값 목록
부모 엔티티에 완전히 종속된 데이터
개별 ID 필요 없음
개별 조회/수정 필요 없음
데이터 수가 많지 않음
수정 빈도가 낮음
```

예시:

```text
선행 조건 문자열 목록
태그 목록
간단한 키워드 목록
짧은 설명 목록
```

---

### **별도 엔티티 분리 권장**

```text
각 항목에 ID 필요
각 항목을 독립적으로 관리
각 항목에 추가 속성 필요
검색/통계/정렬 조건이 복잡
수정 빈도가 높음
데이터 수가 많음
```

예시:

```text
선행 조건별 난이도 관리
선행 조건별 필수 여부 관리
선행 조건별 링크 제공
선행 조건별 완료 여부 체크
선행 조건 개별 수정 API 제공
```

---

## **7. 핵심 정리**

`@ElementCollection`은 엔티티가 아닌 값 타입 컬렉션을 별도 테이블에 저장하는 방식

`List<String>` 같은 단순 값 목록을 저장할 때 편리함

값은 주인 엔티티에 종속됨  
부모 저장, 삭제, 수정에 따라 함께 관리됨  
별도 cascade 설정 대상 아님

하지만 각 값에 고유 ID 없음  
개별 값 변경 추적 어려움  
변경 시 전체 삭제 후 재삽입 발생 가능  
기본 키 구성상 `null`과 중복 제약 주의 필요  
데이터가 커지거나 항목별 속성이 필요하면 별도 엔티티로 분리 필요

최종 기준:

```text
단순 문자열 목록 → @ElementCollection
의미 있는 하위 데이터 → 별도 엔티티
```