
# ApplicationRunner 



- **스프링 애플리케이션이 실행된 직후** 특정 로직을 실행할 수 있도록 제공되는 인터페이스.
- **기동 직후 한 번** 실행해야 하는 초기화 로직에서 사용   
	(초기 데이터 입력, 캐시 워밍업, 외부 연동 점검, 파라미터 파싱 등)


### **인터페이스 정의**

```
@FunctionalInterface
public interface ApplicationRunner {
    void run(ApplicationArguments args) throws Exception;
}
```

### **실행 시점**

- SpringApplication.run(...)이 완료된 직후 실행됨.
    
- 즉, **스프링 컨텍스트 초기화 → 빈 로딩 완료 → ApplicationRunner 실행** 순서.
    

  

### **예제**

```
@Component
public class MyAppRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("애플리케이션 실행 직후 동작");

        if (args.containsOption("user")) {
            System.out.println("User: " + args.getOptionValues("user"));
        }

        args.getNonOptionArgs().forEach(System.out::println);
    }
}
```

@Component 은 Spring Boot에서 Runner로 인식시키기 위해 선언

실행:

```
java -jar myapp.jar --user=jiho test1 test2
```

출력:

```
애플리케이션 실행 직후 동작
User: [jiho]
test1
test2
```

---

## **CommandLineRunner 와 비교**


- ApplicationRunner와 유사하지만, 인자를 단순히 **문자열 배열(String[])**로 받음.
- 간단히 실행 인자만 쓰고 싶을 때 적합.

  

### **인터페이스 정의**

```
@FunctionalInterface
public interface CommandLineRunner {
    void run(String... args) throws Exception;
}
```

### **예제**

```
@Component
public class MyCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner 실행");
        for (String arg : args) {
            System.out.println(arg);
        }
    }
}
```

실행:

```
java -jar myapp.jar test1 test2
```

출력:

```
CommandLineRunner 실행
test1
test2
```

---

## **ApplicationRunner vs CommandLineRunner 비교표**

|**항목**|**ApplicationRunner**|**CommandLineRunner**|
|---|---|---|
|인자 타입|ApplicationArguments|String[]|
|옵션/비옵션 구분|가능 (containsOption, getNonOptionArgs)|불가능|
|단순 사용성|다소 복잡|간단|
|실행 시점|동일 (컨텍스트 초기화 직후)|동일|
|주요 활용|초기 데이터 세팅, 외부 파라미터 파싱|간단한 실행 인자 처리|
