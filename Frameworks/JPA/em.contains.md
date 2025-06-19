em.contains(entity)는 **JPA의 EntityManager에서 해당 엔티티 객체가 현재 영속성 컨텍스트에 포함되어 있는지 여부**를 확인하는 메서드


## **기본 개념**

```
boolean contains(Object entity)
```

- 인자로 전달된 객체(entity)가 **현재 영속성 컨텍스트에 관리되고 있는지**를 반환합니다.
    
- true: 영속 상태 (managed 상태)
    
- false: 비영속 (new), 준영속(detached), 삭제된 상태 등
    
## **사용 목적**

- remove() 같은 작업을 하기 전에, 해당 엔티티가 영속 상태인지 확인해서 **예외를 방지**하거나 **적절한 처리를 하기 위해** 사용합니다.
    
- 예를 들어, 영속 상태가 아니라면 merge()로 관리 상태로 전환한 뒤에 remove()합니다.
    

  
---
## **예제**

```
if (em.contains(refreshToken)) {
    em.remove(refreshToken);
} else {
    RefreshToken managedToken = em.merge(refreshToken);
    em.remove(managedToken);
}
```

- 위 코드는 refreshToken이 영속 상태인지 먼저 체크하고,
    
- 비영속 상태라면 merge()로 영속 상태로 만든 후 제거합니다.
    

## **왜 필요할까?**

- em.remove()는 **영속 상태의 엔티티만 삭제**할 수 있기 때문에, **비영속 상태에서 remove()를 호출하면 예외**가 발생합니다.
    
- 따라서 contains()로 체크한 뒤, 필요하면 merge()를 통해 영속화해야 안전하게 삭제할 수 있습니다.
    


## **참고: 영속성 상태 비교**

|**상태**|**설명**|em.contains() **결과**|
|---|---|---|
|영속(managed)|트랜잭션 안에서 persist된 상태|true|
|비영속(new)|아직 persist되지 않은 새 객체|false|
|준영속(detached)|DB와 연결 끊긴 상태 (예: em.detach())|false|
|삭제(removed)|remove()된 상태|false|
