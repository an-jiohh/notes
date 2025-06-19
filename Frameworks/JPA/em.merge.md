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

---

## 실행시 DB조회 여부

#### **1. 기존에 존재하는 ID (DB에 해당 PK가 있음)**

```
Member detached = new Member();
detached.setId(1L);
detached.setName("홍길동");
em.merge(detached);
```

이 경우:

- merge()는 id = 1L인 엔티티가 **DB에 실제로 존재하는지 확인하기 위해 SELECT 쿼리를 수행**
    
- **DB에서 기존 데이터를 조회한 후**, 그 값과 detached 객체의 값을 병합
    
- 최종적으로는 **managed 객체를 반환하고**, 변경된 값이 있으면 update가 나감
    

즉, **DB 조회 O → UPDATE 쿼리**

#### **2. ID는 있지만 DB에 없음 (없는 PK로 merge 시도)**

```
Member detached = new Member();
detached.setId(9999L);  // 존재하지 않는 ID
detached.setName("홍길동");
em.merge(detached);
```

이 경우:

- SELECT로 먼저 조회함 → **조회 결과 없음**
    
- JPA는 “이건 새로운 엔티티인가 보다”라고 판단하고
    
- INSERT를 시도함 (실제로 DB에 없는 ID지만 강제 삽입)
    

  

즉, **DB 조회 O → INSERT 쿼리**

#### **3. ID가 없는 경우 (즉,** **@Id == null)**

```
Member detached = new Member();
detached.setName("홍길동");
em.merge(detached);
```

이 경우:

- ID가 없으므로 DB 조회 없이 바로 새로 삽입함
    
- 즉, persist()와 유사한 동작을 함
    

  

즉, **DB 조회 X → INSERT 쿼리**
#### **요약**

| **상황**       | **DB 조회 (SELECT)** | **결과 쿼리** |
| ------------ | ------------------ | --------- |
| 존재하는 ID      | O                  | UPDATE    |
| 없는 ID        | O                  | INSERT    |
| ID 없음 (null) | X                  | INSERT    |
