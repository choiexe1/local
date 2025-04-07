---
title: 
tags:
  - java
  - programming
  - spring
  - orm
  - jpa
  - database
publish: true
date: 2024-12-16 00:00
---

## JPA 소개

스프링과 `JPA`는 자바 엔터프라이즈(기업) 시장의 주력 기술이다. 스프링이 `DI`를 포함한 어플리케이션 전반의 다양한 기능을 제공한다면, `JPA`는 `ORM` 데이터 접근 기술을 제공한다.

스프링 + 데이터 접근 기술의 조합을 구글 트렌드로 비교했을 때

- 글로벌에서는 스프링 + `JPA` 조합을 80% 이상 사용한다.
- 국내에서도 스프링 + `JPA` 조합을 50% 정도 사용하고, 2015년 부터 점점 그 추세가 증가하고 있다.

`JPA`는 스프링만큼이나 방대하고, 학습해야할 분량도 많다. 하지만 한번 배워두면 데이터 접근 기술에서 매우 큰 생산성 향상을 얻을 수 있다. 대표적으로 `JdbcTemplate`이나 `MyBatis`같은 `SQL Mapper` 기술은 `SQL`을 개발자가 직접 작성해야 하지만, `JPA`를 사용하면 SQL도 `JPA`가 대신 작성하고 처리해준다.

실무에서는 `JPA`를 더욱 편리하게 사용하기 위해 `Spring Data JPA`와 `Querydsl`이라는 기술을 함께 사용한다. 중요한 것은 `JPA`다. `Spring Data JPA`, `Querydsl`은 `JPA`를 편리하게 사용하도록 도와주는 도구라 생각하면 된다.

이 강의에서는 모든 내용을 다루지 않고, `JPA`와 `Spring Data JPA`, 그리고 `Querydsl`로 이어지는 전체 그림을 볼 것이다. 그리고 이 기술들을 우리 어플리케이션에 적용하면서 자연스럽게 왜 사용해야 되는지, 어떤 장점이 있는지 이해할 수 있게 된다.

#### JPA

- Java Persistence API
- 자바 진영의 ORM 표준 인터페이스

**트랜잭션을 지원하는 쓰기 지연 기능**

트랜잭션을 커밋할 때 까지 INSERT SQL을 모은다. 커밋 시 `JDBC BATCH SQL` 기능을 사용해서 한번에 SQL을 전송한다. 따라서 네트워크 통신 횟수를 줄이고, 성능을 개선한다.

**지연 로딩과 즉시 로딩**

- 지연 로딩: 객체가 실제 사용될 때 로딩
- 즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회

지연 로딩은 객체 그래프를 탐색할 때, 객체의 연관 관계에 있는 객체의 속성에 접근하는 시점에 데이터를 불러오는 방식이다. 지연 로딩의 단점으로는 `team`의 속성에 접근하려면 쿼리가 두 번 실행된다는 단점이 있다.

다음의 그림을 참고하자.

![[jpa-1.png]]

특정 객체를 가져올 때 어플리케이션 내에서 일반적으로 멤버와 팀 데이터를 함께 가져와서 사용해야 한다면 즉시 로딩을 통해 최초의 쿼리 한번으로 데이터를 가져올 수 있다. 말 그대로 JOIN을 걸어서 가져오는 것이다. 다음의 그림을 참고하자.

![[jpa-2.png]]

- 지연 로딩은 쿼리가 두 번 실행된다. 쉽게 말해, 데이터베이스 서버와 두 번의 통신이 발생한다. 이는 자원 낭비로 이어지는 문제가 발생한다.
- 따라서 개발 할 때는 지연 로딩으로 개발하다가, 최적화가 필요한 어느 지점에서는 즉시 로딩으로 변경하여 성능을 개선할 수 있다.

## JPA 설정

`spring-boot-starter-data-jpa` 라이브러리를 사용하면 JPA와 스프링 데이터 JPA를 스프링 부트와 통합하고, 설정도 아주 간단히 할 수 있다.

`build.gradle`에 다음 의존 관계를 추가하고, `spring-boot-starter-jdbc` 라이브러리를 제거한다.

```gradle title="build.gradle"
// JPA
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
// JDBC
//    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
```

`spring-boot-starter-data-jpa`에는 `spring-boot-starter-jdbc` 라이브러리도 함께 포함 되어있다. 따라서 해당 라이브러리의 의존관계를 제거해도 된다.

참고로 `mybatis-spring-boot-starter`도 `spring-boot-starter-jdbc`를 포함한다.

다음으로 `main`과 `test`의 `application.properties`에 다음 설정을 추가한다.

