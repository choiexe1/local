---
title: 
tags:
  - java
  - programming
  - spring
  - database
  - querydsl
publish: true
date: 2024-12-24 00:01
---

## Querydsl

- 쿼리에 특화된 프로그래밍 언어라고 볼 수 있음
- 단순, 간결, 유창
- 다양한 저장소 쿼리 기능 통합
- 타입 세이프

`Querydsl`은 쿼리를 `Type-safe`하게 작성할 수 있게 지원하는 프레임워크다. 주로 JPA 쿼리에 사용된다.

그럼 쿼리의 타입 안정성은 도대체 무엇일까? 쿼리를 작성하게 되면 일반적으로 문자열로 작성하게 된다. 문자열로 작성하게 되는 쿼리엔 여러 문제점이 있는데 당장 떠오르는 문제들만 나열해보면 다음과 같다.

- 컴파일 타임에 문자열에 대한 문제를 인지할 수 없다.
  - [[자바의 메모리 구조]]에 따르면 런타임에 메서드 영역인 상수 문자열 풀에 `new String()`으로 불변 문자열 객체가 생성되고 해당 객체를 런타임에 관리하기 때문이다.
- 정의한 테이블(스키마)의 필드를 모두 외울 수 없다. 따라서 개발자는 개발 시에 정의된 스키마를 참고해가며 개발해야 한다.
- IDE의 코드 어시스턴스 기능을 활용할 수 없다.
- 특정 조건이 존재하는 쿼리, 즉 동적 쿼리를 생성하려면 문자열 더하기 연산과 조건문을 통해 작성해야 한다. 이는 개발 뿐만 아니라 가독성을 해치고 유지보수를 힘들게 한다.

찾아보면 더 많은 문제들이 있겠지만, 어쨌든 `Querydsl`은 이 문제들을 모두 해결해준다.

## Querydsl 설정

`build.gradle`에 다음과 같이 작성한다.

```gradle title="build.gradle"

dependencies {
// Querydsl 추가
implementation 'com.querydsl:querydsl-jpa'
annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jpa"
annotationProcessor "jakarta.annotation:jakarta.annotation-api"
annotationProcessor "jakarta.persistence:jakarta.persistence-api"
}

//Querydsl 추가, 자동 생성된 Q클래스 gradle clean으로 제거
clean {
	delete file('src/main/generated')
}
```

그리고 프로젝트의 `gradle task`를 통해 `clean` 작업을 실행하고, 다시 `compileJava`를 통해 컴파일하면 다음과 같이 `QType`이 생성된다.

> [!warning] 프로젝트 빌드 설정마다 다른 QType 생성 경로 (Gradle VS Intellij IDEA)
> 프로젝트를 빌드 할 때, gradle을 사용하면 annotationProcessor가 자동으로 생성하는 QType의 경로가 프로젝트 루트의 build/generated/sources/annotationProcessor/java/main 하위에 생성된다.
>
> IntelliJ IDEA를 사용하면 src/main/generated 하위에 QType이 생성된다.

```java
/**
 * QItem is a Querydsl query type for Item */@Generated("com.querydsl.codegen.DefaultEntitySerializer")
public class QItem extends EntityPathBase<Item> {

    private static final long serialVersionUID = -570080939L;

    public static final QItem item = new QItem("item");

    public final NumberPath<Long> id = createNumber("id", Long.class);

    public final StringPath itemName = createString("itemName");

    public final NumberPath<Integer> price = createNumber("price", Integer.class);

    public final NumberPath<Integer> quantity = createNumber("quantity", Integer.class);

    public QItem(String variable) {
        super(Item.class, forVariable(variable));
    }

    public QItem(Path<? extends Item> path) {
        super(path.getType(), path.getMetadata());
    }

    public QItem(PathMetadata metadata) {
        super(Item.class, metadata);
    }

}
```

> 참고로 `QType`은 컴파일 시점에 자동으로 생성되므로 버전관리에 포함하지 않는 것이 좋다. (.gitignore)
>
> 또, 만약 Gradle을 통해 프로젝트를 빌드한다면 `build.gradle`의 `clean` 작업을 지워도 상관없다.

설정은 끝났으므로 다음과 같이 `Querydsl`을 이용한 `JpaItemRepositoryV3`를 작성한다.

```java
@Repository
@Transactional
public class JpaItemRepositoryV3 implements ItemRepository {
    private final EntityManager em;
    private final JPQLQueryFactory query;

    public JpaItemRepositoryV3(EntityManager em) {
        this.em = em;
        this.query = new JPAQueryFactory(em);
    }

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
        Item item = em.find(Item.class, id);
        return Optional.ofNullable(item);
    }

	@Override
	public List<Item> findAll(ItemSearchCond cond) {
	    String itemName = cond.getItemName();
	    Integer maxPrice = cond.getMaxPrice();

	    return query
	            .select(item)
	            .from(item)
	            .where(likeItemName(itemName), maxPrice(maxPrice))
	            .fetch();
	}

	private BooleanExpression likeItemName(String itemName) {
	    if (StringUtils.hasText(itemName)) {
	        return item.itemName.like("%" + itemName + "%");
	    }

	    return null;
	}

	private BooleanExpression maxPrice(Integer maxPrice) {
	    if (maxPrice != null) {
	        return item.price.loe(maxPrice);
	    }

	    return null;
	}
}
```

- `Querydsl`을 사용하려면 `JPAQueryFactory`가 필요하다. `JPAQueryFactory`는 `JPQL`을 만들기 때문에, `EntityManager`가 필요하다.
- 설정 방식은 `JdbcTemplate`를 설정하는 것과 유사하다.

**findAll**

다음 코드는 SQL에 대해 알고 있다면 누가 봐도 쉽게 이해할 수 있을 것이다.

```java
@Override
public List<Item> findAll(ItemSearchCond cond) {
    String itemName = cond.getItemName();
    Integer maxPrice = cond.getMaxPrice();

    return query
            .select(item)
            .from(item)
            .where(likeItemName(itemName), maxPrice(maxPrice))
            .fetch();
}

private BooleanExpression likeItemName(String itemName) {
    if (StringUtils.hasText(itemName)) {
        return item.itemName.like("%" + itemName + "%");
    }
    return null;
}

private BooleanExpression maxPrice(Integer maxPrice) {
    if (maxPrice != null) {
        return item.price.loe(maxPrice);
    }
    return null;
}
```

- `Querydsl`에서 `where(A, B)`에 다양한 조건들을 직접 넣을 수 있는데, 이렇게 넣으면 AND 조건으로 처리된다. 참고로 `where`에 `null`을 입력하면 해당 조건은 무시한다.
- 이 코드의 또 다른 장점은 `likeItemName()`, `maxPrice()`같은 `BooleanExpression`을 반환하는 동적 쿼리문을 다른 쿼리를 사용할 때 재사용할 수 있게 된다는 점이다. 이처럼 쿼리 조건을 부분적으로 모듈화 할 수 있다.

#### 정리

- `Querydsl`을 사용하면 동적 쿼리를 매우 깔끔하게 작성할 수 있다.
- 쿼리에 오타나 문제가 있어도 컴파일 시점에 오류를 막을 수 있다.
- 메서드 추출을 통해서 부분적으로 코드를 재사용할 수 있다. (예: `likeItemName` 메서드)

---

References: 김영한의 스프링 DB 2편

Links to this page:
