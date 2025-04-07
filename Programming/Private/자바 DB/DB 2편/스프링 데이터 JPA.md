---
title: 
tags:
  - java
  - programming
  - spring
  - database
  - jpa
  - spring-data-jpa
publish: true
date: 2024-12-24 00:00
---

## 스프링 데이터 JPA 주요 기능

스프링 데이터 JPA는 JPA를 편리하게 사용할 수 있도록 도와주는 라이브러리이다. 수많은 편리 기능을 제공하지만 가장 대표적인 기능은 다음과 같다.

- 공통 인터페이스 기능
- 쿼리 메서드 기능

![[spring-data-jpa-1.png]]

- `JpaRepository` 인터페이스를 통해서 기본적인 CRUD 기능을 제공한다.
- 공통화 가능한 기능이 거의 모두 포함되어 있다.

**JpaRepository 사용법**

- 다음과 같이 `JpaRepository` 인터페이스를 상속 받고, 제네릭에 관리할 엔티티, 엔티티의 ID를 주면 된다.
- 그러면 `JpaRepository`가 제공하는 기본 CRUD 기능을 모두 사용할 수 있다.

```java
public interface ItemRepository extends JpaRepository<Item, Long> {}
```

**스프링 데이터 JPA가 구현 클래스를 대신 생성**

![[spring-data-jpa-2.png]]

- `JpaRepository` 인터페이스만 상속 받으면, 스프링 데이터 JPA가 동적 프록시 기술을 사용해서 구현 클래스를 만들어준다. 그리고 만든 구현 클래스의 인스턴스를 만들어서 스프링 빈으로 등록한다.
- 따라서 개발자는 따로 구현 클래스 없이도 인터페이스만 만들면 기본 CRUD 기능을 사용할 수 있다.

**쿼리 메서드 기능**

스프링 데이터 JPA는 인터페이스에 메서드만 적어두면, 메서드 이름을 분석해서 쿼리를 자동으로 만들고 실행해주는 기능을 제공한다.

```java title="순수 JPA 리포지토리"
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
	return em.createQuery("select m from Member m where m.username = :username and m.age > :age")
			.setParameter("username", username)
			.setParameter("age", age).getResultList();
}
```

순수 JPA를 사용하면 직접 JPQL을 작성하고, 파라미터도 직접 바인딩 해야한다. 그런데 스프링 데이터 JPA를 사용하면 메서드의 이름을 분석해서 필요한 JPQL을 만들고 실행해준다. 물론 JPQL은 JPA가 SQL로 번역해서 실행한다.

물론 아무 이름이나 사용해도 되는 것은 아니고, 다음과 같은 규칙을 따라야 한다.

- 조회의 경우: `find..By`, `read..By`, `query..By`, `get..By`
- Count: `count..By` 반환 타입 `long`
- EXISTS: `exists..By` 반환 타입 `boolean`
- 삭제: `delete..By`, `remove..By` 반환 타입 `long`
- DISTINCT: `findDistinct`, `findMemberDistinctBy`
- LIMIT: `findFirst3`, `findFirst`, `findTop`, `findTop3`

**JPQL 직접 사용하기**

```java
public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long> {
	//쿼리 메서드 기능
	List<Item> findByItemNameLike(String itemName);

	@Query("select i from Item i where i.itemName like :itemName and i.price <= : price")
	List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);
	//쿼리 직접 실행

}
```

- 쿼리 메서드 기능 대신에 직접 `JPQL`을 사용하고 싶을 때는 `@Query`와 함께 JPQL을 작성하면 된다. 이때는 메서드 이름으로 실행하는 규칙은 무시된다.
- 참고로 스프링 데이터 JPA는 `JPQL` 뿐만 아니라 JPA의 네이티브 쿼리 기능도 지원한다. `JPQL` 대신에 `SQL`을 직접 작성할 수 있다.

