
# **Atomic Class**

  

멀티스레드 환경에서 synchronized를 사용하지 않고도 **간단한 변수에 대해 안전하게 동시성 제어를 수행**하기 위해 java.util.concurrent.atomic 패키지가 제공됨.

-> 일단적인 변수가 아닌 동시성제어가 가능한 변수를 제공하여 해당 변수를 사용하게끔함
  

Atomic Class는 **원자성(Atomicity)**을 보장하여 데이터 경쟁 없이 변수의 안전한 수정이 가능하도록 설계됨.

---

## **2. Atomic Class의 개념**

  

Atomic Class는 내부적으로 **CAS(Compare-And-Swap)** 기반으로 동작하며,

다음 두 가지를 보장:

- **원자성 (Atomicity):** 더하기, 빼기, 값 변경 등 연산을 중간에 끊기지 않고 한 번에 수행
    
- **가시성 (Visibility):** 변수 값 변경이 모든 스레드에 즉시 반영됨 (내부적으로 volatile 사용)

**CAS(Compare-And-Swap) 동작 방식**
1. 
2. 


따라서 간단한 연산에 대해 synchronized 없이도 안전하게 동시성 제어 가능

---

## **3. 주요 Atomic Class 종류**

|**클래스**|**설명**|
|---|---|
|AtomicInteger|int형 변수의 원자 연산 지원|
|AtomicLong|long형 변수의 원자 연산 지원|
|AtomicBoolean|boolean형 변수의 원자 연산 지원|
|AtomicReference<T>|참조형 변수(T)의 원자 연산 지원|
|AtomicIntegerArray|배열 요소 각각에 대해 원자 연산 지원|

---

## **4. 대표 사용 예시**

  

### **4.1. AtomicInteger**

```
private final AtomicInteger count = new AtomicInteger(0);

public void increment() {
    count.incrementAndGet();  // 원자적으로 +1 수행
}

public int getCount() {
    return count.get();
}
```

### **4.2. AtomicBoolean**

```
private final AtomicBoolean flag = new AtomicBoolean(false);

public void enable() {
    flag.set(true);
}

public boolean isEnabled() {
    return flag.get();
}
```

### **4.3. AtomicReference**

```
private final AtomicReference<String> ref = new AtomicReference<>("초기값");

public void update() {
    ref.set("변경값");
}

public String getValue() {
    return ref.get();
}
```

---

## **5. 내부 동작 원리 (CAS)**

  

**CAS(Compare-And-Swap)**를 활용하여 다음과 같은 방식으로 원자성을 보장:

1. 변수의 현재 값을 읽음
    
2. 예상한 값(기존 값)과 현재 값이 일치하는지 비교
    
3. 일치하면 새 값으로 교체
    
4. 실패하면 다시 시도
    

  

이 과정을 반복하면서 락 없이도 동시성 제어를 수행

---

## **6. 실무 사용 시 주의사항**

- 단순한 카운터, 플래그, 참조 값 교체에는 매우 유용
    
- 복합 상태(여러 변수 조합)에는 부적합 → 이때는 synchronized 또는 명시적 Lock 필요
    
- 경쟁이 매우 심한 환경에서는 CAS도 반복적으로 실패해 오히려 성능 저하 가능
    

---

## **7. Atomic Class와 Volatile의 차이**

|**항목**|volatile|**Atomic Class**|
|---|---|---|
|원자성|보장 안 됨|보장함 (CAS 기반)|
|가시성|보장됨|보장됨|
|복합 연산 지원|불가능|가능 (increment, compareAndSet 등 지원)|
|사용 용도|단순 상태 플래그 전파|단순 변수의 안전한 연산|

---

## **8. 결론**

  

Atomic Class는:

- 간단한 변수에 대해 synchronized 없이 안전하게 동시성 제어를 제공
    
- 내부적으로 CAS를 활용하여 락 없는 원자 연산을 지원
    
- 복합 연산이나 여러 상태를 다루는 경우에는 여전히 락이 필요
    
- 상황에 따라 synchronized, Lock, Atomic Class를 적절히 선택하는 것이 실무 핵심
    