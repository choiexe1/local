---
title: 
tags:
  - java
  - spring
  - pointcut
  - aop
publish: true
date: 2025-01-04 11:40
comments: true
---
## 배경
- 포인트컷은 어드바이스를 적용할지의 여부에 대해 판별하는 기능이다.
- 스프링 AOP에서의 포인트컷은 `AspectJ` 표현식을 차용해서 사용한다.

## 포인트컷 지시자(Pointcut Designator)
- `@Pointcut("execution(* hello.aop.order..*(..))")`
- 포인트컷 표현식은 `execution`과 같이 포인트컷 지시자로 시작한다.

## execution 문법
```
execution(접근제어자? 반환타입 선언타입?메서드이름(파라미터) 예외?)
```
- 메소드 실행 조인 포인트를 매칭한다.
- `?`는 생략할 수 있다.
- `*` 같은 패턴을 지정할 수 있다.

## 예시
- 먼저 예시를 위해 다음의 순서로 프로젝트를 구성한다.
- 어노테이션 생성
	- `ClassAop`
	- `MethodAop`

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface ClassAop {  
}

@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface MethodAop {  
    String value();  
}
```
- `@Target`은 자바의 클래스 타입에 적용할 것인지, 메서드에 적용할 것인지와 같은 적용 유형을 의미한다.
- `@Retention`은 어느 시점까지 타겟의 메모리를 유지할 것인지 결정하는 어노테이션이다.
	- 위에서는 `RetentionPolicy.Runtime`을 통해서 런타임에 어노테이션 정보를 메모리에 갖고 있는다는 것이다.
	- 즉, `Reflection API`등을 사용해서 어노테이션 정보를 알 수 있다는 의미가 된다.
- `MethodAop`는 문자열 타입의 `value` 속성을 가진다.
- 다음으로 `MemberService` 인터페이스와 구현체를 구현한다.
```java
public interface MemberService {  
    String hello(String param);  
}

@ClassAop  
@Component  
public class MemberServiceImpl implements MemberService {  
  
    @Override  
    @MethodAop("test value")  
    public String hello(String param) {  
        return "ok";  
    }  
  
    public String internall(String param) {  
        return "ok";  
    }  
}
```


### execution 지시자
#### 정확히 매칭
```java
@Slf4j  
@SpringBootTest  
public class ExecutionTest {  
    AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();  
    Method helloMethod;  
  
    @BeforeEach  
    void init() throws NoSuchMethodException {  
        helloMethod = MemberServiceImpl.class.getMethod("hello", String.class);  
    }  
  
