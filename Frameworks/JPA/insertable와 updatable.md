
# `insertable = false`, `updatable = false` 정리


JPA에서 `@Column`의 `insertable`, `updatable` 옵션은 해당 필드를 **INSERT / UPDATE SQL에 포함할지 여부**를 결정한다.

```java
@Column(name = "create_at", insertable = false, updatable = false)
private LocalDateTime createAt;
````

위 설정은 다음 의미를 가진다.

- `insertable = false`  
    → INSERT SQL에 해당 컬럼을 포함하지 않음
- `updatable = false`  
    → UPDATE SQL에 해당 컬럼을 포함하지 않음

즉, 해당 컬럼은 **Java/JPA가 직접 값을 넣거나 수정하지 않고, DB가 관리하도록 맡긴다**는 의미이다.

---

## 2. insertable = false

`insertable = false`는 JPA가 INSERT 쿼리를 만들 때 해당 컬럼을 제외하라는 뜻이다.

예를 들어 엔티티를 저장할 때:

```java
Room room = Room.builder()
        .name("test-room")
        .build();

roomRepository.save(room);
```

`createAt`, `updatedAt` 값을 자바에서 직접 넣지 않았다면 값은 `null`일 수 있다.

  

만약 `insertable = false`가 없다면 JPA가 다음과 같은 SQL을 만들 수 있다.

```sql
INSERT INTO room (name, create_at, updated_at)
VALUES ('test-room', NULL, NULL);
```

이 경우 DB 입장에서는 `create_at`, `updated_at`에 `NULL`을 직접 넣으라는 뜻으로 해석할 수 있다.

  

그러면 DB에 설정된 기본값이 기대대로 동작하지 않을 수 있다.

```sql
create_at DATETIME DEFAULT CURRENT_TIMESTAMP
```

`DEFAULT CURRENT_TIMESTAMP`는 보통 해당 컬럼이 INSERT 문에서 **생략되었을 때** 동작한다.

  

그래서 `insertable = false`를 사용하면 JPA는 다음처럼 SQL을 만든다.

```sql
INSERT INTO room (name)
VALUES ('test-room');
```

이렇게 되면 DB가 자동으로 현재 시간을 넣는다.

```text
create_at  → DEFAULT CURRENT_TIMESTAMP 작동
updated_at → DEFAULT CURRENT_TIMESTAMP 작동
```

정리하면:

```text
insertable = false
→ JPA가 INSERT SQL에서 해당 컬럼을 제외함
→ Java가 null을 직접 넣지 않음
→ DB의 DEFAULT CURRENT_TIMESTAMP가 작동할 수 있음
```

---

## 3.**updatable = false

`updatable = false`는 JPA가 UPDATE 쿼리를 만들 때 해당 컬럼을 제외하라는 뜻이다.

예를 들어 방 이름을 수정한다고 하면:

```java
room.changeName("new-room");
```

DB 컬럼이 다음과 같이 설정되어 있을 수 있다.

```sql
updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
```

이 설정은 해당 행의 다른 컬럼이 수정될 때 `updated_at`을 자동으로 현재 시간으로 변경한다.

  

그런데 JPA가 `updated_at` 컬럼까지 UPDATE SQL에 포함하면 문제가 생길 수 있다.

```sql
UPDATE room
SET name = 'new-room',
    updated_at = '2026-04-30 02:17:45'
WHERE id = 1;
```

이 경우 DB가 자동으로 `updated_at`을 바꾸는 것이 아니라, JPA가 가진 값을 직접 넣게 된다.

  

반대로 `updatable = false`를 사용하면 JPA는 다음처럼 SQL을 만든다.

```sql
UPDATE room
SET name = 'new-room'
WHERE id = 1;
```

그러면 DB의 `ON UPDATE CURRENT_TIMESTAMP`가 작동해서 `updated_at`이 자동으로 현재 시간으로 변경된다.

  

정리하면:

```text
updatable = false
→ JPA가 UPDATE SQL에서 해당 컬럼을 제외함
→ Java/JPA가 수정 시간을 직접 덮어쓰지 않음
→ DB의 ON UPDATE CURRENT_TIMESTAMP가 작동할 수 있음
```

---

## 3. 읽기와 쓰기 관점에서 정리

`insertable = false`, `updatable = false`를 사용해도 값을 조회하는 것은 가능하다.

```java
room.getCreateAt();
room.getUpdatedAt();
```

다만 JPA가 INSERT나 UPDATE할 때 해당 컬럼을 SQL에 포함하지 않을 뿐이다.

|**구분**|**동작**|
|---|---|
|조회|가능|
|INSERT|SQL에 포함하지 않음|
|UPDATE|SQL에 포함하지 않음|
|값 관리 주체|DB|
|주 사용 목적|생성일, 수정일을 DB 자동 기능에 맡기기|

---

## **최종 정리**

```java
@Column(insertable = false, updatable = false)
```

는 다음 의미이다.

```text
이 컬럼은 JPA가 직접 INSERT/UPDATE하지 말고,
DB의 DEFAULT 또는 ON UPDATE 규칙에 맡긴다.
```

따라서 다음과 같은 DB 컬럼과 함께 자주 사용된다.

```sql
create_at DATETIME DEFAULT CURRENT_TIMESTAMP,
updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
```

핵심은 다음과 같다.

```text
insertable = false
→ INSERT 시 컬럼을 제외해서 DB 기본값이 작동하게 함

updatable = false
→ UPDATE 시 컬럼을 제외해서 DB 자동 수정 시간이 작동하게 함
```