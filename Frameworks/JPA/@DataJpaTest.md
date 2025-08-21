# **@DataJpaTest**

  

@DataJpaTest는 Spring Boot에서 **JPA 관련 빈만 슬라이스로 로드**하여 Repository/Entity 중심의 **가벼운 단위 테스트**를 수행하기 위한 어노테이션

- Entity, Repository, JPA 설정만 로딩 → 빠름
    
- 각 테스트는 트랜잭션 안에서 실행되고 **자동 롤백**
    
- 기본은 **내장 메모리 DB(H2/Derby)** 사용(교체 가능)
    

---

## **언제 쓰나**

- Repository 메서드 쿼리 동작 검증
    
- Entity 매핑, 연관관계, 제약조건(UNIQUE/NOT NULL) 검증
    
- JPQL/Query Method/Specification 동작 확인
    
- N+1 여부 감지(간단 범위)
    

서비스/컨트롤러/시큐리티 등은 로드되지 않습니다. 필요한 컴포넌트는 @Import 또는 @MockBean으로 주입합니다.

---

## **기본 동작 요약**

- 슬라이스 로딩: JPA 컴포넌트, TestEntityManager(JPA 제공 시)
    
- 트랜잭션: 각 테스트 자동 롤백
    
- 데이터베이스: 내장 DB 자동 구성(미지정 시)
    
- SQL 스크립트: @Sql로 사전/사후 데이터 준비 가능
    

---

## **사용 예제**


### **간단 Repository 테스트**

```
// src/test/java/com/example/user/UserRepositoryTest.java
package com.example.user;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void 사용자_저장_조회() {
        User user = new User("jiho", "password123");
        userRepository.save(user);

        Optional<User> loaded = userRepository.findByUsername("jiho");

        assertThat(loaded).isPresent();
        assertThat(loaded.get().getUsername()).isEqualTo("jiho");
    }
}
```

### **실제 DB(Testcontainers 또는 로컬 DB) 사용**

```
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) // 내장 DB로 교체 금지
class UserRepositoryRealDbTest {
    // DataSource는 application-test.yml 또는 Testcontainers로 구성
}
```

### **필요 Bean 주입**

```
@DataJpaTest
@Import({JpaConverters.class}) // 테스트에 필요한 추가 구성
class PostRepositoryTest { /* ... */ }
```

---

## **@SpringBootTest**

## **와 비교**

|**항목**|**@DataJpaTest**|**@SpringBootTest**|
|---|---|---|
|로딩 범위|JPA 슬라이스(Entity/Repository/JPA 설정)|애플리케이션 전체|
|속도|빠름|상대적으로 느림|
|트랜잭션|테스트마다 자동 롤백|보장되지 않음(설정 필요)|
|목적|Repository/Entity 단위 검증|통합/E2E 검증|

---

## **자주 쓰는 테스트 패턴**

- **테스트 데이터 준비**: @Sql, TestEntityManager, Fixture 메서드
    
- **제약 조건 검증**: UNIQUE/NOT NULL 위반 시 예외(assertThrows)
    
- **페이지/정렬**: PageRequest 조합으로 결과 보장 검증
    
- **성능 힌트**: @EntityGraph/fetch join 결과 범위 확인
    
- **마이그레이션 검증**: Flyway/Liquibase를 테스트 DB에도 적용
    

---

## **주의사항**

- 슬라이스 테스트이므로 **서비스/컨트롤러 빈 없음**
    
- 내장 DB와 운영 DB 방언 차이로 **쿼리 동작이 달라질 수 있음** → 필요 시 실제 DB/Testcontainers로 검증
    
- 영속성 컨텍스트 캐시로 인해 **즉시 조회값**이 DB 상태와 다를 수 있음 → flush()/clear() 적절히 사용
    
- 벌크 업데이트/네이티브 쿼리 후 1차 캐시 동기화 필요