> [!summary] 스프링 데이터 JPA
> 스프링 데이터 JPA는 JPA를 편리하게 사용하도록 도와주는 도구이다. 따라서 JPA 자체를 잘 이해하는 것이 가장 중요하다.

## 스프링 데이터 JPA 적용 1

스프링 데이터 JPA는 `spring-boot-starter-data-jpa` 라이브러리를 넣어주면 된다.

```gradle title="build.gradle"
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

```java
public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long> {
    List<Item> findByItemNameLike(String itemName);

    List<Item> findByPriceLessThanEqual(Integer price);


    // 쿼리 메서드 (아래 메서드와 같은 기능 수행)
    List<Item> findByItemNameLikeAndPriceLessThanEqual(String itemName, Integer price);

    // 쿼리 직접 실행
    @Query("SELECT i FROM Item i where i.itemName like :itemName and i.price <= :price")
    List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);
}
```

- 스프링 데이터 JPA가 제공하는 `JpaRepository` 인터페이스를 인터페이스 상속 받으면 기본적인 CRUD 기능을 사용할 수 있다.
- 그런데 이름으로 검색하거나, 가격으로 검색하는 기능은 공통으로 제공할 수 있는 기능이 아니다. 따라서 쿼리 메서드 기능을 사용하거나 `@Query`를 사용해서 직접 쿼리를 실행하면 된다.

여기서는 데이터를 조건에 따라 4가지로 분류해서 검색한다.

- 모든 데이터 조회 (JpaRepository를 상속 받으면 기본 제공)
- 이름 조회
- 가격 조회
- 이름 + 가격 조회

동적 쿼리를 사용하면 좋겠지만, 기본적으로 JPA 기반이므로 스프링 데이터 JPA는 동적 쿼리에 약하다. 따라서 이 문제는 이후에 `Querydsl`을 사용해보면서 동적 쿼리로 깔끔하게 해결해본다.

참고로 스프링 데이터 JPA도 `Example`이라는 기능으로 약간의 동적 쿼리 기능을 지원하지만 실무에서 사용하기는 기능이 빈약하다. 실무에서 `JPQL` 동적 쿼리는 `Querydsl`을 사용하는 것이 좋다.

## 스프링 데이터 JPA 적용 2

```java title="ItemServiceV1.java"
@Service
@RequiredArgsConstructor
public class ItemServiceV1 implements ItemService {

    private final ItemRepository itemRepository;

    @Override
    public Item save(Item item) {
        return itemRepository.save(item);
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        itemRepository.update(itemId, updateParam);
    }

    @Override
    public Optional<Item> findById(Long id) {
        return itemRepository.findById(id);
    }

    @Override
    public List<Item> findItems(ItemSearchCond cond) {
        return itemRepository.findAll(cond);
    }
}
```

- `ItemService`는 `ItemRepository`에 의존하기 때문에 `ItemService`에서 앞서 구현한 `SpringDataJpaItemRepository`를 그대로 사용할 수 없다.
- 물론 `ItemService`가 `SpringDataJpaItemRepository`를 직접 사용하도록 고치면 가능하겠지만, 그렇게 되면 서비스 레이어가 특정 구현 기술에 의존하게 된다.
- `ItemService`의 변경 없이 `ItemService`가 `ItemRepository`에 대한 의존을 유지하면서 DI를 통해 구현 기술을 변경하고 싶다.

따라서 다음과 같이 `SpringDataJpaItemRepository`를 내부에서 직접 사용하면서 `ItemRepository`를 구현하는 클래스를 작성한다. 이 클래스는 `ItemRepository`로 사용될 수 있으면서도 서비스 계층에서의 변경이 발생하지 않도록 어댑터 역할을 해주는 셈이다.

```java
@Repository
@Transactional
@RequiredArgsConstructor
public class JpaItemRepositoryV2 implements ItemRepository {
    private final SpringDataJpaItemRepository repository;

