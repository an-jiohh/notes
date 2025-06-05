
##  1.  트랜잭션이 commit될 때

  
@Transactional 메서드가 끝나는 순간 → 자동으로 flush() 발생

```
@Transactional
public void saveMemo() {
    Memo memo = new Memo("오늘도 열심히!");
    em.persist(memo);  // 아직 DB에 insert 안 됨 (영속성 컨텍스트에만 존재)
    
    // 여기까지는 flush 안 됨
} // 메서드 종료 시점 → 트랜잭션 commit → flush 자동 호출 → insert 쿼리 날림
```

---

##  2. JPQL, CriteriaQuery 실행 직전에

DB에서 select/update/delete 하려고 할 때, **DB 상태와 불일치 방지**를 위해 자동 flush

```
@Transactional
public void saveAndFind() {
    Memo memo = new Memo("flush 전 테스트");
    em.persist(memo);  // 아직 flush 안 됨

    // JPQL 실행 직전에 flush() 자동 호출됨
    List<Memo> result = em.createQuery("SELECT m FROM Memo m", Memo.class)
                          .getResultList();
}
```

 이때 memo는 DB에 반영되지 않은 상태지만, 쿼리 실행 직전에 flush가 일어나면서 DB에 반영되고 조회됨.

---

##  3. @GeneratedValue(strategy = IDENTITY) 전략 사용 시 persist() 직후

  

IDENTITY 전략은 DB가 ID를 생성하므로, persist() 시점에 바로 insert가 되어야 ID를 받을 수 있음 → **즉시 flush**

```
@Entity
public class Memo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 이 전략 때문에!
    private Long id;

    private String content;
}

@Transactional
public void saveWithIdentity() {
    Memo memo = new Memo("즉시 flush 테스트");
    em.persist(memo);  // insert 쿼리 즉시 발생 (flush 자동 호출)

    System.out.println(memo.getId());  // 여기서 이미 ID 값이 있음
}
```

즉시 insert 되는 이유는 DB로부터 **생성된 ID 값을 받아와야 하기 때문**

---

## 4. 강제로 flush() 호출한 경우

개발자가 명시적으로 호출할 수 있음


```
@Transactional
public void manualFlush() {
    Memo memo = new Memo("수동 flush");
    em.persist(memo);   // 아직 insert 안 됨
    em.flush();         // 이 시점에 insert 쿼리 발생

    // 이후 쿼리 없이도 DB에 반영된 상태
}
```

---

## 5. 연관 엔티티 저장 중 cascade 사용 시

자식 엔티티 저장 시 ID가 필요하여 flush가 자동 호출되기도 함  
주의: 이는 Hibernate의 내부 최적화 방식에 따라 다름)

```
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "user", cascade = CascadeType.PERSIST)
    private List<Memo> memos = new ArrayList<>();
}

@Transactional
public void cascadeFlush() {
    User user = new User();
    Memo memo = new Memo("cascade 테스트");
    user.getMemos().add(memo);
    memo.setUser(user);

    em.persist(user);  // memo까지 같이 persist 되며, IDENTITY로 인해 flush 발생
}
```

---

## 6. 트랜잭션 롤백 조건 판단 시 (비교적 드물게)

  예외 발생 후 트랜잭션 처리 전에 flush가 필요한 경우도 있으나 일반적으로 직접 체감하기 어려움

---

##  정리

|**상황**|**자동 flush 발생 여부**|
|---|---|
|트랜잭션 commit 직전|✅ 발생|
|JPQL 쿼리 실행 전|✅ 발생|
|IDENTITY 전략 persist 직후|✅ 발생|
|cascade 연관 persist|경우에 따라 ✅ 발생|
|flush() 직접 호출|✅ 발생|
|단순 persist 후 아무 작업 없음|❌ 발생하지 않음|

---
