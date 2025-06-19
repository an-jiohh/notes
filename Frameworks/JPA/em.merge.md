em.merge()는 JPA에서 **비영속(detached) 상태의 엔티티를 영속(managed) 상태로 다시 변경**할 때 사용하는 메서드

---

## **기본 시그니처**

```
<T> T merge(T entity);
```

- 인자로 전달된 entity는 비영속 상태일 수 있습니다.
    
- 반환값은 **새로 영속 상태로 등록된 엔티티 인스턴스**입니다 (주의: **원본 객체와는 다름**).
    

---

## **사용 목적**

- **DB에서 가져온 엔티티가 영속성 컨텍스트에서 분리(detached)되었을 때**, 그 변경사항을 다시 반영하고 싶을 때 사용
    
- **새로운 트랜잭션 범위 안에서 업데이트하려는 객체를 다시 영속화**할 때
    

---

## **예제**

```
// 1. 트랜잭션 A에서 엔티티를 조회
Member member = em.find(Member.class, 1L);

// 2. 트랜잭션 A 종료 → member는 detached 상태가 됨
// 3. 이후 상태에서 필드 수정
member.setName("홍길동");

// 4. 트랜잭션 B에서 다시 영속성 컨텍스트에 반영하고 싶을 때
Member mergedMember = em.merge(member);

// 5. 주의: mergedMember가 영속 상태이고,
//         기존 member 객체는 여전히 detached 상태임
```

---

## **주의할 점**

|**항목**|**설명**|
|---|---|
|**반환 객체**|merge()는 **새로운 영속 객체**를 반환함 (기존 객체와 다름)|
|**기존 객체는 여전히 detached**|변경 내용은 복사되지만, 원래 인스턴스는 관리되지 않음|
|**null을 merge하면**|IllegalArgumentException 발생|
|**없는 ID로 merge 시도 시**|새 엔티티로 인식되어 insert 쿼리가 발생할 수 있음|

---

## **실무에서 자주 쓰는 패턴**

```
if (em.contains(entity)) {
    em.remove(entity);
} else {
    em.remove(em.merge(entity));  // merge로 영속화 후 삭제
}
```

또는

```
SomeEntity managed = em.merge(detachedEntity);
```

이후 managed 객체로 업데이트/삭제 수행

---

## **언제 쓰면 안 될까?**

- 굳이 merge()하지 않고도 트랜잭션 안에서 persist한 객체는 그대로 쓰는 게 낫습니다.
    
- 실수로 new 객체를 merge()하면 예상치 못한 insert가 일어날 수 있으므로 주의해야 합니다.
    