    @Override
    public Item save(Item item) {
        return repository.save(item);
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = repository.findById(itemId).orElseThrow();
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    @Override
    public Optional<Item> findById(Long id) {
        return repository.findById(id);
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        if (StringUtils.hasText(itemName) && maxPrice != null) {
            return repository.findItems("%" + itemName + "%", maxPrice);
        } else if (StringUtils.hasText(itemName)) {
            return repository.findByItemNameLike("%" + itemName + "%");
        } else if (maxPrice != null) {
            return repository.findByPriceLessThanEqual(maxPrice);
        } else {
            return repository.findAll();
        }
    }
}
```

#### 런타임 의존 관계

```mermaid
graph LR

ItemService --> jpaItemRepositoryV2 --> ProxySpringDataJPA
```

- `ProxySpringDataJPA`는 동적 프록시로 생성된 `SpringDataJpaItemRepository`의 구현체이다.
- 이렇게 `JpaItemRepository`가 중간에서 어댑터 역할을 해준 덕분에 `ItemService`가 사용하는 `ItemRepository` 인터페이스를 그대로 유지할 수 있게 됐고, 클라이언트인 `ItemService`의 코드를 변경하지 않아도 된다.

#### 기능 분석

`update()`: 앞서 JPA 강의에서 학습했듯이, 트랜잭션이 커밋 될 때 변경 내용이 자동으로 데이터베이스에 반영된다.

`findAll()`: 모든 조건에 부합할 때는 `findByNameLikeAndPriceLessThanEqual()`을 사용해도 되고, `repository.findItems()`를 사용해도 된다. 그런데 보는 것 처럼 조건이 2개만 되어도 이름이 너무 길어지는 단점이 있다. 따라서 스프링 JPA가 제공하는 메서드 이름으로 쿼리를 자동으로 만들어주는 기능과 `@Query`로 직접 작성하는 기능 중에 적절한 선택이 필요하다.

추가로 코드를 살펴보면 동적 쿼리가 아니라, 상황에 따라 각각 스프링 데이터 JPA의 메서드를 호출해서 상당히 비효율적이다. 스프링 데이터 JPA는 동적 쿼리 기능에 대한 지원이 매우 약하다.

#### SpringDataJpaConfig

```java
@Configuration
@RequiredArgsConstructor
public class SpringDataJpaConfig {
    private final SpringDataJpaItemRepository springDataJpaItemRepository;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JpaItemRepositoryV2(springDataJpaItemRepository);
    }
}
```

해당 컨픽을 적용하고 테스트를 실행해보면 정상적으로 작동한다.

> [!warning] Hibernate 5.6.6 ~ 5.6.7
> 하이버네이트 5.6.6 버전 또는 5.6.7 버전을 사용하면 Like 문을 사용할 때 다음 예외가 발생한다.
> `java.lang.IllegalArgumentException: Parameter value [\] did not match expected type [java.lang.String (n/a)]`
>
> 강의 예제로 제공한 스프링 부트 프로젝트는 스프링 부트 2.6.5 버전을 사용하는데, 이 버전은 하이버네이트 5.6.7을 사용한다. 따라서 다운그레이드를 통해서 버그가 없는 5.6.5.Final로 변경해야한다.

#### 정리

스프링 데이터 JPA의 대표적인 기능을 알아보았는데, 스프링 데이터 JPA는 이 외에도 수 많은 편리 기능을 제공한다. 최근 몇일 간 여태 학습했던 내용을 복습하며 작은 프로젝트를 진행했는데 그 과정에서 만났던 귀찮은 포인트들을 모두 해결해준다. 대표적으로 페이징 처리를 위한 기능도 제공한다.

또 스프링 데이터 JPA는 많은 개발자들이 똑같은 코드로 중복 개발하는 부분, 예를 들면 공통적으로 엔티티의 총 갯수를 카운팅하거나 고유한 속성을 통해 데이터를 가져오는 등의 기능들을 공통 인터페이스와 동적 프록시 기술로 편리하게 개발 할 수 있도록 해준다.

---

References: 김영한의 스프링 DB 2편

Links to this page:
