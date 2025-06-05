

###  em.persist() 후 INSERT 쿼리가 실제로 나가는 시점 (= flush 발생 시점)

  

em.persist(entity)는 영속성 컨텍스트에 객체를 등록만 하고, 실제로 DB에 반영되지는 않습니다.

**DB에 INSERT 쿼리가 나가는 시점은 flush 시점**입니다.

  

####  Flush가 자동으로 발생하는 경우

  

다음과 같은 시점에 em.flush()가 **자동으로 호출**됩니다:

1. **트랜잭션 커밋 시점**
    
    - @Transactional이 붙은 메서드가 정상 종료 → 트랜잭션 커밋 직전
        
    - 이때 flush가 일어나고, INSERT 쿼리가 DB에 날아감
        
    
2. **JPQL/QueryDSL/NativeQuery 실행 전**
    
    - DB로부터 무언가를 조회하거나 쿼리를 실행할 때
        
    - JPA는 쿼리의 정확성을 보장하기 위해 flush를 수행함
        
    
3. **em.flush()를 직접 호출한 경우**
    
    - 강제로 flush할 수도 있음 (예: em.flush())
        
    
4. **영속성 컨텍스트가 플러시 모드 AUTO이고 강제 동기화가 필요한 경우**
    
    - FlushModeType.AUTO(기본값)에서는 위와 같은 트리거에서 자동 flush가 발생함
        
    

---

### **예시 코드 흐름**

```
@Transactional
public void saveMemo() {
    Memo memo = new Memo("내용");
    em.persist(memo);  // INSERT 쿼리 안 나감 (영속성 컨텍스트에만 등록)

    // 여기서 SELECT 쿼리를 실행하면 자동 flush 발생
    List<User> users = em.createQuery("SELECT u FROM User u", User.class).getResultList();
    
    // 혹은 메서드가 끝나면 트랜잭션 커밋 시 flush
}
```

---

정리

|**구분**|**자동 flush 발생 여부**|**INSERT 쿼리 발생 여부**|
|---|---|---|
|em.persist() 호출 직후|❌|❌|
|쿼리 실행 직전 (JPQL 등)|✅|✅|
|트랜잭션 커밋 시|✅|✅|
|em.flush() 수동 호출|✅|✅|
