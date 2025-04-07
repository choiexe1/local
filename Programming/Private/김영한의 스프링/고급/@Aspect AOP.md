---
title: 
tags:
  - java
  - proxy
  - programming
  - spring
  - aop
publish: true
date: 2025-01-03 19:44
---
## 배경
- 스프링 어플리케이션에 프록시를 적용하려면 `Advisor`를 만들어서 스프링 빈으로 등록하면 된다.
- 앞서 배운 자동 프록시 생성기가 `Advisor`를 찾아서 모두 자동으로 처리해주기 때문이다.
- `Advisor`를 직접 구현하는 것도 편리하긴 하지만, `Advice`와 `Pointcut`을 각각 따로 구현해야 한다.
- 스프링은 `@Aspect`, `@Around` 어노테이션으로 매우 편리하게 `Pointcut`과 `Advice`로 구성 되어 있는 `Advisor` 생성 기능을 지원한다.

> `@Aspect`는 관점 지향 프로그래밍(AOP)을 가능하게 하는 `AspectJ` 프로젝트에서 제공하는 어노테이션이다. 스프링은 이것을 차용해서 프록시를 통한 AOP를 가능하게 한다.

## @Aspect
- 어노테이션 기반 프록시를 적용할 때 필요하다.
- 이 어노테이션을 적용하면 스프링이 제공하는 [[스프링이 제공하는 빈 후처리기|자동 프록시 생성 빈 후처리기]]가 `Advisor`로 변환하는 작업을 수행한다.

## @Around
- 포인트컷 역할을 하는 어노테이션이다.
- `@Around()`의 값에 포인트컷 표현식을 넣는다. 표현식은 `AspectJ` 표현식을 사용한다.
- `@Around()`가 적용된 메서드는 어드바이스가 된다.

## 자동 프록시 생성기의 두 가지 기능
- 스프링이 제공하는 자동 프록시 생성 빈 후처리기 `AnnotationAwareAspectJAutoProxyCreator`는 `Advisor`를 찾아서 필요한 프록시를 생성하고 적용한다.
- 추가적으로 `@Aspect`를 찾아서 이것을 `Advisor`로 변환한다.
- 그래서 이름 앞에 `AnnotationAware`(어노테이션을 인식하는)가 붙어있는 것이다.

## @Aspect를 어드바이저로 변환해서 저장하는 과정

![[aspect-aop-1.png]]
1. **실행**: 스프링 어플리케이션 로딩 시점에 자동 프록시 생성기를 호출한다.
2. **모든 @Aspect 빈 조회**: 자동 프록시 생성기는 스프링 컨테이너에서 `@Aspect` 어노테이션이 적용된 스프링 빈을 모두 조회한다.
3. **어드바이저 생성**: `@Aspect` 어드바이저 빌더를 통해 `@Aspect` 어노테이션 정보를 기반으로 어드바이저를 생성한다.
4. **@Aspect 기반 어드바이저 저장**: 생성한 어드바이저를 `@Aspect` 어드바이저 빌더 내부에 저장한다.

## 자동 프록시 생성 전체 흐름
![[aspect-aop-2.png]]
1. **생성**: 스프링 빈 대상이 되는 객체를 생성한다.
2. **전달**: 생성된 객체를 빈 저장소에 등록하기 직전에 빈 후처리기에 전달한다.
3. **Advisor 조회**
	1. **Advisor 빈 조회**: 스프링 컨테이너에서 `Advisor` 빈을 모두 조회한다.
	2. **@Aspect를 변환한 Advisor 조회**: `@Aspect` 어드바이저 빌더 내부에 저장된 `Advisor`를 모두 조회한다.
4. **프록시 적용 대상 체크**
	1. 앞서 조회한 `Advisor`에 포함되어 있는 포인트컷을 사용해서 해당 객체가 프록시를 적용할 대상인지 판단한다. 
	2. 해당 객체의 모든 메서드를 포인트컷에 하나 하나 매칭해서 조건이 하나라도 만족되면 프록시 적용 대상이 된다.
6. **프록시 생성**: 프록시 적용 대상이면 프록시를 생성하고 프록시를 반환한다. 만약 프록시 적용 대상이 아니라면 원본 객체를 반환한다.
7. **빈 등록**: 반환된 객체는 스프링 빈으로 등록된다.


## 예시

#### @Aspect와 @Around를 이용한 AOP
```java title="LogTraceAspect"
@Slf4j  
@Aspect  
public class LogTraceAspect {  
    private final LogTrace trace;  
  
    public LogTraceAspect(LogTrace trace) {  
        this.trace = trace;  
    }  
  
    @Around("execution(* hello.proxy.app..*(..))")  
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {  
        TraceStatus status = null;  
  
        try {  
            String message = joinPoint.getSignature().toShortString();  
  
            status = trace.begin(message);  
  
            Object result = joinPoint.proceed();  
  
            trace.end(status);  
            return result;  
        } catch (Exception e) {  
            trace.exception(status,e);  
            throw e;  
        }  
    }  
}
```
- `ProceedingJoinPoint`는 어드바이스에서 살펴본 `MethodInvocation invocation`과 유사한 기능이다.
- 내부에 실제 호출 대상, 전달 인자, 어떤 객체와 어떤 메서드가 호출되었는지 정보가 포함되어 있다.
- 실제 호출 대상을 호출하기 위해서는 `joinPoint.proceed()`를 호출하면 된다.

```java title="AopConfig"
@Configuration  
@Import({AppV1Config.class, AppV2Config.class})  
public class AopConfig {  
    @Bean  
    public LogTraceAspect logTraceAspect(LogTrace trace) {  
        return new LogTraceAspect(trace);  
    }  
}
```
- 여기서 등록한 스프링 빈 `LogTraceAspect`는 어드바이저로 변환이 된다.

## 정리
- `@Aspect`를 사용해서 어노테이션 기반 프록시를 매우 편리하게 적용할 수 있다.
- 로그 추적기처럼 어플리케이션 전반에 로그를 남기는 기능은 특정 기능 하나에 관심이 있는 것이 아니다.
- 어플리케이션 여러 기능들 사이에 걸쳐 들어가는 관심사이다.
- 이것을 바로 **횡단 관심사(cross-cutting concerns)**라고 한다.
- 지금까지 학습한 프록시가 바로 이 횡단 관심사 문제를 해결하는 방법이다.

![[aspect-aop-3.png]]


---

References: 김영한의 스프링 핵심 원리 - 고급편

Links to this page: [[스프링이 제공하는 빈 후처리기]]
