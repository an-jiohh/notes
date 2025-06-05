# JPA JPQL은 엔티티의 필드와 관계를 기준으로
  

---

## 상황 예시: Memo와 User 엔티티

  

예를 들어 다음과 같이 Memo 엔티티가 있고, 유저와 다대일 연관관계를 가지고 있다고 가정

```
@Entity
public class Memo {
    @Id @GeneratedValue
    private Long id;

    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    private User user;

    private LocalDateTime createAt;
}
```

이제 특정 유저가 작성한 메모 목록을 JPQL로 조회하고자함

---

## 방법 1: m.user.id = :userId

```
String jpql = "SELECT m FROM Memo m WHERE m.user.id = :userId";
List<Memo> memos = em.createQuery(jpql, Memo.class)
                     .setParameter("userId", userId)
                     .getResultList();
```

- m.user는 Memo의 필드명으로 User 객체와의 연관관계    
- .id는 User 객체의 기본키를 의미
- JPA는 이를 SQL로 변환할 때 자동으로 user_id 컬럼을 기준으로 쿼리를 생성

  

💡 **실제로 실행되는 SQL 예시:**

```
SELECT m1_0.id, m1_0.content, m1_0.create_at, m1_0.user_id 
FROM memos m1_0 
WHERE m1_0.user_id = ?
```

---

## 방법 2:m.user = :user(객체 비교)

```
User user = em.find(User.class, userId);
String jpql = "SELECT m FROM Memo m WHERE m.user = :user";
List<Memo> memos = em.createQuery(jpql, Memo.class)
                     .setParameter("user", user)
                     .getResultList();
```

- 이 방법은 User 객체를 직접 비교하는 방식
    
- JPA는 내부적으로 user.getId()를 사용해 SQL로 변환
    
- 이미 User 객체가 있는 상황이라면 간결하게 사용할 수 있음
    

  

⚠ 하지만 User 객체를 먼저 조회해야 하므로 **추가적인 DB 접근이 발생할 수 있음

---

## 잘못된 방법: m.user_id = :userId

```
String jpql = "SELECT m FROM Memo m WHERE m.user_id = :userId"; // ❌
```

- user_id는 **DB 테이블 컬럼명**
    
- JPQL은 **객체 지향 쿼리 언어이므로 엔티티의 필드명**을 기준으로 작성
    
- 위 쿼리는 컴파일은 되지만 **실행 시 예외가 발생**


---

## **🛠 실제 DB 컬럼명을 쓰고 싶다면? → Native Query 사용**

  

DB 쿼리를 그대로 작성해야 할 이유가 있다면 nativeQuery를 사용하는 것으로

```
@Query(value = "SELECT * FROM memos WHERE user_id = :userId", nativeQuery = true)
List<Memo> findByUserId(@Param("userId") Long userId);
```

- 테이블명, 컬럼명 등 **DB 구조에 강하게 의존**하므로, 유지보수에 주의가 필요
- 가능하면 JPQL로 처리하는 것이 더 안전