```properties title="application.properties"
#JPA log
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

- `org.hibernate.SQL=DEBUG`: 하이버네이트가 생성하고 실행하는 SQL을 확인할 수 있다.
- `org.hibernate.type.descriptor.sql.BasicBinder=TRACE`: SQL에 바인딩 되는 파라미터를 확인할 수 있다.
- `spring.jpa.show-sql=true`: 참고로 이런 설정도 있다. 이전 설정은 `logger`를 통해서 SQL이 출력된다. 반면 이 설정은 `System.out` 콘솔을 통해서 SQL이 출력된다. 따라서 이 설정은 권장하지 않는다.

## JPA 적용 1 - 개발

JPA에서 가장 중요한 부분은 객체와 테이블을 매핑하는 것이다. JPA가 제공하는 어노테이션을 이용해서 `Item` 객체와 테이블을 매핑해본다.

```java
@Data
@Entity
public class Item {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String itemName;

    @Column
    private Integer price;
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

- `@Entity` 어노테이션은 JPA가 사용하는 객체라는 뜻이다. 이 어노테이션이 있어야 JPA가 인식할 수 있다. 이렇게 `@Entity` 어노테이션이 붙은 객체를 JPA에서는 엔티티라고 한다.
- `@Id` 어노테이션은 테이블의 PK와 해당 필드를 매핑한다.
- `@GeneratedValue(strategy = GenerationType.IDENTITY)`는 PK 생성 전략을 데이터베이스에서 생성하는 `IDENTITY` 방식을 사용한다고 지정하는 옵션이다.
- `@Column` 어노테이션은 객체의 필드를 테이블의 컬럼과 매핑한다.
  - `@Column(name = "XXX")`와 같이, 객체의 필드를 테이블의 특정 컬럼에 매핑 할 수 있다.
  - `@Column(name = "XXX", length = 10)`과 같이 JPA는 매핑 정보로 DDL도 생성할 수 있는데, 그 때 컬럼의 길이 값으로 활용된다.
  - `@Column`을 생략할 경우 필드의 이름을 테이블 컬럼 이름으로 사용한다. 참고로 지금처럼 스프링 부트와 통합해서 사용하면 필드 이름을 테이블 컬럼 명으로 변경할 때 객체 필드의 카멜 케이스를 테이블 컬럼의 언더스코어로 자동 변환한다.
    - 따라서 `itemName` -> `item_name`으로 변경된다.

JPA 엔티티는 `public` 또는 `protected` 접근 제어자가 붙은 기본 생성자가 다음처럼 반드시 존재해야한다.

```java
public Item() {
}
```

이렇게 하면 기본 매핑은 모두 끝난다. 이제 JPA를 실제 사용하는 코드를 작성해본다.

#### JpaItemRepository

```java
@Slf4j
@Repository
@Transactional
@RequiredArgsConstructor
public class JpaItemRepository implements ItemRepository {
    private final EntityManager em;

    @Override
    public Item save(Item item) {
        em.persist(item);

        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item item = em.find(Item.class, itemId);
        item.setItemName(updateParam.getItemName());
        item.setPrice(updateParam.getPrice());
        item.setQuantity(updateParam.getQuantity());
    }

    @Override
    public Optional<Item> findById(Long id) {
        return Optional.ofNullable(em.find(Item.class, id));
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String jpql = "select i from Item i";
        Integer maxPrice = cond.getMaxPrice();
        String itemName = cond.getItemName();
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            jpql += " where";
        }
        boolean andFlag = false;
        if (StringUtils.hasText(itemName)) {
            jpql += " i.itemName like concat('%',:itemName,'%')";
            andFlag = true;
        }
        if (maxPrice != null) {
            if (andFlag) {
                jpql += " and";
            }
            jpql += " i.price <= :maxPrice";
        }
        log.info("jpql={}", jpql);

        TypedQuery<Item> query = em.createQuery(jpql, Item.class);
        if (StringUtils.hasText(itemName)) {
            query.setParameter("itemName", itemName);
        }
        if (maxPrice != null) {
            query.setParameter("maxPrice", maxPrice);
        }
        return query.getResultList();
    }
}
```

`Lombok`으로 `final` 키워드가 사용된 `EntityManager`를 주입 받는 생성자를 생성한다. JPA의 모든 동작은 이 `EntityManager`를 통해서 이루어진다.

엔티티 매니저는 내부에 데이터소스를 가지고 있다. 따라서 데이터베이스에 접근할 수 있다. 그리고 JDBC API를 내부적으로 사용한다.

`@Transactional` 어노테이션을 리포지토리 클래스에 적용했다. JPA의 모든 데이터 변경(등록, 수정, 삭제)은 트랜잭션 안에서 이루어져야 한다. 조회는 트랜잭션이 없어도 가능하다. 그런데 변경의 경우 일반적으로 서비스 계층에서 트랜잭션을 시작하기 때문에 문제가 없다.

하지만 이번 예제에서는 복잡한 비즈니스 로직이 없어서 서비스 계층에서 트랜잭션을 걸지 않았다. JPA에서는 데이터 변경시 트랜잭션이 필수다. 따라서 리포지토리에 트랜잭션을 걸어주었다.

다시 한번 강조하지만 일반적으로는 비즈니스 로직을 시작하는 서비스 계층에 트랜잭션을 걸어주는 것이 맞다.

> 참고로 JPA를 설정하려면 `EntityManagerFactory`, `JpaTransactionManager`, `DataSource`등 다양한 설정을 해야한다. 스프링 부트는 이 과정을 모두 자동화 해준다.

다음으로 JpaConfig을 작성하고 스프링 부트 어플리케이션 설정 클래스로 등록한다.

```java
@Configuration
@RequiredArgsConstructor
public class JpaConfig {
    private final EntityManager em;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JpaItemRepository(em);
    }
}
```

테스트를 실행해보면 모두 정상적으로 패스한 것을 확인할 수 있다.

로그 출력을 살펴보면 JPA가 어떤 SQL을 생성하는지, 파라미터는 무엇이 사용되었는지 확인할 수 있다.

```
select item0_.id as id1_0_0_, item0_.item_name as item_nam2_0_0_, item0_.price as price3_0_0_, item0_.quantity as quantity4_0_0_ from item item0_ where item0_.id=?
binding parameter [1] as [BIGINT] - [18]
```

## JPA 적용 2 - 리포지토리 분석

#### save()

```java
@Override
public Item save(Item item) {
    em.persist(item);
    return item;
}
```

`em.persist(item)`: JPA에서 객체를 테이블에 저장할 때는 엔티티 매니저가 제공하는 `persist()` 메서드를 사용하면 된다.

그러면 JPA는 다음과 같은 SQL을 만들어서 실행한다.

```sql
insert into item (id, item_name, price, quantity) values (default, ?, ?, ?)
```

##### JPA가 만들어서 실행한 SQL

`id`에 값이 빠져있거나, `deafult`로 되어있는 것을 확인할 수 있다. PK 키 생성 전략을 `IDENTITY`로 사용했기 때문에 JPA가 이런 쿼리를 만들어서 실행한 것이다. 물론 쿼리 실행 이후에 `Item` 객체의 `id` 필드에 데이터베이스가 생성한 `PK`가 들어가게 된다. (Mybatis와 유사하게, SQL 실행 이후 생성된 ID를 받아서 객체의 `id`로 설정한다.)

#### update()

```java
@Override
public void update(Long itemId, ItemUpdateDto updateParam) {
    Item item = em.find(Item.class, itemId);
    item.setItemName(updateParam.getItemName());
    item.setPrice(updateParam.getPrice());
    item.setQuantity(updateParam.getQuantity());
}
```

##### JPA가 만들어서 실행한 SQL

```sql
update item set item_name=?, price=?, quantity=? where id=?
```

`update()`를 살펴보면 `em.update()` 같은 메서드를 전혀 호출하지 않았다. 그런데 어떻게 UPDATE SQL이 실행되는 것일까? JPA는 트랜잭션이 커밋되는 시점에, 변경된 엔티티 객체가 있는지 확인한다. 특정 엔티티 객체가 변경된 경우에는 UPDATE SQL을 실행한다.

좀 더 자세하게는, JPA는 처음 조회하는 시점에 원본 객체를 복사해서 내부적으로 스냅샷이라는 것을 가지고 있다. 그 스냅샷과 현재 객체를 트랜잭션 커밋 시점에 비교하고, 바뀐게 있으면 UPDATE SQL을 만들어서 실행한다.

테스트의 경우 마지막에 트랜잭션이 롤백되기 때문에 JPA는 UPDATE SQL을 실행하지 않는다. 따라서 테스트에서 UPDATE SQL을 확인하려면 `@Commit` 어노테이션을 사용하자.

#### findById()

```java
@Override
public Optional<Item> findById(Long id) {
    return Optional.ofNullable(em.find(Item.class, id));
}
```

JPA에서 엔티티 객체를 PK 기준으로 조회할 때 `find()`를 사용하고 조회 타입과 PK 값을 전달하면 된다. 그러면 JPA가 다음과 같은 조회 SQL을 만들어서 실행하고, 결과를 객체로 바로 변환해준다.

##### JPA가 만들어서 실행한 SQL

```sql
select
	item0_.id as id1_0_0_,
	item0_.item_name as item_nam2_0_0_,
	item0_.price as price3_0_0_,
	item0_.quantity as quantity4_0_0_
from item item0_
where item0_.id=?
```

JPA(하이버네이트)가 만들어서 실행한 SQL은 별칭이 조금 복잡하다. 조인이 발생하거나 복잡한 조건에서도 문제 없도록 기계적으로 만들다보니 이런 결과가 나온 듯 하다.

#### findAll()

```java
@Override
public List<Item> findAll(ItemSearchCond cond) {
    String jpql = "select i from Item i";
    // 동적 쿼리는 생략.
    TypedQuery<Item> query = em.createQuery(jpql, Item.class);
    return query.getResultList();
}
```

JPA는 JPQL(Java Persistence Query Language)이라는 객체 지향 쿼리 언어를 제공한다. 주로 여러 데이터를 복잡한 조건으로 조회할 때 사용한다.

SQL이 테이블을 대상으로 한다면, JPQL은 엔티티 객체를 대상으로 SQL을 실행한다고 생각하면 된다. 엔티티 객체를 대상으로 하기 때문에 `FROM` 절 다음 `Item` 엔티티 객체 이름이 들어간다. 엔티티 객체와 속성의 대소문자는 구분해야 한다.

JPQL은 SQL과 문법이 거의 비슷하기 때문에 개발자들이 쉽게 적응할 수 있다. 결과적으로 JPQL을 실행하면 그 안에 포함된 엔티티 객체(예: `Item`)의 매핑 정보를 활용해서 SQL을 만들게 된다.

##### 실행된 JPQL

```sql
select i from Item i
where i.itemName like concat('%',:itemName,'%')
	and i.price <= :maxPrice
```

##### JPQL을 통해 실행된 SQL

```sql
select
	item0_.id as id1_0_,
	item0_.item_name as item_nam2_0_,
	item0_.price as price3_0_,
	item0_.quantity as quantity4_0_
from item item0_
where (item0_.item_name like ('%'||?||'%'))
	and item0_.price<=?
```

JPQL에서 파라미터는 다음과 같이 입력한다.

- `where price <= :maxPrice`
- `query.setParameter("maxPrice", maxPrice)`

##### 동적 쿼리 문제

JPA를 사용해도 동적 쿼리 문제가 남아있다. 동적 쿼리는 뒤에서 설명하는 `Querydsl`이라는 기술을 활용하면 매우 깔끔하게 사용할 수 있다. 실무에서는 동적 쿼리 문제 때문에 JPA를 사용할 때 `Querydsl`도 함께 선택하게 된다.

## JPA 적용 3 - 예외 변환

JPA의 경우 예외가 발생하면 JPA 예외가 발생한다.

```java
@Repository
@Transactional
public class JpaItemRepository implements ItemRepository {
	private final EntityManager em;

	@Override
	public Item save(Item item) {
		em.persist(item);
		return item;
	}
}
```

- 이 리포지토리에서 주입 받은 `EntityManager`는 순수한 JPA 기술이고, 스프링과는 관계가 없다. 따라서 엔티티 매니저는 예외가 발생하면 JPA 관련 예외를 발생시킨다.
- JPA는 `PersistenceException`과 그 하위 예외를 발생 시킨다.
  - 추가로 JPA는 `IllegalStateException`, `IllegalArgumentException`을 발생시킬 수 있다.
- 그렇다면 JPA 예외를 스프링 예외 추상화(`DataAccessException`)으로 어떻게 변환할 수 있을까?
- 비밀은 바로 `@Repository` 어노테이션에 있다.

##### 예외 변환 전

![[jpa-3.png]]

**@Repository의 기능**

- `@Repository`가 붙은 클래스는 컴포넌트 스캔의 대상이 된다.
- `@Repository`가 붙은 클래스는 예외 변환 AOP의 적용 대상이 된다.
  - 스프링과 JPA를 함께 사용하는 경우 스프링은 JPA 예외 변환기(`PersistenceExceptionTranslator`)를 등록한다.
  - 예외 변환 AOP 프록시는 JPA 관련 예외가 발생하면 JPA 예외 변환기를 통해 발생한 예외를 스프링 데이터 접근 예외로 변환한다.

##### 예외 변환 후

![[jpa-4.png]]

결과적으로 리포지토리에 `@Repository` 어노테이션만 있으면 스프링이 예외 변환을 처리하는 AOP를 만들어준다.

> 스프링 부트는 `PersistenceExceptionTranslationPostProcessor`를 자동으로 등록하는데, 여기에서 `@Repository`를 AOP 프록시로 만드는 어드바이저가 등록된다.

아마 이런 프록시 패턴을 사용하기 위해서 `public` 또는 `protected` 접근 제어자가 붙은 생성자가 필요한 것 같다.

---

References: 김영한의 스프링 DB 2편

Links to this page:
