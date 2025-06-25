# **ExecutorService**
  

멀티스레드 프로그래밍에서 개발자가 자주 겪는 문제 중 하나는 **스레드 직접 생성과 관리의 복잡성**이다. Java에서 new Thread()로 스레드를 직접 생성하는 방식은 간단하지만, 대규모 작업 처리나 고성능 시스템에서는 다음과 같은 실질적인 문제가 발생한다.

- 스레드가 과도하게 생성되면 시스템 자원 고갈
    
- 스레드 생성과 소멸의 반복으로 인한 성능 저하
    
- 스레드 개수 제어 및 종료 관리의 어려움
    
이러한 문제를 해결하기 위해 **스레드풀 기반의 작업 실행 관리**가 필요해졌으며, 이를 위해 Java에서는 ExecutorService라는 표준 API를 제공한다.

## **ExecutorService 개념**

  

ExecutorService는 Java에서 멀티스레드 작업을 보다 효율적이고 안정적으로 관리하기 위해 제공하는 **스레드풀 기반의 작업 실행 프레임워크**이다. 개발자는 스레드 생성 및 종료를 직접 관리할 필요 없이, ExecutorService를 통해 작업을 제출하고 실행 결과를 제어할 수 있다.

  

대표적인 구현체는 다음과 같다.

|**메서드**|**설명**|
|---|---|
|Executors.newFixedThreadPool(n)|고정 크기의 스레드풀 생성|
|Executors.newCachedThreadPool()|필요에 따라 스레드 생성 후 재활용|
|Executors.newSingleThreadExecutor()|단일 스레드로 순차 실행|
|Executors.newScheduledThreadPool(n)|예약 작업 또는 주기적 실행 지원|

  

아래는 Executors.newFixedThreadPool()을 사용해 5개의 스레드로 10개의 작업을 처리하는 예제이다.

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExecutorServiceExample {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(5);

        for (int i = 0; i < 10; i++) {
            int taskNumber = i;
            executorService.submit(() -> {
                System.out.println("작업 " + taskNumber + "을 처리하는 스레드: " + Thread.currentThread().getName());
                try {
                    Thread.sleep(1000); // 작업 처리 시간 시뮬레이션
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        executorService.shutdown();
    }
}
```

### **Thread 직접 사용하는 경우**

가장 기본적인 멀티스레드 방식은 new Thread()를 사용하는 방식이다. 이 방법은 소규모 테스트나 간단한 작업에서는 유용하지만, 실무 시스템에서는 여러 가지 한계가 있다.

```
public class BasicThreadExample {

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            int taskNumber = i;
            Thread thread = new Thread(() -> {
                System.out.println("작업 " + taskNumber + "을 처리하는 스레드: " + Thread.currentThread().getName());
                try {
                    Thread.sleep(1000); // 작업 처리 시간 시뮬레이션
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
            thread.start();
        }
    }
}
```

**Thread 직접 생성의 문제점**

- 작업마다 새로운 스레드가 생성됨
    
- 스레드 생성 및 소멸 비용이 누적됨
    
- 스레드 개수 통제가 어려워 자원 고갈 가능성 존재
    
- 대규모 작업에 비효율적
    
- 스레드 종료나 예외 관리가 복잡함
    



실제 시스템에서 작업량이 많거나, 동시에 많은 요청을 처리해야 하는 상황에서는 이러한 방식이 비효율적이고 불안정하다.

---

## **ExecutorService의 주요 메서드**

  

ExecutorService는 다양한 작업 제출 및 종료 제어 기능을 제공한다. 주요 메서드는 다음과 같다.

|**메서드**|**설명**|
|---|---|
|submit(Runnable task)|Runnable 작업을 제출, Future<?> 반환|
|submit(Callable<T> task)|Callable 작업을 제출, Future<T> 반환|
|invokeAll(Collection tasks)|여러 작업을 모두 실행 후 완료될 때까지 대기, List<Future> 반환|
|invokeAny(Collection tasks)|여러 작업 중 하나가 완료되면 바로 반환|
|shutdown()|더 이상 새로운 작업을 받지 않음, 기존 작업은 모두 마무리|
|shutdownNow()|실행 중인 작업을 중단 시도, 큐에 남은 작업도 제거|
|isShutdown()|shutdown() 호출 여부 확인|
|isTerminated()|모든 작업 종료 여부 확인|
|awaitTermination(timeout, unit)|지정 시간 동안 종료 대기, true/false 반환|

### **submit과 Callable 예시**

```
import java.util.concurrent.*;

public class CallableExample {

    public static void main(String[] args) throws Exception {
        ExecutorService executorService = Executors.newFixedThreadPool(3);

        Callable<String> task = () -> {
            Thread.sleep(1000);
            return "작업 완료";
        };

        Future<String> future = executorService.submit(task);

        System.out.println("결과 대기 중...");
        String result = future.get(); // 결과가 올 때까지 블로킹
        System.out.println("결과: " + result);

        executorService.shutdown();
    }
}
```

---

## **ExecutorService의 안정성 원리**

  

### **1. 스레드 수 제한을 통한 자원 통제**

  

ExecutorService는 내부적으로 **스레드풀(Thread Pool)**을 관리한다. 스레드풀을 통해 최대 스레드 수를 명확히 제한할 수 있으며, 이를 통해 다음과 같은 안정성이 확보된다.

- 스레드 폭증 방지
    
- 시스템 메모리 및 CPU 사용량 예측 가능
    
- 과도한 스레드 생성으로 인한 성능 저하 최소화
    

  

### **2. 스레드 재사용을 통한 비용 절감**

  

스레드풀 내부의 스레드는 한 번 생성되면 재사용된다. 작업이 끝나면 스레드가 소멸하는 것이 아니라 풀에 대기 상태로 남아, 다음 작업을 처리한다.

  

### **3. 작업 큐를 통한 흐름 제어**

  

ExecutorService는 내부적으로 **BlockingQueue**를 사용해 작업을 관리한다. 스레드풀이 바쁠 경우, 새로 들어오는 작업은 큐에 쌓이고, 스레드가 여유로워지면 순차적으로 처리된다.

  

### **4. 종료 및 예외 관리의 체계화**

  

ExecutorService는 shutdown(), shutdownNow() 등의 메서드를 통해 스레드풀 종료를 체계적으로 관리한다. 이를 통해 리소스 누수나 비정상 종료를 방지할 수 있다.

  

### **5. JVM 차원의 최적화 기반**

  

ExecutorService는 JVM의 스레드 최적화 기능을 활용해, 코어 수에 맞는 병렬 처리 및 IO 효율 분배를 지원한다.

  

## **실무 관점에서의 적용 예시**

  

대규모 트래픽 처리, 배치 작업, 비동기 요청 처리 등에서 사용될 수  있다.

  

### **실무 코드 예시**

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.CountDownLatch;

public class ExecutorServiceRealExample {

    public static void main(String[] args) throws InterruptedException {
        int threadCount = 100;
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        CountDownLatch latch = new CountDownLatch(threadCount);

        for (int i = 0; i < threadCount; i++) {
            executorService.submit(() -> {
                try {
                    System.out.println("작업 처리 스레드: " + Thread.currentThread().getName());
                } finally {
                    latch.countDown();
                }
            });
        }

        latch.await();
        executorService.shutdown();
        System.out.println("모든 작업 완료");
    }
}
```