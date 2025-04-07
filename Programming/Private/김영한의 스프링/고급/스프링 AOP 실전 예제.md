---
title: 
tags:
  - java
  - spring
  - aop
publish: true
date: 2025-01-04 16:15
comments: true
---
## 배경
- 배경 지식은 [[스프링 AOP 이론]], [[스프링 AOP 구현]], [[스프링 AOP 포인트컷 지시자]]를 참고하면 된다.

## 예시
- 학습한 내용들을 활용해서 다음과 같이 어노테이션 기반의 유용한 스프링 AOP를 만들어본다.
	- `@Trace`: 커스텀 어노테이션으로 로그 출력하기
	- `@Retry`: 커스텀 어노테이션으로 예외 발생 시 재시도 하기

#### 예제 프로젝트 구현
```java
@Repository  
public class ExamRepository {  
    private static int seq = 0;  
  
    /**  
     * 5번에 1번 실패하는 요청  
     */  
    public String save(String itemId) {  
        seq++;  
          
        if (seq % 5 == 0) {  
            throw new IllegalStateException("예외 발생");  
        }  
  
        return "ok";  
    }  
}
```
- `ExamRepository.save()`는 5번 요청하면 한 번 예외가 발생한다.

```java
@Service  
@RequiredArgsConstructor  
public class ExamService {  
    private final ExamRepository examRepository;  
  
    public void request(String itemId) {  
        examRepository.save(itemId);  
    }  
}
```
- `ExamService`는 단순한 구조의 서비스이다. 리포지토리의 `save()`를 호출한다.

```java
@SpringBootTest  
@Slf4j  
public class ExamTest {  
    @Autowired  
    ExamService examService;  
  
    @Test  
    void test() {  
        for (int i = 0; i < 5; i++) {  
            log.info("client request i = {}", i);  
            examService.request("data" + i);  
        }  
    }  
}
```
- 5번 요청하도록 테스트를 구현한다.
- 5번 요청 시점에 예외가 발생한다.

#### @Trace 구현
```java
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface Trace {  
}
```

```java
@Slf4j  
@Aspect  
public class TraceAspect {  
    @Before("@annotation(hello.aop.exam.annotation.Trace)")  
    public void doTrace(JoinPoint joinPoint) {  
        Object[] args = joinPoint.getArgs();  
        log.info("[trace] {} args = {}", joinPoint.getSignature(), args);  
    }  
}
```
- `ExamService.request()`와 `ExamRepository.save()`에 `@Trace`를 적용한다. 예시 코드는 생략한다.
- `@Aspect`를 스프링 빈으로 등록하면 [[스프링이 제공하는 빈 후처리기]]가 자동으로 어드바이저로 변환한다. 따라서 `TraceAspect`를 스프링 빈으로 등록 해야한다. 예시 코드는 생략한다.

```
2025-01-04T16:30:51.504+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.ExamTeest                 : client request i = 0
2025-01-04T16:30:51.508+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data0]
2025-01-04T16:30:51.508+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data0]
2025-01-04T16:30:51.508+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.ExamTeest                 : client request i = 1
2025-01-04T16:30:51.508+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data1]
2025-01-04T16:30:51.508+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data1]
2025-01-04T16:30:51.508+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.ExamTeest                 : client request i = 2
2025-01-04T16:30:51.508+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data2]
2025-01-04T16:30:51.509+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data2]
2025-01-04T16:30:51.509+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.ExamTeest                 : client request i = 3
2025-01-04T16:30:51.509+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data3]
2025-01-04T16:30:51.509+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data3]
2025-01-04T16:30:51.509+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.ExamTeest                 : client request i = 4
2025-01-04T16:30:51.509+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data4]
2025-01-04T16:30:51.509+09:00  INFO 5226 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data4]
```
- 결과를 살펴보면 의도한대로 함수의 시그니처와 전달된 파라미터가 잘 출력된다.
- 5번째 요청에서는 예외가 발생한다. 해당 예외 로그는 생략한다.

#### @Retry 구현
- 현재 로직은 5번째 요청에서 예외가 발생하도록 작성되어 있다.
- 예외가 발생하면 재시도하여 복구하도록 AOP를 구현한다.
- 이 어노테이션에는 재시도 횟수로 사용할 값이 있다. 기본 값으로 `3`을 사용한다.

```java
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface Retry {  
    int value() default 3;  
}
```

```java
@Slf4j  
@Aspect  
public class RetryAspect {  
    @Around("@annotation(retry)")  
    public Object doRetry(ProceedingJoinPoint joinPoint, Retry retry) throws Throwable {  
        log.info("[retry] {}, args = {}", joinPoint.getSignature(), retry);  
  
        int max = retry.value();  
        Exception exceptionHolder = null;  
  
        for (int count = 1; count <= max; count++) {  
            try {  
                log.info("[retry] count/max = {}/{}", count, max);  
                return joinPoint.proceed();  
            } catch (Exception e) {  
                exceptionHolder = e;  
            }  
        }  
  
        throw exceptionHolder;  
    }  
}
```
- `@annotation(retry)`
	- `@annotation` 지시자를 통해 파라미터에 정의된 `Retry`와 동일한 어노테이션이 적용된 대상을 포인트컷 매치 대상으로 한다.
- `int max`
	- 어노테이션의 속성 `value`를 가져와서 최대 재시도 횟수를 지정한다.
- `exceptionHolder`
	- 최대 재시도 횟수를 넘어가면 `exception`을 다시 던져야하므로 `exceptionHolder`를 통해 예외를 잠시 보관한다.
- 실행하면 다음과 같은 로그가 출력된다.

```java
2025-01-04T16:47:44.350+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.ExamTest                  : client request i = 0
2025-01-04T16:47:44.353+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data0]
2025-01-04T16:47:44.353+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data0]
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] String hello.aop.exam.ExamRepository.save(String), args = @hello.aop.exam.annotation.Retry(4)
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] count/max = 1/4
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.ExamTest                  : client request i = 1
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data1]
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data1]
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] String hello.aop.exam.ExamRepository.save(String), args = @hello.aop.exam.annotation.Retry(4)
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] count/max = 1/4
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.ExamTest                  : client request i = 2
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data2]
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data2]
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] String hello.aop.exam.ExamRepository.save(String), args = @hello.aop.exam.annotation.Retry(4)
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] count/max = 1/4
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.ExamTest                  : client request i = 3
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data3]
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data3]
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] String hello.aop.exam.ExamRepository.save(String), args = @hello.aop.exam.annotation.Retry(4)
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] count/max = 1/4
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.ExamTest                  : client request i = 4
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] void hello.aop.exam.ExamService.request(String) args = [data4]
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.TraceAspect           : [trace] String hello.aop.exam.ExamRepository.save(String) args = [data4]
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] String hello.aop.exam.ExamRepository.save(String), args = @hello.aop.exam.annotation.Retry(4)
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] count/max = 1/4
2025-01-04T16:47:44.354+09:00  INFO 5349 --- [aop] [           main] hello.aop.exam.aop.RetryAspect           : [retry] count/max = 2/4
```
- `TryAspect`와 `RetryAspect`가 동시에 적용되어 있어서 로그가 길다.
- 핵심은 로그 마지막 줄의 `count/max` 로그인데, 2번 시도해서 복구가 되었다는 의미다.
---

References: 김영한의 스프링 핵심 원리 - 고급편

Links to this page: [[스프링 AOP 이론]], [[스프링 AOP 구현]], [[스프링 AOP 포인트컷 지시자]], [[스프링이 제공하는 빈 후처리기]]