    @Test  
    void printMethod() {  
        // public java.lang.String hello.aop.member.MemberServiceImpl.hello(java.lang.String)
        log.info("helloMethod = {}", helloMethod);  
    }  
}
```
- `printMethod()`를 실행해보면 주석 처리 된 내용과 동일한 `public java.lang.String hello.aop.order.aop.member.MemberServiceImpl.hello(java.lang.String)`가 출력 된다.
- 다음의 포인트컷 표현식은 이 메소드 정보와 정확히 매치된다.
```java
@Test  
void exactMatch() {  
    // public java.lang.String hello.aop.member.MemberServiceImpl.hello(java.lang.String)에 정확히 매치  
    pointcut.setExpression("execution(public String hello.aop.member.MemberServiceImpl.hello(String))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```
- **매칭 조건**
	- 접근제어자?: `public`
	- 반환타입: `String`
	- 선언타입?: `hello.aop.member.MemberServiceImpl`
	- 메서드이름: `hello`
	- 파라미터: `(String)`
	- 예외?: 생략
- `MemberServiceImpl.hello(String)` 메서드와 포인트컷 표현식의 모든 내용이 정확하게 일치한다. 따라서 `true`를 반환하므로 테스트를 통과한다.

#### 가장 많이 생략한 포인트컷
```java
@Test  
void allMatch() {  
    pointcut.setExpression("execution(* *(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```
- **매칭 조건**
	- 접근제어자?: 생략
	- 반환타입: `*`
	- 선언타입?: 생략
	- 메서드이름: `*`
	- 파라미터: `(..)`
	- 예외?: 생략
- 파라미터에서 `..`은 파라미터의 타입과 파라미터 수가 상관없다는 뜻이다.

#### 이름으로 매치
```java
@Test  
void nameMatch() {  
    pointcut.setExpression("execution(* hello(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}

@Test  
void namePatternMatch() {  
    pointcut.setExpression("execution(* hel*(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}

@Test  
void namePatternMatch2() {  
    pointcut.setExpression("execution(* *el*(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```
- 메서드 이름으로 매치하는 포인트컷이다.
- 이름의 앞 뒤에 `*`을 사용해 와일드카드로 사용할 수 있다.

#### 패키지 매칭
```java
@Test  
void packageExactMatch1() {  
    pointcut.setExpression("execution(* hello.aop.member.MemberServiceImpl.*o*(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}

@Test  
void packageExactMatch2() {  
    pointcut.setExpression("execution(* hello.aop.member.*.*(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}

@Test  
void packageExactFalse() {  
    pointcut.setExpression("execution(* hello.aop.*.*(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();  
}  
  
@Test  
void packageMatchSubPackage1() {  
    pointcut.setExpression("execution(* hello.aop.member..*.*(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
@Test  
void packageMatchSubPackage2() {  
    pointcut.setExpression("execution(* hello.aop..*.*(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```
- `hello.aop.member.*(1).*(2)`
	- `(1)`: 타입
	- `(2)`: 메서드 이름
- 패키지에서 `.`과 `..`의 차이를 이해해야 한다.
- `.`: 정확히 해당 위치의 패키지
- `..`: 해당 위치의 패키지와 그 하위 패키지도 포함

#### 타입 매치
```java
@Test  
void typeExactMatch() {  
    pointcut.setExpression("execution(* hello.aop.member.MemberServiceImpl.*(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
@Test  
void typeSuperMatch() {  
    pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```
- `typeExactMatch()`는 타입 정보가 정확하게 일치하기 때문에 매칭된다.
- `typeSuperMatch()`는 부모 타입을 선언해도 그 자식 타입은 매칭된다.
	- `MemberServiceImpl`은 `MemberService` 인터페이스를 구현한 자식 타입이다.
	- 따라서 매칭된다.

```java
@Test  
void typeMatchInternal() throws NoSuchMethodException {  
    pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");  
      
    Method internallMethod = MemberServiceImpl.class.getMethod("internal", String.class);  
    assertThat(pointcut.matches(internallMethod, MemberServiceImpl.class)).isFalse();  
}
```
- 슈퍼 타입을 표현식에 선언한 경우 슈퍼에 선언한 메서드가 자식 타입에 있어야 매칭에 성공한다.
- 따라서 이 경우 `MemberServiceImpl`의 슈퍼 타입인 `MemberService` 인터페이스에 정의되지 않은 메서드는 매치되지 않는다.

#### 파라미터 매치
```java
// 모든 타입 허용  
// 정확히 하나의 파라미터 허용  
@Test  
void argsMatchStar() {  
    pointcut.setExpression("execution(* *(*))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
// 모든 타입 허용  
// 숫자와 무관하게 모든 파라미터 허용  
// (), (xxx), (xxx, xxx)  
@Test  
void argsMatchAll() {  
    pointcut.setExpression("execution(* *(..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}  
  
// String 타입으로 시작  
// 숫자와 무관하게 모든 파라미터와 모든 타입 허용  
// (String), (String, xxx), (String, xxx, xxx)  
@Test  
void argsMatchComplex() {  
    pointcut.setExpression("execution(* *(String, ..))");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}
```
- `hello` 메서드는 파라미터로 문자열을 받기 때문에 `argsMatch()`의 경우 테스트가 통과한다. 
- 반면 `noArgsMatch()`는 테스트에 실패한다. 아무 파라미터도 받지 않는 `hello` 메서드가 존재하지 않기 때문이다.
- **execution 파라미터 매칭 규칙**
	- `(String)`: 정확히 `String` 타입 파라미터
	- `()`: 파라미터가 없어야한다.
	- `(*)`: 모든 타입을 허용하는 하나의 파라미터
	- `(*, *)`: 모든 타입을 허용하는 두 개의 파라미터
	- `(..)`: 모든 타입을 허용하는 `N`개의 파라미터, `0..*`로 이해하면 된다.
	- `(String, ..)`: 첫 파라미터가 `String`로 시작해야하고 그 뒤로는 모든 타입을 허용하는 `N`개의 파라미터

### within 지시자
- `within` 지시자는 특정 타입 내의 조인 포인트에 대한 매칭을 제한한다.
- 쉽게 이야기해서 해당 타입이 매칭되면 그 안의 메서드(조인 포인트)들이 자동으로 매칭된다.

```java
@Test  
void withinExact() throws NoSuchMethodException {  
    pointcut.setExpression("within(hello.aop.member.MemberServiceImpl)");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
    assertThat(pointcut.matches(internal, MemberServiceImpl.class)).isTrue();  
}
```
- `within` 지시자로 `MemberServiceImpl`의 모든 메서드가 자동으로 매칭 된다.
- 따라서 해당 타입의 메서드들은 모두 매칭된다.

```java
@Test  
void withinStar() {  
    pointcut.setExpression("within(hello.aop.member.*Service*)");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();  
    assertThat(pointcut.matches(internal, MemberServiceImpl.class)).isTrue();  
}
```
- `execution` 지시자와 마찬가지로 와일드카드를 사용할 수 있다.

```java
@Test  
void withinSuperType() {  
    pointcut.setExpression("within(hello.aop.member.*Service)");  
    assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();  
    assertThat(pointcut.matches(internal, MemberServiceImpl.class)).isFalse();  
}
```
- 슈퍼 타입인 `MemberService` 인터페이스와 매칭을 시도하는 케이스이다.
- `within` 지시자를 사용할 때는 정확하게 타입이 일치해야만 매칭된다. 하위 타입은 매칭되지 않는다는 이야기다.
- 이 부분에서 `execution` 지시자와 차이가 있다.

### args 지시자
- `args` 지시자는 인자가 주어진 타입의 인스턴스인 조인 포인트로 매칭한다.
- 기본 문법은 `execution` 지시자의 `args` 부분과 같다.
- **execution 지시자와 args 지시자의 차이**
	- `execution` 지시자는 파라미터 타입이 정확하게 매칭되어야 한다. `execution`은 클래스에 선언된 정보를 기반으로 판단한다.
	- `args`는 부모 타입을 허용한다. `args`는 실제 넘어온 파라미터 객체 인스턴스를 보고 판단한다.

```java
@Test  
void args() {  
    //hello(String)과 매칭  
    assertThat(pointcut("args(String)")  
            .matches(helloMethod, MemberServiceImpl.class)).isTrue();  
    assertThat(pointcut("args(Object)")  
            .matches(helloMethod, MemberServiceImpl.class)).isTrue();  
    assertThat(pointcut("args()")  
            .matches(helloMethod, MemberServiceImpl.class)).isFalse();  
    assertThat(pointcut("args(..)")  
            .matches(helloMethod, MemberServiceImpl.class)).isTrue();  
    assertThat(pointcut("args(*)")  
            .matches(helloMethod, MemberServiceImpl.class)).isTrue();  
    assertThat(pointcut("args(String,..)")  
            .matches(helloMethod, MemberServiceImpl.class)).isTrue();  
}

@Test  
void argsVsExecution() {  
    //Args  
    assertThat(pointcut("args(String)")  
            .matches(helloMethod, MemberServiceImpl.class)).isTrue();  
    assertThat(pointcut("args(java.io.Serializable)")  
            .matches(helloMethod, MemberServiceImpl.class)).isTrue();  
    assertThat(pointcut("args(Object)")  
            .matches(helloMethod, MemberServiceImpl.class)).isTrue();  
  
    //Execution  
    assertThat(pointcut("execution(* *(String))")  
            .matches(helloMethod, MemberServiceImpl.class)).isTrue();  
    assertThat(pointcut("execution(* *(java.io.Serializable))") //매칭 실패  
            .matches(helloMethod, MemberServiceImpl.class)).isFalse();  
    assertThat(pointcut("execution(* *(Object))") //매칭 실패  
            .matches(helloMethod, MemberServiceImpl.class)).isFalse();  
}
```
- `args`는 상위 타입을 허용한다.
- `execution`은 정확하게 매칭되어야 한다.

### @Target, @Within
- 정의와 설명이 너무 추상적이고 이해하기 어려워서 생략한다.
- 다음의 예시를 살펴보자, 두 어노테이션을 적용한다고 가정한다.
- `@target(hello.aop.member.annotation.ClassAop)`
- `@within(hello.aop.member.annotation.ClassAop)`

```java
@ClassAop  
@Component  
public class MemberServiceImpl implements MemberService {  
  
    @Override  
    @MethodAop("test value")  
    public String hello(String param) {  
        return "ok";  
    }  
  
    public String internal(String param) {  
        return "ok";  
    }  
}
```
- `MemberServiceImpl` 타입엔 `@ClassAop` 어노테이션이 적용되어 있다.
- `MemberServiceImpl` 타입의 메서드 중 `hello()`엔 `@MethodAop` 어노테이션이 적용되어 있다.
- **@target vs @within**
	- `@target`은 클래스(타입) 인스턴스의 모든 메서드를 조인 포인트로 적용한다.
	- `@within`은 해당 타입 내부에 있는 메서드만 조인 포인트로 적용한다.

![[pointcut-designater-1.png]]
```java
@Slf4j  
@Import({AtTargetAtWithinTest.Config.class})  
@SpringBootTest  
public class AtTargetAtWithinTest {  
    @Autowired  
    Child child;  
  
    @Test  
    void success() {  
        log.info("child Proxy={}", child.getClass());  
        child.childMethod();  
        child.parentMethod();  
    }  
  
    static class Config {  
        @Bean  
        public Parent parent() {  
            return new Parent();  
        }  
  
        @Bean  
        public Child child() {  
            return new Child();  
        }  
  
        @Bean  
        public AtTargetAtWithinAspect atTargetAtWithinAspect() {  
            return new AtTargetAtWithinAspect();  
        }  
    }  
  
    static class Parent {  
        public void parentMethod() {  
        }  
    }  
  
    @ClassAop  
    static class Child extends Parent {  
        public void childMethod() {  
        }  
    }  
  
    @Slf4j  
    @Aspect
    static class AtTargetAtWithinAspect {  
        //
        @Around("execution(* hello.aop..*(..)) && @target(hello.aop.member.annotation.ClassAop)")  
        public Object atTarget(ProceedingJoinPoint joinPoint) throws Throwable {  
            log.info("[@target] {}", joinPoint.getSignature());  
            return joinPoint.proceed();  
        }  
  
        //
        @Around("execution(* hello.aop..*(..)) && @within(hello.aop.member.annotation.ClassAop)")  
        public Object atWithin(ProceedingJoinPoint joinPoint) throws Throwable {  
            log.info("[@within] {}", joinPoint.getSignature());  
            return joinPoint.proceed();  
        }  
    }  
}
```
- `@target`
	- 인스턴스 기준으로 모든 메서드의 조인 포인트를 선정, 부모 타입의 메서드도 적용  
- `@within`
	- 선택된 클래스 내부에 있는 메서드만 조인 포인트로 선정, 부모 타입의 메서드는 적용되지 않음  

### @annotation 지시자
- `@annotation(hello.aop.member.annotation.MethodAop)`
- `@annotation`: 메서드가 주어진 어노테이션을 가지고 있는 조인 포인트를 매칭
- 메서드가 주어진 어노테이션이 적용 되어 있다면 매칭하는 것이다.

```java
@Slf4j  
@SpringBootTest  
@Import(AtAnnotationAspect.class)  
public class AtAnnotationTest {  
    @Autowired  
    MemberService memberService;  
  
    @Test  
    void success() {  
        log.info("memberService Proxy = {}", memberService.getClass());  
        memberService.hello("helloA");  
    }  
  
    @Slf4j  
    @Aspect    
    static class AtAnnotationAspect {  
        @Around("@annotation(hello.aop.member.annotation.MethodAop)")  
        public Object doAtAnnotation(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {  
            log.info("[@annotation] {}", proceedingJoinPoint.getSignature());  
            return proceedingJoinPoint.proceed();  
        }  
    }  
}
```

### @args 지시자
- `@args`: 전달된 인수의 런타임 타입에 `@Check` 어노테이션이 있는 경우에 매칭
- `@args(test.Check)`

### bean 지시자
- `bean` 지시자는 스프링 전용 포인트컷 지시자이다. 빈의 이름으로 지정한다.
- 말 그대로 스프링 빈의 이름으로 AOP 적용 여부를 지정한다.
- `bean(orderService) || bean(*Repository)`

```java
@Slf4j  
@SpringBootTest  
@Import(BeanTest.BeanAspect.class)  
public class BeanTest {  
    @Autowired  
    OrderService orderService;  
  
    @Test  
    void success() {  
        orderService.orderItem("itemA");  
    }  
  
    @Aspect  
    static class BeanAspect {  
        @Around("bean(orderService) || bean(*Repository)")  
        public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {  
            log.info("[bean] {}", joinPoint.getSignature());  
            return joinPoint.proceed();  
        }  
    }  
}
```

### 매개변수 전달
- 포인트컷 표현식을 사용해서 어드바이스에 매개변수를 전달할 수 있다.
- `this`, `target`, `args`, `@target`, `@within`, `@annotation`, `@args`

```java
@Slf4j  
@SpringBootTest  
@Import(ParameterTest.ParameterAspect.class)  
public class ParameterTest {  
    @Autowired  
    MemberService memberService;  
  
    @Test  
    void success() {  
        log.info("memberService Proxy = {}", memberService.getClass());  
        memberService.hello("helloA");  
    }  
  
    @Slf4j  
    @Aspect    
    static class ParameterAspect {  
        @Pointcut("execution(* hello.aop.member..*.*(..))")  
        private void allMember() {  
        }  
  
        @Around("allMember()")  
        public Object logArgs1(ProceedingJoinPoint joinPoint) throws Throwable {  
            Object arg1 = joinPoint.getArgs()[0];  
            log.info("[logArgs1] {}, arg = {}", joinPoint.getSignature(), arg1);  
            return joinPoint.proceed();  
        }  
    }  
}
```
- **출력 결과**
	- `[logArgs1] String hello.aop.member.MemberServiceImpl.hello(String), argType = class java.lang.String, arg = helloA`
	- `joinPoint.getArgs()[0]`으로 파라미터를 가져올 수 있다.

```java
@Around("allMember() && args(arg, ..)")  
public Object logArgs2(ProceedingJoinPoint joinPoint, Object arg) throws Throwable {  
    log.info("[logArgs2] {}, arg = {}", joinPoint.getSignature(), arg);  
    return joinPoint.proceed();  
}
```
- 어드바이스의 파라미터로 `arg`를 가져올 수 있다.


```java
@Before("allMember() && args(arg, ..)")  
public void logArgs3(String arg) {  
    log.info("[logArgs3] arg = {}", arg);  
}
```
- 파라미터 타입을 문자열로 지정해주어도 된다.
- 참고로 포인트컷의 타입과 어드바이스의 타입이 맞지 않으면 포인트컷과 매칭 자체가 되지 않아서 실행되지 않는다.
- 위의 경우 `arg`로 첫 번째 파라미터의 이름을 정의 했는데, 어떤 타입이든 첫 번째 매개변수가 있고, 그 뒤에 어떤 타입이든 `N`개의 파라미터가 있는 메서드에만 적용된다.

```java
@Before("allMember() && this(obj)")  
public void thisArgs(JoinPoint joinPoint, MemberService obj) {  
    log.info("[this] {}, obj = {}", joinPoint.getSignature(), obj.getClass());  
}  
  
@Before("allMember() && target(obj)")  
public void targetArgs(JoinPoint joinPoint, MemberService obj) {  
    log.info("[target] {}, obj = {}", joinPoint.getSignature(), obj.getClass());  
}
```
- `this`는 런타임에 실행되고 있는 인스턴스를 의미한다.
	- 따라서 **프록시 객체 정보**를 의미한다.
- `target`은 인스턴스의 클래스 정보이다.
	- 따라서 프록시 객체의 타겟인 **실제 대상 타입 정보**를 의미한다.

```java
@Before("allMember() && @target(annotation)")  
public void atTarget(JoinPoint joinPoint, ClassAop annotation) {  
    log.info("[@target] {}, obj = {}", joinPoint.getSignature(), annotation);  
}  
  
@Before("allMember() && @within(annotation)")  
public void atWithin(JoinPoint joinPoint, ClassAop annotation) {  
    log.info("[@within] {}, obj = {}", joinPoint.getSignature(), annotation);  
}  
  
@Before("allMember() && @annotation(annotation)")  
public void atAnnotation(JoinPoint joinPoint, MethodAop annotation) {  
    log.info("[@annotation] {}, annotationValue = {}", joinPoint.getSignature(), annotation.value());  
}
```
- `@target`, `@within`은 해당 타입의 어노테이션을 전달 받는다.
- `@annotation`은 메서드의 어노테이션을 전달 받는다.
	- 여기서 `atAnnotation`을 주의 깊게 보면, `annotation.value()`를 통해서 어노테이션에 적용 시의 `value` 속성을 가져올 수 있다.
---

References: 김영한의 스프링 핵심 원리 - 고급편

Links to this page: [[스프링 AOP 실전 예제]]
