# **CountDownLatch**

  

멀티스레드 프로그래밍에서는 특정 스레드들이 **모두 작업을 완료한 뒤에 다음 로직을 진행해야 하는 상황**이 자주 발생한다. 예를 들어:

- 여러 작업을 병렬로 실행하고, 모두 끝난 뒤 최종 결과를 출력해야 하는 경우
    
- 서버 초기화 시 여러 컴포넌트를 병렬로 준비하고, 모두 준비될 때까지 대기해야 하는 경우
    

  

이때, 스레드의 작업 완료 여부를 일일이 체크하거나, join()을 일일이 호출하는 것은 번거롭고 비효율적이다.

  

이런 문제를 해결하기 위해 Java에서 **CountDownLatch**가 제공된다.

  

## **CountDownLatch 개념**

  

**CountDownLatch**는 지정된 개수만큼 카운트가 감소할 때까지 스레드가 대기하도록 만드는 동기화 도구이다.

  

### **특징 요약**

- 초기 카운트를 설정
    
- 다른 스레드들이 작업을 마칠 때마다 countDown() 호출
    
- 카운트가 0이 되면 await()로 대기 중인 스레드가 모두 깨어남
    
- 일회성 도구로, 카운트가 0이 된 뒤 재사용 불가
    

---

## **CountDownLatch 기본 사용법**

```
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {

    public static void main(String[] args) throws InterruptedException {
        int workerCount = 3;
        CountDownLatch latch = new CountDownLatch(workerCount);

        for (int i = 0; i < workerCount; i++) {
            new Thread(() -> {
                System.out.println("작업 수행 중: " + Thread.currentThread().getName());
                try {
                    Thread.sleep(1000); // 작업 시뮬레이션
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                latch.countDown();
                System.out.println("작업 완료: " + Thread.currentThread().getName());
            }).start();
        }

        System.out.println("메인 스레드 대기 중");
        latch.await();
        System.out.println("모든 작업 완료, 메인 스레드 진행");
    }
}
```

---

## **주요 메서드**

|**메서드**|**설명**|
|---|---|
|CountDownLatch(int count)|카운트 초기값 설정|
|countDown()|카운트 1 감소, 카운트가 0이 되면 대기 중인 스레드 깨움|
|await()|카운트가 0이 될 때까지 현재 스레드 대기|
|await(long time, TimeUnit unit)|지정 시간까지 대기, 시간 초과 시 false 반환|

---

## **실무 활용 예시**

  

### **1. 병렬 작업 동기화**

```
ExecutorService executorService = Executors.newFixedThreadPool(5);
CountDownLatch latch = new CountDownLatch(10);

for (int i = 0; i < 10; i++) {
    executorService.submit(() -> {
        try {
            // 비즈니스 로직
        } finally {
            latch.countDown();
        }
    });
}

latch.await();
executorService.shutdown();
System.out.println("모든 비즈니스 로직 처리 완료");
```

### **2. 테스트 환경에서 동시성 재현**

  

서로 다른 스레드들이 특정 지점에서 동시에 출발해야 하는 경우에도 사용 가능하다. 이 경우는 CyclicBarrier가 더 적합하지만, 간단한 형태로 CountDownLatch 두 개를 조합해도 비슷한 효과를 낼 수 있다.

---

## **CountDownLatch와 다른 도구 비교**

|**동기화 도구**|**특징**|
|---|---|
|CountDownLatch|카운트 기반 대기, 일회성|
|CyclicBarrier|지정된 수의 스레드가 모두 도달하면 동시에 실행 재개|
|Semaphore|제한된 개수의 동시 접근 제어|

---

## **주의사항**

- **재사용 불가**: 카운트가 0이 되면 해당 CountDownLatch는 다시 사용할 수 없다. 새로 생성해야 한다.
    
- **deadlock 주의**: countDown() 호출 누락 시 영원히 대기 상태에 빠질 수 있다.
    