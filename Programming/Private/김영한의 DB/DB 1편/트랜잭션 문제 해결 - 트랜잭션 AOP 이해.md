---
title: 
tags:
  - java
  - programming
  - spring
  - aop
  - transaction
  - database
publish: true
date: 2024-12-13 00:01
---

## 트랜잭션 문제 해결 - 트랜잭션 AOP 이해

지금까지 트랜잭션을 편리하게 처리하기 위해서 트랜잭션 추상화도 도입하고, 추가로 반복적인 트랜잭션 로직을 해결하기 위해 트랜잭션 템플릿도 도입했다.

트랜잭션 템플릿 덕분에 트랜잭션을 처리하는 반복 코드는 해결할 수 있었다. 하지만 서비스 계층에 순수한 비즈니스 로직만 남긴다는 목표는 아직 달성하지 못했다.

이럴 때 스프링 AOP를 통해 프록시를 도입하면 문제를 깔끔하게 해결할 수 있다. 참고로 AOP는 앞서 [[스프링 입문 2#AOP (Aspect Oriented Programming)]]에서 잠깐 알아봤었다.

## 프록시를 통한 문제 해결

프록시(Proxy)의 뜻은 대리인으로 무언가를 대신 처리해준다는 의미다.

**프록시 도입 전**

![[transaction-aop-1.png]]

프록시를 도입하기 전에는 다음과 같이 서비스 로직에서 트랜잭션을 직접 시작한다.

```java
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

try {
	bizLogic(fromId, toId, money);
	transactionManager.commit(status);
} catch (Exception e) {
	transactionManager.rollback(status);
	throw new IllegalStateException(e);
}
```

**프록시 도입 후**

![[transaction-aop-2.png]]

프록시를 사용하면 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 명확하게 분리할 수 있다.

```java title="트랜잭션 프록시 코드 예시"
public class TransactionProxy {
	private MemberService target;

	public void logic() {
		TransactionStatus status = transactionManager.getTransaction(..);

		try {
			target.logic();
			transactionManager.commit(status);
		} catch (Exception e) {
			transactionManager.rollback(status);
			throw new IllegalStateException(e);
		}
	}
}
```

```java title="트랜잭션 프록시 적용 후 서비스 코드 예시"
public class Service {
	public void logic() {
		//트랜잭션 관련 코드 제거, 순수 비즈니스 로직만 남음
		bizLogic(fromId, toId, money);
	}
}
```

- **프록시 도입 전**: 서비스에 비즈니스 로직과 트랜잭션 처리 로직이 함께 섞여있다.
- **프록시 도입 후**: 트랜잭션 프록시가 트랜잭션 처리 로직을 모두 가져간다. 그리고 트랜잭션을 시작한 후에 실제 서비스를 대신 호출한다. 트랜잭션 프록시 덕분에 서비스 계층에는 순수한 비즈니스 로직만 남길 수 있다.

## 스프링이 제공하는 트랜잭션 AOP

- 스프링이 제공하는 AOP 기능을 사용하면 프록시를 매우 편리하게 적용할 수 있다.
- 물론 스프링 AOP를 직접 사용해서 트랜잭션을 처리해도 되지만, 트랜잭션은 매우 중요한 기능이고 전세계 누구나 다 사용하는 기능이다. 따라서 스프링은 트랜잭션 AOP를 처리하기 위한 모든 기능을 제공한다.
- 그리고 스프링 부트를 사용하면 트랜잭션 AOP를 처리하기 위해 필요한 스프링 빈들도 자동으로 등록해준다.
- 개발자는 트랜잭션 처리가 필요한 곳에 `@Transactional` 어노테이션만 붙여주면 된다. 스프링의 트랜잭션 AOP는 이 어노테이션을 인식해서 트랜잭션 프록시를 적용한다.

**@Transactional**

- `org.springframework.transaction.annotaion.Transactional`

## 트랜잭션 문제 해결 - 트랜잭션 AOP 적용

트랜잭션 AOP를 사용하는 새로운 서비스 클래스를 만든다.

```java title="MemberServiceV3_3.java"
/**
 * 트랜잭션 - @Transactional AOP
 */
@Slf4j
@RequiredArgsConstructor
public class MemberServiceV3_3 {
    private final MemberRepositoryV3 memberRepository;

    @Transactional
    public void transfer(String fromId, String toId, int money) throws SQLException {
        bizLogic(fromId, toId, money);
    }

    private void bizLogic(String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);

        memberRepository.update(fromId, fromMember.getMoney() - money);
        validation(toMember);
        memberRepository.update(toId, toMember.getMoney() + money);
    }

    private static void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체 중 예외 발생");
        }
    }
}
```

- 순수한 비즈니스 로직만 남기고, 트랜잭션 관련 코드는 모두 제거했다.
- 스프링이 제공하는 트랜잭션 AOP를 적용하기 위해 `@Transactional` 어노테이션을 추가했다.
- `@Transactional` 어노테이션은 메서드에 붙여도 되고, 클래스에 붙여도 된다. **클래스에 붙이면 외부에서 호출 가능한 `public` 메서드가 AOP 적용 대상이 된다.**

이제 다음과 같이 테스트 코드를 작성한다.

```java title="MemberServiceV3_3Test.java"
class MemberServiceV3_3Test {
    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";

    private MemberRepositoryV3 memberRepositoryV3;
    private MemberServiceV3_3 memberServiceV3_3;

    @BeforeEach
    void beforeEach() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        memberRepositoryV3 = new MemberRepositoryV3(dataSource);
        memberServiceV3_3 = new MemberServiceV3_3(memberRepositoryV3);
    }

    @AfterEach
    void afterEach() throws SQLException {
        memberRepositoryV3.delete(MEMBER_A);
        memberRepositoryV3.delete(MEMBER_B);
        memberRepositoryV3.delete(MEMBER_EX);
    }

    @Test
    @DisplayName("정상 이체")
    void transfer() throws SQLException {
        // GIVEN
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberB = new Member(MEMBER_B, 10000);

        memberRepositoryV3.save(memberA);
        memberRepositoryV3.save(memberB);

        // WHEN
        memberServiceV3_3.transfer(memberA.getMemberId(), memberB.getMemberId(), 2000);

        // THEN
        Member findMemberA = memberRepositoryV3.findById(memberA.getMemberId());
        Member findMemberB = memberRepositoryV3.findById(memberB.getMemberId());

        Assertions.assertThat(findMemberA.getMoney()).isEqualTo(8000);
        Assertions.assertThat(findMemberB.getMoney()).isEqualTo(12000);
    }

    @Test
    @DisplayName("이체중 예외 발생")
    void transferToEx() throws SQLException {
        // GIVEN
        Member memberA = new Member(MEMBER_A, 10000);
        Member memberEx = new Member(MEMBER_EX, 10000);

        memberRepositoryV3.save(memberA);
        memberRepositoryV3.save(memberEx);

        // WHEN
        Assertions
                .assertThatThrownBy(
                        () -> memberServiceV3_3.transfer(memberA.getMemberId(), memberEx.getMemberId(), 2000))
                .isInstanceOf(IllegalStateException.class);

        // THEN
        Member findMemberA = memberRepositoryV3.findById(memberA.getMemberId());
        Member findMemberB = memberRepositoryV3.findById(memberEx.getMemberId());

        Assertions.assertThat(findMemberA.getMoney()).isEqualTo(10000);
        Assertions.assertThat(findMemberB.getMoney()).isEqualTo(10000);
    }
}
```

테스트를 실행해보면, 이체중 예외 발생 테스트에서 예외 발생 시 롤백이 되지 않고 있다. 이유는 현재 테스트에서는 스프링 컨테이너를 사용하지 않고 직접 객체를 생성해서 의존 관계를 주입하고 있다.

> 스프링 AOP를 적용하려면 어드바이저, 포인트컷, 어드바이스가 필요하다. 스프링은 트랜잭션 AOP 처리를 위해 다음 클래스를 제공한다.
>
> 스프링 부트를 사용하면 해당 빈들은 스프링 컨테이너에 자동으로 등록된다.
>
> - 어드바이저: `BeanFactoryTransactionAttributeSourceAdvisor`
> - 포인트컷: `TransactionAttributeSourcePointcut`
> - 어드바이스: `TransactionInterceptor`

스프링 AOP는 스프링 컨테이너 없이 적용할 수 없다. 따라서 이제 스프링 컨테이너를 사용해서 필요한 클래스들을 외부에서 주입 받도록 테스트를 다음과 같이 변경한다.

```java
@SpringBootTest
class MemberServiceV3_3Test {
    public static final String MEMBER_A = "memberA";
    public static final String MEMBER_B = "memberB";
    public static final String MEMBER_EX = "ex";

    @Autowired
    private MemberRepositoryV3 memberRepositoryV3;
    @Autowired
    private MemberServiceV3_3 memberServiceV3_3;

    @TestConfiguration
    static class TestConfig {
        @Bean
        DataSource dataSource() {
            return new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        }

        @Bean
        PlatformTransactionManager transactionManager() {
            return new DataSourceTransactionManager(dataSource());
        }

        @Bean
        MemberRepositoryV3 memberRepositoryV3() {
            return new MemberRepositoryV3(dataSource());
        }

        @Bean
        MemberServiceV3_3 memberServiceV3_3() {
            return new MemberServiceV3_3(memberRepositoryV3());
        }
    }
}
```

- 기존에 직접 생성자를 통해 객체를 생성하고 `@BeforeEach`를 통해 주입하던 코드에서, 객체를 직접 스프링 빈으로 등록하고 컨테이너를 통해 의존관계를 자동 주입 받도록 변경 했다.
- `@SpringBootTest`: 스프링 AOP를 적용하려면 스프링 컨테이너가 필요하다. 이 어노테이션이 있으면 테스트 시 스프링 부트를 통해 스프링 컨테이너를 생성한다. 그리고 테스트에서 `@Autowired`를 통해 스프링 컨테이너가 관리하는 빈들을 사용할 수 있다.
- `@TestConfiguration`: 이 어노테이션을 사용하면 테스트 클래스 안에서 내부 설정 클래스를 만들어서 사용한다. 스프링 부트가 자동으로 생성하는 빈들에 추가로 필요한 스프링 빈들을 등록하고 테스트를 수행할 수 있다.
- `TestConfig`
  - `DataSource`: 스프링에서 기본으로 사용할 데이터소스를 스프링 빈으로 등록한다. 추가로 트랜잭션 매니저에서도 사용한다.
  - `DataSourceTransactionManager`: 트랜잭션 매니저를 스프링 빈으로 등록한다.
    - 스프링이 제공하는 트랜잭션 AOP는 스프링 빈에 등록된 트랜잭션 매니저를 찾아서 사용하기 때문에 트랜잭션 매니저를 스프링 빈으로 등록해두어야 한다.

#### AOP 프록시 적용 확인

다음의 테스트 코드를 통해 정말 스프링 AOP가 적용되서 작동하는지 확인해보자.

```java
@Test
void AopCheck() {
    log.info("memberService class={}", memberService.getClass());
    log.info("memberRepository class={}", memberRepository.getClass());
    Assertions.assertThat(AopUtils.isAopProxy(memberService)).isTrue();
    Assertions.assertThat(AopUtils.isAopProxy(memberRepository)).isFalse();
}
```

출력 결과는 다음과 같다.

```
memberService class=class hello.jdbc.service.MemberServiceV3_3$$SpringCGLIB$$0
memberRepository class=class hello.jdbc.repository.MemberRepositoryV3
```

앞서 `@Transactional`을 적용한건 `MemberServiceV3_3`이므로 `MemberRepository`는 `isAopProxy()` 호출 시 `false`가 반환된다. 테스트가 통과한다.

## 트랜잭션 문제 해결 - 트랜잭션 AOP 정리

트랜잭션 AOP가 적용된 전체 흐름을 정리해본다.

![[transaction-aop-3.png]]

1. `@Transactional` 어노테이션이 적용되어 있으면 스프링이 트랜잭션을 적용하는 프록시 객체를 만든다.
2. 스프링 컨테이너는 의존관계 주입 시 트랜잭션 어노테이션이 적용되어 있는 클래스 대신, 프록시 객체를 주입한다.
3. 클라이언트가 서비스를 호출하면, 프록시 객체가 호출된다.
4. 프록시 객체는 스프링 컨테이너에 등록된 트랜잭션 매니저를 획득한다.
5. 트랜잭션 매니저는 마찬가지로 스프링 컨테이너에 등록된 데이터소스를 가지고 커넥션을 생성하고 수동 커밋 모드로 트랜잭션을 시작한다. 그리고 이 커넥션을 트랜잭션 동기화 매니저에 보관한다.
6. 프록시 객체는 이제 실제 서비스 로직을 호출한다.
7. 실제 서비스 로직은 리포지토리를 호출한다.
8. 리포지토리는 트랜잭션 동기화 매니저에 보관된 커넥션을 획득하고 사용한다.

#### 선언적 트랜잭션 관리 vs 프로그래밍 방식의 트랜잭션 관리

**선언적 트랜잭션 관리**

`@Transactional` 어노테이션 하나만 선언해서 매우 편리하게 트랜잭션을 적용하는 것을 선언적 트랜잭션 관리라 한다. 선언적 트랜잭션 관리는 과거 XML에 설정하기도 했다.

이름 그대로 해당 로직에 트랜잭션을 적용하겠다 라고 어딘가에 선언하기만 하면 트랜잭션이 적용되는 방식이다.

**프로그래밍 방식의 트랜잭션 관리**

트랜잭션 매니저 또는 트랜잭션 템플릿 등을 사용해서 트랜잭션 관련 코드를 직접 작성하는 것을 프로그래밍 방식의 트랜잭션 관리라고 한다.

- 선언적 트랜잭션 관리가 프로그래밍 방식에 비해서 훨씬 간편하고 실용적이기 때문에 실무에서는 대부분 선언적 트랜잭션 관리를 사용한다.
- 프로그래밍 방식의 트랜잭션 관리는 스프링 컨테이너나 스프링 AOP 기술 없이 간단히 사용할 수 있지만 실무에서는 대부분 스프링 컨테이너와 스프링 AOP를 사용하기 때문에 거의 사용되지 않는다.
- 프로그래밍 방식 트랜잭션 관리는 테스트 시에 가끔 사용될 때는 있다.

> [!summary] 정리
>
> - 스프링이 제공하는 선언적 트랜잭션 관리 덕분에 드디어 트랜잭션 관련 코드를 순수한 비즈니스 로직에서 제거할 수 있었다.
> - 개발자는 트랜잭션이 필요한 곳에 `@Transactional` 어노테이션 하나만 추가하면 된다. 나머지는 스프링 트랜잭션 AOP가 자동으로 처리해준다.
> - `@Transactional` 어노테이션의 자세한 사용법은 뒤에서 다시 설명한다.

---

References: 김영한의 스프링 DB 1편

Links to this page: [[스프링 입문 2]], [[스프링 AOP 실무 주의사항]]