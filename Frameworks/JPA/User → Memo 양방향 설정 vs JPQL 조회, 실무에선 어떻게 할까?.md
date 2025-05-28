
# JPA 연관관계 설계: `User → Memo` 양방향 설정 vs JPQL 조회, 실무에선 어떻게 할까?

예를 들어 `User`와 `Memo`가 1:N 관계일 때, `User`에 `List<Memo>`를 두고 **양방향 연관관계 + LAZY 로딩**으로 설정할지,  
아니면 **단방향 + JPQL로 조회**하는 방식으로 설계할지 결정해야 함.

이번 글에서는 이 두 방식의 장단점을 비교하고, 실무에서 어떤 기준으로 선택해야 하는지 정리해 봄.

---

## 전제

- 하나의 `User`는 여러 개의 `Memo`를 가질 수 있음 (1:N 관계)
- 현재 `Memo`는 `@ManyToOne`으로 `User`를 참조하고 있음 → 즉, 연관관계의 주인은 `Memo`임
- `User`에서 `Memo`를 직접 가져올 필요가 있는지 고민해야 하는 상황임

---

## 선택지 1. 양방향 연관관계 + LAZY 로딩

```java
@Entity
public class User {
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Memo> memos = new ArrayList<>();
}
````

이렇게 설정하면 user.getMemos()로 메모 리스트를 바로 가져올 수 있어 객체지향적으로는 깔끔함.

하지만 다음과 같은 문제점들이 발생할 수 있음.

  

### **1. 의도치 않은 쿼리 발생**

```
User user = userRepository.findById(1L).get();
user.getMemos().size(); // 이 시점에 쿼리 발생
```

→ 개발자가 Memo를 로딩하려는 의도가 없었더라도, getMemos()를 호출하면서 추가 쿼리가 발생함

  

### **2. N+1 문제**

```
List<User> users = userRepository.findAll();
for (User user : users) {
    System.out.println(user.getMemos().size()); // 사용자 수만큼 쿼리 발생
}
```

→ 유저가 100명이라면 User 1번 + Memo 100번 = 총 101번 쿼리 실행됨

→ 성능 저하와 디버깅 난이도 상승

  

### **3. 연관관계 복잡도 증가**

- User와 Memo 모두 연관 필드를 가짐 → 양방향
    
- mappedBy, 연관관계 편의 메서드 작성 등 관리할 코드가 많아짐
    
- 연관관계 동기화를 놓칠 경우 의도치 않은 상태 발생 가능
    

---

## **선택지 2. 단방향 연관관계 + JPQL 조회**

  

Memo만 User를 참조하고, User에서는 Memo에 대한 필드를 아예 두지 않음.

필요할 때는 MemoRepository에서 JPQL을 사용해 조회함.

```
public interface MemoRepository extends JpaRepository<Memo, Long> {
    List<Memo> findByUserId(Long userId);
}
```

또는

```
@Query("select m from Memo m where m.user.id = :userId")
List<Memo> findMemosByUserId(@Param("userId") Long userId);
```

### **장점**

- 연관관계가 단순해짐 → 도메인 설계가 깔끔해짐
    
- 성능 제어가 쉬움 → 필요한 시점에만 쿼리를 명시적으로 날릴 수 있음
    
- N+1 문제나 의도치 않은 지연 로딩 쿼리를 원천 차단 가능
    
- 유지보수가 편해짐 → 의도한 쿼리만 나가므로 디버깅이 쉬움
    

  

### **단점**

- user.getMemos() 같은 직관적인 탐색이 불가능함
    
- 항상 별도의 Repository 메서드를 통해 메모를 조회해야 함
    

---

## **정리: 언제 어떤 방식을 선택해야 하는가?**

|**상황**|**적합한 방식**|
|---|---|
|메모 조회가 자주 일어나지 않음|단방향 + JPQL 조회|
|도메인 로직에서 메모에 자주 접근함|양방향 + LAZY 로딩|
|리스트 조회 시 성능이 중요함|단방향 + fetch join 사용|
|설계의 단순함과 명확성이 중요함|단방향 추천|

---

## **실무 기준에서의 결론**

- User → Memo 양방향 연관관계는 **정말 필요한 경우가 아니라면 피하는 것이 좋음**
- 대부분의 조회는 **JPQL 또는 Spring Data JPA의 메서드 쿼리로 해결하는 편이 낫다**고 판단됨
- **불필요한 연관관계는 객체 설계를 복잡하게 만들고, 성능 이슈를 유발할 가능성이 큼**
- 연관관계를 줄이고 명시적 쿼리로 관리하는 것이 **예측 가능한 시스템 설계**로 이어짐

---

## **부록: 실무 팁**

- 쿼리 최적화가 필요한 경우 @EntityGraph, fetch join, @BatchSize 등을 활용할 수 있음
- open-in-view가 true일 경우 view 렌더링 중에 쿼리가 나갈 수 있음 → 설정에 유의할 것
- DTO 변환 시 getMemos().size() 같은 접근은 무심코 N+1 문제를 유발할 수 있음