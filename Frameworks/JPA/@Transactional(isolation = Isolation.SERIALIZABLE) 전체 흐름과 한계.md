
## **@Transactional(isolation = Isolation.SERIALIZABLE) 전체 흐름과 한계**

동시성 이슈에 대해 실제로 이슈를 발생시켜보고 이를 해결해보는 과정속에서 비관적 락과 같은 명시적 락을 걸어야하는 부분이지만 "isolation = Isolation.SERIALIZABLE" 속성이 존재하여 테스트해보았다.  

해당 설정이 있으면 왜 비관적 락이나 명시적 락을 따로 사용하는지 궁금했기 때문이였다.  

이러한 과정을 기술한 문서이다.  

---

### **isolation = Isolation.SERIALIZABLE??**

```
@Transactional(isolation = Isolation.SERIALIZABLE)
public void someLogic() {
    // 비즈니스 로직
}
```

- 트랜잭션을 시작하면서
    
- DB에게 “이 트랜잭션을 SERIALIZABLE 격리 수준으로 처리해줘”라고 **요청**
    
- 이 요청을 기반으로 DB는 내부적으로 충돌을 최소화하기 위해 조치를 취함
    

**중요:** **이 설정이 곧바로 SELECT FOR UPDATE처럼 행 락을 강제하는 건 아님**

이부분 때문에 어떤 문제가 있는지 확인하는데 오래걸렸다.  

---

### **MySQL InnoDB 기준 실제 동작**

- SERIALIZABLE 모드가 적용되면:
    
    - 일반 SELECT도 내부적으로 **공유 락(Shared Lock)**을 걸거나
        
    - **필요하다면** 충돌을 막기 위한 추가적인 잠금이나 제어를 시도
        
    
- 하지만:
    
    - 개발자가 SELECT FOR UPDATE를 명시적으로 쓰는 것만큼 강력하고 확정적인 락은 아님
        
    - 락을 거는 기준과 시점은 DB 내부 최적화 로직에 따라 결정됨
        
    

---

### **동시성 시도시 교착상태 발생**



```
@Transactional(isolation = Isolation.SERIALIZABLE)
public void decreaseStock(Long id, Long quantity) {
    Stock stock = stockRepository.findById(id).orElseThrow();
    stock.decreaseStock(quantity);
    stockRepository.saveAndFlush(stock);
}
```

**상황 흐름**

1. 트랜잭션 A → findById → 공유 락 획득
    
2. 트랜잭션 B → findById → 공유 락 획득
    
3. A, B 모두 동시에 같은 행 읽기 성공 (공유 락은 읽기 허용)
    
4. A, B 모두 UPDATE 실행 시도 → Exclusive Lock 필요
    
5. 서로 상대방이 공유 락을 풀기 전까지 대기
    
6. 교착상태(Deadlock) 발생 → MySQL이 한 트랜잭션 강제 실패 처리
    

---

### **실무에서 오해하기 쉬운 부분**

|**오해**|**실제 동작**|
|---|---|
|SERIALIZABLE이면 자동으로 강한 락이 걸린다|공유 락 걸릴 수 있지만, 행에 확정적 Exclusive Lock은 아님|
|교착상태가 방지된다|공유 락 경쟁 → 쓰기 충돌로 오히려 교착상태 잘 발생|
|SELECT FOR UPDATE와 같다|아니다. 개발자가 직접 SELECT FOR UPDATE를 써야 확실함|

---

### **결론**

- @Transactional(isolation = Isolation.SERIALIZABLE)에만 의존하지 말고
    
- **명확한 비관적 락을 직접 명시**해야 함
    
**예시:**

```
@Transactional(isolation = Isolation.SERIALIZABLE)
public void decreaseStock(Long id, Long quantity) {
    Stock stock = stockRepository.findByIdForUpdate(id).orElseThrow();
    stock.decreaseStock(quantity);
    stockRepository.saveAndFlush(stock);
}
```

또는 JPA 기준

```
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select s from Stock s where s.id = :id")
Optional<Stock> findByIdForUpdate(@Param("id") Long id);
```

---

### **핵심 결론**

  

✅ @Transactional(SERIALIZABLE)은 DB에 강한 격리 수준 요청을 보내는 것

✅ 내부적으로 공유 락 걸릴 수 있으나 확정적 Exclusive Lock은 아님

✅ 공유 락만으론 동시성 문제 해결 불충분, 오히려 교착상태 유발 가능

✅ 확실한 제어를 원하면 **SELECT FOR UPDATE 또는 JPA의 비관적 락 필수**
