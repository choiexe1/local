---
title: 
tags:
  - java
  - spring
  - aop
publish: true
date: 2025-01-05 10:05
comments: true
---
## 배경
- 스프링은 프록시 방식의 AOP를 사용한다.
- 따라서 AOP를 적용하려면 항상 프록시를 통해 대상 객체(Target)를 호출해야 한다.
- 이 내용은 [[스프링 트랜잭션 이해#트랜잭션 AOP 주의 사항 - 프록시 내부 호출 1|트랜잭션 AOP 주의사항 - 프록시 내부 호출]]에서 살펴본 적이 있다.

## 프록시와 내부 호출 문제
- AOP를 적용하면 스프링은 대상 객체 대신에 프록시를 스프링 빈으로 등록한다.
- 따라서 스프링은 의존관계 주입 시 항상 프록시 객체를 주입한다. 프록시 객체가 주입 된다.
- 따라서 대상 객체(Target)를 직접 호출하는 문제는 일반적으로 발생하지 않는다.
- 그런데 대상 객체의 내부에서 메서드 호출이 발생하면 **프록시를 거치지 않고 대상 객체를 직접 호출하는 문제**가 발생한다.

## 프록시와 내부 호출 문제 - 예시
```java
@Slf4j
@Component
public class CallServiceV0 {  
    public void external() {  
        log.info("call external");  
        internal(); // 내부 this.internal() 호출  
    }  
  
    public void internal() {  
        log.info("call internal");  
    }  
}
```
- 자바에서 메서드를 호출 할 때 대상을 지정하지 않으면 앞에 자기 자신의 인스턴스를 뜻하는 `this`가 붙게된다.
	- `internal() -> this.internal()`

```java
@Aspect  
@Slf4j  
public class CallLogAspect {  
    @Before("execution(* hello.aop.internalcall..*.*(..))")  
    public void doLog(JoinPoint joinPoint) {  
        log.info("aop = {}", joinPoint.getSignature());  
    }  
}
```
- `CallServiceV0`에 적용되는 애스펙트이다.

```java
@SpringBootTest  
@Import(CallLogAspect.class)  
@Slf4j  
class CallServiceV0Test {  
    @Autowired  
    CallServiceV0 callServiceV0;  
  
    @Test  
    void external() {  
        callServiceV0.external();  
    }  
  
    @Test  
    void internal() {  
        callServiceV0.internal();  
    }  
}
```

```
2025-01-05T10:23:02.388+09:00  INFO 3933 --- [aop] [           main] h.aop.internalcall.aop.CallLogAspect     : aop = void hello.aop.internalcall.CallServiceV0.external()
2025-01-05T10:23:02.389+09:00  INFO 3933 --- [aop] [           main] hello.aop.internalcall.CallServiceV0     : call external
2025-01-05T10:23:02.389+09:00  INFO 3933 --- [aop] [           main] hello.aop.internalcall.CallServiceV0     : call internal
2025-01-05T10:23:02.396+09:00  INFO 3933 --- [aop] [           main] h.aop.internalcall.aop.CallLogAspect     : aop = void hello.aop.internalcall.CallServiceV0.internal()
2025-01-05T10:23:02.396+09:00  INFO 3933 --- [aop] [           main] hello.aop.internalcall.CallServiceV0     : call internal
```
- 실행 결과를 살펴보면 `external()` 메서드 호출 시 내부의 `internal()` 메서드 호출 부분엔 AOP가 적용되지 않고 있다.
- 이는 프록시를 거쳐서 호출된 것이 아니라 단순히 대상 객체(Target)가 자기 자신의 메서드를 호출한 것이기 때문이다.
	- `internal()`을 호출하는 케이스는 프록시를 거치기 때문에 AOP가 적용되는 것을 확인할 수 있다.

## 프록시와 내부 호출 문제 - 대안
- **내부 호출 문제를 해결하는 방법들**
	1. 자기 자신 주입
	2. 지연 조회
	3. 구조 변경

#### 자기 자신 주입

^9d1e99

```java
@Slf4j  
@Component  
public class CallServiceV1 {  
    private CallServiceV1 callServiceV1;  
  
    @Autowired  
    public void setCallServiceV1(CallServiceV1 callServiceV1) {  
        this.callServiceV1 = callServiceV1;  
    }  
  
    public void external() {  
        log.info("call external");  
        callServiceV1.internal();  
    }  
  
    public void internal() {  
        log.info("call internal");  
    }  
}
```

```
2025-01-05T11:00:54.485+09:00  INFO 4375 --- [aop] [           main] h.aop.internalcall.aop.CallLogAspect     : aop = void hello.aop.internalcall.CallServiceV1.external()
2025-01-05T11:00:54.486+09:00  INFO 4375 --- [aop] [           main] hello.aop.internalcall.CallServiceV1     : call external
2025-01-05T11:00:54.486+09:00  INFO 4375 --- [aop] [           main] h.aop.internalcall.aop.CallLogAspect     : aop = void hello.aop.internalcall.CallServiceV1.internal()
2025-01-05T11:00:54.486+09:00  INFO 4375 --- [aop] [           main] hello.aop.internalcall.CallServiceV1     : call internal
```
- `CallServiceV1`은 Aspect의 포인트컷에 의해 AOP 적용 대상이다. 따라서 프록시 객체가 스프링 빈으로 등록된다.
- 그런데 자기 자신을 생성자로 주입 받으려고 하면 당연히 오류가 발생한다.
- 따라서 수정자(Setter)를 통해 `CallServiceV1`을 의존 관계 주입 받게 되면 프록시 객체가 주입된다.
- 프록시 객체인 `callServiceV1.internal()`을 호출해서 프록시를 통해 호출하는 방식이다.

> 참고로 스프링 부트 2.6 버전 이상부터는 이런 순환 참조를 기본적으로 금지한다.
> 
> application.properties에 spring.main.allow-circular-references=true 옵션을 설정해야 한다.


#### 지연 조회
- 앞서 [[#^9d1e99|자기 자신 주입]] 방법에서 생성자 주입에 실패하는 이유는 자기 자신을 생성하면서 동시에 주입해야 하기 때문이다.
- 지연 조회 방법은 스프링 빈을 지연해서 조회하는 방법이다.
- 두 가지 방법이 존재한다.
	- [[스프링 컨테이너와 스프링 빈|스프링 컨테이너]]를 직접 주입 받아서 빈을 획득하고 사용하는 단순한 방법
	- [[빈 스코프#ObjectFactory, ObjectProvider|오브젝트 프로바이더]]를 통한 DL(Dependency Lookup)

```java title="스프링 컨테이너 직접 주입"
@Slf4j  
@Component  
public class CallServiceV2 {  
    private final ApplicationContext applicationContext;  
  
    public CallServiceV2(ApplicationContext applicationContext) {  
        this.applicationContext = applicationContext;  
    }  
  
    public void external() {  
        log.info("call external");  
        CallServiceV2 callServiceV2 = applicationContext.getBean(CallServiceV2.class);  
        callServiceV2.internal();  
    }  
  
    public void internal() {  
        log.info("call internal");  
    }  
}
```
- 스프링 컨테이너를 직접 주입받아 사용한 예시이다.
- 그런데 스프링 컨테이너는 단순히 지연 조회만을 위해서 사용하기에는 기능이 너무 많다.
- 따라서 오브젝트 프로바이더를 사용하는 것을 권장한다.


```java title="오브젝트 프로바이더를 통한 DL"
@Slf4j  
@Component  
public class CallServiceV2 {  
    private final ObjectProvider<CallServiceV2> callServiceProvider;  
  
    public CallServiceV2(ObjectProvider<CallServiceV2> callServiceProvider) {  
        this.callServiceProvider = callServiceProvider;  
    }  
  
    public void external() {  
        log.info("call external");  
        CallServiceV2 callServiceV2 = callServiceProvider.getObject();  
        callServiceV2.internal();  
    }  
  
    public void internal() {  
        log.info("call internal");  
    }  
}
```
- 오브젝트 프로바이더를 통한 예시이다.
- 오브젝트 프로바이더는 DL에 특화된 기능이다.

#### 구조 변경
- 앞선 방법들은 자기 자신을 주입하거나 제공자(Provider)를 사용해야 하는 것 처럼 조금 어색한 모습을 만들었다.
- 가장 나은 대안은 내부 호출이 발생하지 않도록 구조 자체를 변경하는 것이다.
- 스프링에서도 이 방법을 가장 권장한다.

```java title="InternalService"
@Slf4j  
@Component  
public class InternalService {  
    public void internal() {  
        log.info("call internal");  
    }  
}
```
- 기존에 `CallService`에서 호출하던 내부 메서드를 외부로 분리한다.
- `InternalService`를 독립적인 스프링 빈으로 등록한다.

```java
@Slf4j  
@Component  
@RequiredArgsConstructor  
public class CallServiceV3 {  
    private final InternalService internalService;  
  
    public void external() {  
        log.info("call external");  
        internalService.internal();  
    }  
}
```

```
2025-01-05T11:21:41.547+09:00  INFO 4519 --- [aop] [           main] h.aop.internalcall.aop.CallLogAspect     : aop = void hello.aop.internalcall.CallServiceV3.external()
2025-01-05T11:21:41.548+09:00  INFO 4519 --- [aop] [           main] hello.aop.internalcall.CallServiceV3     : call external
2025-01-05T11:21:41.548+09:00  INFO 4519 --- [aop] [           main] h.aop.internalcall.aop.CallLogAspect     : aop = void hello.aop.internalcall.InternalService.internal()
2025-01-05T11:21:41.548+09:00  INFO 4519 --- [aop] [           main] hello.aop.internalcall.InternalService   : call internal
```
- `CallService`에서는 분리한 `InternalService`를 주입 받아 사용한다.
- 덕분에 자연스럽게 AOP가 적용된다.

## 정리
- 스프링 AOP가 적용되려면 항상 프록시 객체를 통해 대상 객체(Target)을 호출해야 한다.
- 프록시 객체의 대상 객체가 되는 인스턴스의 내부에서 호출이 발생하면 AOP가 적용되지 않는다.


---

References: 김영한의 스프링 핵심 원리 - 고급편

Links to this page: [[트랜잭션 문제 해결 - 트랜잭션 AOP 이해]], [[스프링 컨테이너와 스프링 빈]], [[빈 스코프]]