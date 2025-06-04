# @PersistenceContext

###  들어가며


Spring에서 JPA를 사용하는 도중 EntityManager을 주입받던 중 생각없이 @Autowired를 쓰고있었다.  
정신을 차리고 @PersistenceContext으로 바꾸던 와중에 왜 사용하지에 대한 의문이 들어 정리한 글이다.

## @PersistenceContext
Java EE 및 Spring에서 **JPA(EntityManager)**를 사용할 때 **영속성 컨텍스트(EntityManager)**를 주입하기 위해 사용되는 어노테이션


EntityManager은 하나의 트랜잭션 범위 내에서 엔티티를 관리하게 되는데 이 때문에 Thread-safe하게 작동하여야함  
즉 각 트랜잭션에 따라서 EntityManager가 사용되어야하는데 이때 쓰레드를 감지해서 같은 EntitiyManager을 주입해주는 것이 @PersistenceContext  
트랜잭션이 열리면 같은 EntityManager가 사용되고, 트랜잭션이 끝나면 자동으로 정리된다.

@Autowired와 다른 점은  
@PersistenceContext는 **매 트랜잭션에 맞는 EntityManager를 동적으로 연결**해줌 → 트랜잭션과 딱 맞게 동작한다는 것

---

## **내부 동작 정리**

1. @PersistenceContext를 붙이면,
2. Spring이 **프록시 객체**(대리자)를 대신 주입줌
3. 이 프록시는 실행 중인 트랜잭션이 시작될 때,
4. **현재 트랜잭션에 맞는 진짜 EntityManager**를 찾아 연결해줌
5. 트랜잭션이 끝나면 그 EntityManager는 정리됨

---

## 예시 1 – 동일 트랜잭션인데 값이 없네?

```
@Service
public class UserService {

    @Autowired
    private EntityManager em1;

    @Autowired
    private EntityManager em2;

    @Transactional
    public void testSameTransaction() {
        User user = new User("liam");
        em1.persist(user);

        boolean isContained = em2.contains(user);
        System.out.println("em2 contains user? " + isContained);  // false가 출력될 수 있음
    }
}
```

### **📌 문제점**

- em1과 em2는 @Autowired로 주입되었지만, 내부적으로는 **다른 EntityManager 인스턴스**일 수 있음.
- 즉, em1이 관리 중인 객체를 em2는 모른다고 판단함 → contains() 결과가 false.

---

## **🧪 예시 2 – 트랜잭션 전파 분리로 detach 발생**

  

### **💥 코드 예시**

  

#### **ServiceA**

```
@Service
public class ServiceA {

    @Autowired
    private EntityManager emA;

    @Autowired
    private ServiceB serviceB;

    @Transactional
    public void run() {
        User user = new User("liam");
        emA.persist(user);

        // 새로운 트랜잭션에서 동일 객체 로딩
        serviceB.load(user.getId());

        // 다시 삭제 시도
        emA.remove(user); // ❗ DetachedEntityException 가능성
    }
}
```

#### **ServiceB**

```
@Service
public class ServiceB {

    @Autowired
    private EntityManager emB;

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void load(Long id) {
        User user = emB.find(User.class, id);
        System.out.println("ServiceB loaded user = " + user.getName());
    }
}
```

### **🧨 발생 가능한 오류**

- ServiceB는 **다른 트랜잭션과 다른 EntityManager**를 사용할 수 있음.
    
- user가 ServiceB에서 DB에 반영되면서 flush()되면, 기존 ServiceA에서는 user가 **detached 상태**가 됨.
    
- 따라서 remove(user)에서 IllegalArgumentException, DetachedEntityException이 발생할 수 있음.
    

---

## **🧠 핵심 요약**

| **항목**                       | **@Autowired EntityManager** | **@PersistenceContext EntityManager** |
| ---------------------------- | ---------------------------- | ------------------------------------- |
| 주입 방식                        | DI 기반 (Spring Bean으로 주입)     | JPA 명세 기반 (프록시 기반 주입)                 |
| 트랜잭션 연동                      | 연동 보장되지 않음                   | 트랜잭션 범위 내에서 안전하게 공유                   |
| 같은 트랜잭션 내 contains()         | false일 수 있음                  | true                                  |
| 전파(REQUIRES_NEW) 간 detach 문제 | 발생 가능                        | 최소화 가능                                |
| 실무 권장 여부                     | X                            | O                                     |
