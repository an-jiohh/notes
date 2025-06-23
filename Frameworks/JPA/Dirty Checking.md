
## **Dirty Checking 이란?**

  

**영속성 컨텍스트가 관리하는 엔티티의 값이 변경됐는지 감지해서, 트랜잭션이 끝날 때 자동으로 UPDATE 쿼리를 날리는 기능**

  

즉, 개발자가 직접 UPDATE 쿼리를 작성하지 않아도,

**값만 바꾸고 트랜잭션을 커밋하면 DB에 반영**돼.

---

## **예시**

```
@Transactional
public void changeUserName(Long id, String newName) {
    User user = em.find(User.class, id);  // 1차 캐시에 영속 상태로 로딩

    user.setName(newName);                // 값만 변경

    // 별다른 코드 없어도 트랜잭션 종료 시 자동으로 UPDATE 발생
}
```

이 경우 JPA 내부에서:

- 기존 user의 스냅샷을 기억
    
- 값이 바뀌었는지 비교
    
- 바뀌었으면 UPDATE 쿼리 생성 후 flush
    

---

## **동작 흐름 (내부적으로)**

1. find() 또는 persist()로 엔티티를 영속성 컨텍스트에 넣음
    
2. 원본 스냅샷(초기 상태) 저장
    
3. 값 변경 감지(Setter 호출)
    
4. 트랜잭션 commit 또는 flush 시점에
    
5. 변경된 필드만 추적해 UPDATE 쿼리 자동 생성
    

---

## **특징**

  

✅ **개발자가 쿼리 직접 안 써도 됨**

✅ **변경된 필드만 Update 쿼리에 포함됨**

✅ **트랜잭션 안에서만 동작**

✅ @Transactional 필수

✅ **영속 상태(EntityManager가 관리하는 상태)**일 때만 동작

---

## **Dirty Checking이 안 되는 경우**

- @Transactional 없이 단순 메서드 호출
    
- em.detach() 등으로 비영속 상태로 만들었을 때
    
- merge()처럼 비영속 객체로 작업할 때 일부 통제 어려움
    

---

## **실무 주의사항**

- Dirty Checking 남발하면 원치 않는 쿼리가 날아갈 수도 있음
    
- 정확한 변경 의도를 드러내기 위해 Setter 대신 **명확한 변경 메서드 사용 권장**
		setter 사용시 사용의도가 분명치 않음 = update인지 create인지
    
- 대량 업데이트는 JPQL UPDATE나 Native 쿼리 쓰는 게 더 효율적임
    

---

## **정리**

|**구분**|**설명**|
|---|---|
|Dirty Checking|영속성 컨텍스트가 엔티티 값 변경을 감지해 자동 update|
|조건|트랜잭션 내, 영속 상태일 때만 동작|
|장점|쿼리 작성 필요 없음, 변경 부분만 update|
|단점/주의사항|비영속 상태에서는 동작 안 함, 불필요한 변경 주의 필요|
