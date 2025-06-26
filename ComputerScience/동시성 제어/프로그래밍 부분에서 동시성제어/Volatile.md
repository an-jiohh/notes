
# **Volatile 기반 동시성 제어**

  


멀티스레드 환경에서 각 스레드는 성능 최적화를 위해 **자신만의 CPU 캐시(레지스터 포함)**에 변수 값을 복사해 사용함. 이로 인해 **메인 메모리의 최신 값과 불일치 현상(가시성 문제)**가 발생할 수 있음.

  

이를 해결하기 위한 최소한의 동기화 수단으로 **volatile 키워드**가 존재.

---

## **Volatile의 개념**

  

volatile은 **변수의 가시성(Visibility)을 보장**하는 키워드로, 선언된 변수에 대해:

- **모든 스레드가 메인 메모리의 최신 값을 직접 읽음**
    
- 변수 값 변경 시 즉시 메인 메모리에 반영 을 보장함.

---

## **Volatile의 동작 원리**

- 변수에 volatile을 선언하면:
    
    - 해당 변수는 **스레드 로컬 캐시에 저장되지 않음**
        
    - 항상 **메인 메모리로부터 값을 읽고**, 메인 메모리에 직접 값을 씀
        
    - 컴파일러나 CPU의 **명령어 재정렬(Instruction Reordering)** 일부 제한
        
    
- 단, **원자성(Atomicity)은 보장하지 않음**
    

  

### **플래그 변수**

```
public class Example {
    private volatile boolean running = true;

    public void stop() {
        running = false;
    }

    public void run() {
        while (running) {
            // 작업 수행
        }
    }
}
```

- volatile 없으면 running 값 변경이 다른 스레드에 반영되지 않아 무한 루프 발생 가능
    
- volatile 선언으로 최신 값 즉시 반영
    
## **Volatile로 부족한 경우**

  

다음과 같은 경우에는 volatile만으로 동시성 제어가 불가능하고 별도의 동기화가 필요:

- count++ 같은 복합 연산 → **synchronized**, AtomicInteger 사용
    
- 데이터 일관성 보장 필요 → 락 사용
    
- 임계영역 전체 보호 필요 → 락 사용

---

## 정리 

캐시메모리를 패스하고 메인메모리에서 무조건 값을 읽어오게끔

항상 최신화된 값을 읽어갈 수 있다.

메인 메모리를 보게하는 것은 좋은데

동시에 접근해서 동시에 저장할 경우 문제가 발생 할 수 있다.

→ write Thread가 1개가 없으면 완벽한 동시성 보장 어렵다.