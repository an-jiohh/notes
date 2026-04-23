# Eager Loading과 Lazy Loading

## 1. 개요

`Eager Loading`(즉시 로딩)과 `Lazy Loading`(지연 로딩)은 **연관된 데이터를 언제 조회할지 결정하는 방식**  

예를 들어 `Order` 엔티티가 `Member` 엔티티를 참조하고 있을 때,

- `Eager Loading`은 `Order`를 조회할 때 `Member`도 **즉시 함께 조회**합니다.
- `Lazy Loading`은 `Order`를 먼저 조회하고, `Member`가 **실제로 필요해지는 시점에 조회**합니다.

---

## 2. Eager Loading (즉시 로딩)

### 개념
주 엔티티를 조회할 때 연관된 엔티티도 **처음부터 같이 조회하는 방식**입니다.

### 특징
- 데이터를 조회하자마자 연관 객체도 사용할 수 있습니다.
- 추가 조회 없이 바로 접근 가능한 경우가 많습니다.
- 필요하지 않은 연관 데이터까지 미리 불러올 수 있습니다.

### 예시 상황
`Order`를 조회하는 순간 `Order`에 연결된 `Member`도 함께 조회됩니다.

### JPA 예시
```java
@Entity
public class Order {

    @ManyToOne(fetch = FetchType.EAGER)
    private Member member;
}
```

### 장점
- 연관 객체를 바로 사용할 수 있어 편리합니다.
- 코드가 직관적으로 보일 수 있습니다.

### 단점
- 필요 없는 데이터까지 조회해서 성능이 떨어질 수 있습니다.
- 연관관계가 많아질수록 쿼리가 무거워질 수 있습니다.
- 예상하지 못한 추가 쿼리나 비효율이 생길 수 있습니다.

---

## 3. Lazy Loading (지연 로딩)

### 개념
주 엔티티만 먼저 조회하고, 연관된 엔티티는 **실제로 접근하는 시점에 조회하는 방식**입니다.

### 특징
- 처음 조회는 가볍게 수행됩니다.
- 필요한 시점에만 연관 데이터를 조회합니다.
- 프록시(proxy) 객체를 사용해 나중에 실제 데이터를 불러오는 경우가 많습니다.

### 예시 상황
`Order`를 조회한 뒤에는 `Member`를 아직 조회하지 않고,
`order.getMember()`를 호출하는 순간 `Member`를 조회합니다.

### JPA 예시
```java
@Entity
public class Order {

    @ManyToOne(fetch = FetchType.LAZY)
    private Member member;
}
```

### 장점
- 초기 조회 성능이 좋아질 수 있습니다.
- 필요한 데이터만 조회해서 효율적입니다.

### 단점
- 연관 데이터를 사용할 때 추가 SQL이 실행됩니다.
- 잘못 사용하면 `N+1 문제`가 발생할 수 있습니다.
- 영속성 컨텍스트 밖에서 접근하면 `LazyInitializationException`이 발생할 수 있습니다.

---

## 4. Eager Loading과 Lazy Loading의 차이

| 구분 | Eager Loading | Lazy Loading |
|---|---|---|
| 조회 시점 | 처음 조회할 때 함께 조회 | 실제 접근할 때 조회 |
| 초기 성능 | 무거울 수 있음 | 상대적으로 가벼움 |
| 연관 객체 사용 | 바로 가능 | 접근 시 추가 조회 필요 |
| 불필요한 데이터 조회 | 발생 가능 | 비교적 적음 |
| 주의할 점 | 과도한 조회 | N+1, 지연 로딩 예외 |

---

## 5. 동작 예시

### Lazy Loading 예시 코드
```java
Order order = em.find(Order.class, 1L); // Order만 조회
System.out.println(order.getId());
System.out.println(order.getMember().getName()); // 이 시점에 Member 조회
```

### 예상 흐름
1. `Order` 조회 SQL 실행
2. `order.getId()` 호출 시 추가 조회 없음
3. `order.getMember()` 호출 시 `Member` 조회 SQL 실행
---

## 6. 주 사용 방식

실무에서는 보통 **기본값으로 Lazy Loading을 선호**

- 필요 없는 데이터 조회를 줄일 수 있습니다.
- 성능 문제를 더 세밀하게 제어할 수 있습니다.
- 특정 화면이나 API에서 필요한 연관 데이터만 선택적으로 조회할 수 있습니다.

다만 Lazy Loading만 무조건 좋은 것은 아닙니다.  
특정 상황에서는 연관 데이터를 항상 함께 사용하므로, 오히려 한 번에 조회하는 전략이 더 적절할 수도 있습니다.

실무에서는 보통 다음과 같이 접근합니다.

- 연관관계는 기본적으로 `LAZY`로 설정
- 정말 필요한 경우에만 `fetch join`이나 엔티티 그래프 등을 사용해 함께 조회

---

## 7. 주의해야 할 문제

### 1) N+1 문제
부모 엔티티 목록을 조회한 뒤, 각 엔티티의 연관 객체를 순차적으로 접근하면서 쿼리가 반복 실행되는 문제입니다.

예를 들어,
- `Order` 100개 조회 쿼리 1번
- 각 `Order`의 `Member` 조회 쿼리 100번

이렇게 총 101번의 쿼리가 실행될 수 있습니다.

### 2) LazyInitializationException
지연 로딩 객체에 접근하려고 했는데, 이미 영속성 컨텍스트가 종료된 경우 발생할 수 있습니다.

예를 들어,
- 트랜잭션 종료 후
- 컨트롤러나 뷰에서 지연 로딩 객체 접근

이때 예외가 발생할 수 있습니다.
