---
title: 
tags:
  - java
  - programming
  - spring
  - jdbc
  - database
publish: true
date: 2024-12-15 00:00
---

## JdbcTemplate 소개와 설정

SQL을 직접 사용하는 경우에 스프링이 제공하는 `JdbcTemplate`는 아주 좋은 선택지다. `JdbcTemplate`는 `JDBC`를 매우 편리하게 사용할 수 있게 도와준다.

**설정의 편리함**

- `JdbcTemplate`은 `spring-jdbc` 라이브러리에 포함되어 있는데, 이 라이브러리는 스프링으로 `JDBC`를 사용할 때 기본으로 사용되는 라이브러리이다. 그리고 별도의 복잡한 설정 없이 바로 사용할 수 있다.

**반복 문제 해결**

- `JdbcTemplate`은 템플릿 콜백 패턴을 사용해서, `JDBC`를 직접 사용할 때 발생하는 대부분의 반복 작업을 대신 처리해준다.
- 개발자는 SQL을 작성하고, 전달할 파라미터를 정의하고, 응답 값을 매핑하기만 하면 된다.
- 이 부분은 앞서 [[JdbcTemplate]]에서 학습했다.

**단점**

- 동적 SQL을 해결하기 어렵다.

## JdbcTemplate 설정

```gradle title="build.gradle"
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-jdbc'
	runtimeOnly 'com.h2database:h2'
}
```

위의 두 의존성을 프로젝트에 추가한다. 여기서는 H2 데이터베이스에 접속해야 하기 때문에 H2 데이터베이스의 클라이언트 라이브러리도 추가했다.

## JdbcTemplate 적용 1 - 기본

이제부터 본격적으로 메모리에 사용하던 데이터를 데이터베이스에 저장해본다.

`ItemRepository` 인터페이스가 이미 작성되어 있으므로 해당 인터페이스를 기반으로 `JdbcTemplate`을 사용하는 새로운 구현체를 개발한다.

```java
public class JdbcTemplateItemRepositoryV1 implements ItemRepository {
    private final JdbcTemplate template;

    public JdbcTemplateItemRepositoryV1(DataSource dataSource) {
        this.template = new JdbcTemplate(dataSource);
    }
```

먼저 `JdbcTemplate`를 사용하려면 `JdbcTemplate`에 데이터 소스를 주입 해야한다. 당연한 소리지만 데이터소스를 기반으로 해당 데이터베이스에 대한 커넥션을 획득해야 데이터베이스와 관련된 작업을 수행할 수 있기 때문이다.

이 데이터소스를 수동으로 직접 주입 받아서 생성자에서 `new JdbcTemplate(dataSource)`를 통해 `JdbcTemplate` 객체를 생성한다.

```java
@Override
public Item save(Item item) {
    String sql = "INSERT INTO item (item_name, price, quantity) VALUES (?, ?, ?)";
    KeyHolder keyHolder = new GeneratedKeyHolder();
    template.update(connection -> {
        // 자동 증가 키
        PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
        ps.setString(1, item.getItemName());
        ps.setInt(2, item.getPrice());
        ps.setInt(3, item.getQuantity());
        return ps;
    }, keyHolder);

    long key = keyHolder.getKey().longValue();
    item.setId(key);

    return item;
}
```

앞서 메모리 기반으로 어플리케이션이 동작할 때는 직접 `Item` 객체의 `id`를 증가시키면서 설정했지만 이제는 기본 키 생성을 H2 데이터베이스에 위임하고 있다.

따라서 PK인 `id` 값을 개발자가 직접 지정하는 것이 아니라 비워두고 저장해야 한다. 문제는 이렇게 데이터베이스가 대신 생성해주는 PK `id` 값은 데이터베이스가 생성하기 때문에, 데이터베이스에 `INSERT`가 완료 되어야 생성된 PK를 확인할 수 있다.

`KeyHolder`와 `PreparedStatement ps = connection.prepareStatement(sql, new String[]`를 사용해서 `id`를 지정해주면 `INSERT` 쿼리 실행 이후에 데이터베이스에서 생성된 `id` 값을 조회할 수 있다.

> 뒤에서 `JdbcTemplate`이 제공하는 `SimpleJdbcInsert`라는 훨씬 편리한 기능이 있으므로 대략 이렇게 사용한다 정도로만 알아두면 된다.

```java
@Override
public void update(Long itemId, ItemUpdateDto updateParam) {
    String sql = "UPDATE item SET item_name = ?, price = ?, quantity = ? WHERE id = ?";

    template.update(sql,
            updateParam.getItemName(),
            updateParam.getPrice(),
            updateParam.getQuantity(),
            itemId
    );
}
```

기존 아이템을 변경하는 `update`는 위와 같이 단순하다. SQL을 넘기고 파라미터를 입력한다.

```java
@Override
public Optional<Item> findById(Long id) {
    String sql = "SELECT id, item_name, price, quantity FROM item WHERE id = ?";
    try {
        Item item = template.queryForObject(sql, itemRowMapper(), id);
        return Optional.of(item);
    } catch (EmptyResultDataAccessException e) {
        return Optional.empty();
    }
}
```

`template.queryForObject()` SQL의 결과 로우가 하나일 때 사용한다. 여기서 반환되는 로우를 `itemRowMapper()`라는 콜백 함수를 이용해서 자바 객체로 매핑할 수 있다.

- `template.queryForObject`는 결과가 없으면 `EmptyResultDataAccessException`이 발생한다.
- 결과 로우가 하나가 아니라 둘 이상이면 `IncorrectResultSizeDataAccessException` 예외가 발생한다.

`ItemRepository.findById()` 인터페이스는 결과가 없을 때 `Optional`을 반환해야 한다. 따라서 결과가 없으면 예외를 잡아서 `Optional.empty`를 대신 반환하면 된다.

```java
@Override
public List<Item> findAll(ItemSearchCond cond) {
    String itemName = cond.getItemName();
    Integer maxPrice = cond.getMaxPrice();
    String sql = "select id, item_name, price, quantity from item";

    //동적 쿼리
    if (StringUtils.hasText(itemName) || maxPrice != null) {
        sql += " where";
    }
    boolean andFlag = false;
    List<Object> param = new ArrayList<>();
    if (StringUtils.hasText(itemName)) {
        sql += " item_name like concat('%',?,'%')";
        param.add(itemName);
        andFlag = true;
    }
    if (maxPrice != null) {
        if (andFlag) {
            sql += " and";
        }
        sql += " price <= ?";
        param.add(maxPrice);
    }
    log.info("sql={}", sql);
    return template.query(sql, itemRowMapper(), param.toArray());
}
```

위 코드를 살펴보면 동적으로 쿼리를 생성해야 하기 때문에 각 파라미터의 조건에 따라서 문자열을 추가하거나 빼고 있다는 것을 알 수 있다.

첫 회사에서도 express로 이런 비슷한 쿼리를 자주 작성 했었다.

`template.query`는 결과가 하나 이상 일때 사용한다. 여기서 사용되는 `itemRowMapper()`는 `JdbcTemplate`가 다음과 같은 루프를 돌려준다는 점이다. 따라서 개발자는 `RowMapper`만 구현하면 된다.

```java
while(resultSet 이 끝날 때 까지) {
	rowMapper(rs, rowNum)
}
```

## JdbcTemplate 적용 - 동적 쿼리 문제

결과를 검색하는 `findAll()`에서 어려운 부분은, 사용자가 검색하는 값에 따라서 실행하는 SQL이 동적으로 달라져야 한다는 점이다. 예를 들어서 다음과 같은 케이스들이 존재한다.

- 검색 조건이 없음
- 상품명으로 검색
- 최대 가격으로 검색
- 상품명과 최대 가격으로 검색

각 파라미터의 유무에 따라 실제 전달되는 SQL이 달라져야 한다. 각 케이스에 따른 SQL은 다음과 같다.

```sql
-- 검색 조건이 없음
SELECT id, item_name, price, quantity
FROM item;

-- 상품명(itemName)으로 검색
SELECT id, item_name, price, quantity
FROM item
WHERE item_name LIKE CONCAT('%', ?, '%');

-- 최대 가격(maxPrice)로 검색
SELECT id, item_name, price, quantity
FROM item
WHERE price <= ?;

-- 상품명(itemName), 최대 가격(maxPrice) 둘 다 검색
SELECT id, item_name, price, quantity
FROM item
WHERE item_name LIKE CONCAT('%', ?, '%') AND price <= ?;
```

따라서 개발자는 파라미터의 유무를 확인하고 위의 네 가지 케이스에 대한 SQL 문자열 생성을 모두 개발해야하는 것이다.

단순히 숫자만 계산해봐도 조건이 N개 있을 때 조합 가능한 경우의 수는 2의 N승이다. 조건이 4개만 되어도 16가지 케이스의 동적 문자열 생성 로직을 개발해야 하는 것이다.

물론 영한님이 동적 쿼리 생성하는 부분의 로직을 너무 오버해서 작성한 감이 있긴 하지만.. 어쨌든 개발 하는것 뿐만이 문제가 아니라 유지보수 하기가 복잡해진다.

## JdbcTemplate 적용 3 - 구성과 실행

먼저 다음과 같이 수동으로 빈을 등록한다.

```java
@Configuration
@RequiredArgsConstructor
public class JdbcTemplateV1Configure {
    private final DataSource dataSource;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JdbcTemplateItemRepositoryV1(dataSource);
    }
}
```

여기서 우리는 `DataSource`를 직접 생성하지 않고 스프링 부트가 생성하는 `DataSource`를 사용한다. 따라서 `application.properties`에 다음과 같이 데이터베이스 관련 설정을 해주어야 한다.

```properties title="application.properties"
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa
spring.datasource.password=
```

이렇게 설정만 하면 스프링 부트가 해당 설정을 사용해서 커넥션 풀과 `DataSource`, 트랜잭션 매니저를 스프링 빈으로 자동 등록한다. 이 내용은 [[스프링 부트의 자동 리소스 등록|스프링 부트의 자동 리소스 등록]]에서 확인할 수 있다.

다음으로, 현재 어플리케이션은 `MemoryConfig` 설정을 `@Import` 어노테이션으로 지정하여 사용하고 있기 때문에 해당 부분을 `JdbcTemplateV1Configure`로 변경한다.

```java
@Import(JdbcTemplateV1Configure.class)
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")
public class ItemServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ItemServiceApplication.class, args);
    }

    @Bean
    @Profile("local")
    public TestDataInit testDataInit(ItemRepository itemRepository) {
        return new TestDataInit(itemRepository);
    }
}
```

어플리케이션을 동작해보면 잘 작동하는 것을 확인할 수 있다. 만약 `JdbcTemplate`가 실행하는 SQL 로그를 확인하려면 `application.properties`에 다음을 추가하면 된다.

```properties title="application.properties"
logging.level.org.springframework.jdbc=debug
```

결과는 다음과 같이 출력된다.

```
2024-12-15 00:43:31.994 DEBUG 14843 --- [nio-8080-exec-1] Executing prepared SQL query
2024-12-15 00:43:31.994 DEBUG 14843 --- [nio-8080-exec-1] Executing prepared SQL statement [select id, item_name, price, quantity from item]
```

## JdbcTemplate - 이름 지정 파라미터 1

`JdbcTemplate`를 기본으로 사용하면 파라미터를 순서대로 바인딩 한다.

예를 들어서 다음 코드를 살펴보자.

```java
String sql = "update item set item_name=?, price=?, quantity=? where id=?";
template.update(sql,
		itemName,
		price,
		quantity,
		itemId);
```

여기서는 `itemName`, `price`, `quantity`가 SQL에 있는 `?` 에 순서대로 바인딩 된다. 따라서 순서만 잘 지키면 문제가 될 것은 없다. 그런데 문제는 변경시점에 발생한다.

누군가 SQL 코드의 순서를 변경했고 실수로 파라미터 순서는 변경하지 않으면 SQL에 잘못된 파라미터가 다음과 같이 바인딩 될 수 있다.

```
item_name=itemName, quantity=price, price=quantity
```

결과적으로 `price`와 `quantity`가 바뀌는 매우 심각한 문제가 발생한다. 이럴 일이 없을 것 같지만 실무에서는 파라미터가 10~20개가 넘어가는 일도 아주 많이 발생한다.

미래에 필드를 추가하거나, 수정하면서 이런 문제가 충분히 발생할 수 있다.

버그 중에서 가장 고치기 힘든 버그는 데이터베이스에 데이터가 잘못 들어가는 버그다. 이것은 코드만 고치는 수준이 아니라 데이터베이스의 데이터를 복구해야 하기 때문에 버그를 해결하는데 들어가는 리소스가 어마어마하다.

**개발을 할 때는 코드를 몇 줄 줄이는 편리함도 중요하지만, 모호함을 제거해서 코드를 명확하게 만드는 것이 유지보수 관점에서 매우 중요하다.**

이처럼 파라미터를 순서대로 바인딩 하는 것은 편리하기는 하지만, 순서가 맞지 않아서 버그가 발생할 수도 있으므로 주의해서 사용해야 한다.

## 이름 지정 바인딩

`JdbcTemplate`는 이런 문제를 보완하기 위해 `NamedParameterJdbcTemplate`라는 이름을 지정해서 파라미터를 바인딩 하는 기능을 제공한다.

```java
/**
 * NamedParameterJdbcTemplate
 * SqlParameterSource
 * - BeanPropertySqlParameterSource
 * - MapSqlParameterSource
 * Map
 *
 * BeanPropertyRowMapper
 */
@Slf4j
@RequiredArgsConstructor
public class JdbcTemplateItemRepositoryV2 implements ItemRepository {
    private final NamedParameterJdbcTemplate template;

    public JdbcTemplateItemRepositoryV2(DataSource dataSource) {
        this.template = new NamedParameterJdbcTemplate(dataSource);
    }

    @Override
    public Item save(Item item) {
        String sql = "INSERT INTO item (item_name, price, quantity) VALUES (:itemName, :price, :quantity)";
        KeyHolder keyHolder = new GeneratedKeyHolder();

        SqlParameterSource param = new BeanPropertySqlParameterSource(item);
        template.update(sql, param, keyHolder);

        long key = keyHolder.getKey().longValue();
        item.setId(key);

        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        String sql = "UPDATE item SET item_name = :itemName, price = :price, quantity = :quantity WHERE id = :id";

        SqlParameterSource param = new MapSqlParameterSource()
                .addValue("itemName", updateParam.getItemName())
                .addValue("price", updateParam.getPrice())
                .addValue("quantity", updateParam.getQuantity())
                .addValue("id", itemId);

        template.update(sql, param);
    }

    @Override
    public Optional<Item> findById(Long id) {
        String sql = "SELECT id, item_name, price, quantity FROM item WHERE id = :id";

        try {
            Map<String, Long> param = Map.of("id", id);
            Item item = template.queryForObject(sql, param, itemRowMapper());
            return Optional.of(item);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();
        String sql = "select id, item_name, price, quantity from item";

        SqlParameterSource param = new BeanPropertySqlParameterSource(cond);

        //동적 쿼리
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            sql += " where";
        }
        boolean andFlag = false;
        if (StringUtils.hasText(itemName)) {
            sql += " item_name like concat('%',:itemName,'%')";
            andFlag = true;
        }
        if (maxPrice != null) {
            if (andFlag) {
                sql += " and";
            }
            sql += " price <= :maxPrice";
        }
        log.info("sql={}", sql);

        return template.query(sql, param, itemRowMapper());
    }

    private RowMapper<Item> itemRowMapper() {
        return BeanPropertyRowMapper.newInstance(Item.class);
    }
}
```

- `JdbcTemplateItemRepositoryV2`는 `ItemRepository` 인터페이스를 구현했다.
- `this.template = new NamedParameterJdbcTemplate(datasource)`
  - `NamedParameterJdbcTemplate`도 내부에 `dataSource`가 필요하다.
  - `JdbcTemplateItemRepositoryV2` 생성자를 보면 의존관계 주입은 `dataSource`를 받고 내부에서 `NamedParameterJdbcTemplate`를 생성해서 가지고 있다. 스프링에서는 `JdbcTemplate` 관련 기능을 사용할 때 관례상 이 방법을 많이 사용한다.
  - 물론 `NamedParameterJdbcTemplate`을 스프링 빈으로 직접 등록하고 주입 받아도 된다.

`save()`를 살펴보면 SQL에서 다음과 같이 `?` 대신에 `:파라미터이름`을 받는 것을 확인할 수 있다.

```java
"INSERT INTO item (item_name, price, quantity) VALUES (:itemName, :price, :quantity)"
```

추가로 `NamedParameterJdbcTemplate`은 데이터베이스가 생성해주는 키를 매우 쉽게 조회하는 기능도 제공해준다.

그리고 `BeanPropertyRowMapper.newInstance(클래스정보)`를 통해서 `RowMapper`를 간편하게 구현할 수 있다.

## JdbcTemplate - 이름 지정 파라미터 2

파라미터를 전달하려면 `Map`처럼 `key`, `value` 데이터 구조를 만들어서 전달해야 한다. 여기서 `key`는 `:파라미터이름`으로 지정한 파라미터의 이름이고 `value`는 해당 파라미터의 값이 된다.

그리고 이름 지정 바인딩에서 자주 사용하는 파라미터의 종류는 크게 세 가지로 나뉜다.

- `Map`
- `SqlParameterSource (interface)`
  - `MapSqlParameterSource`
  - `BeanPropertySqlParameterSource`

#### 1. Map

단순히 맵을 사용하는 부분은 다음과 같이 `findById()`에서 확인할 수 있다.

```java
Map<String, Long> param = Map.of("id", id);
Item item = template.queryForObject(sql, param, itemRowMapper());
```

#### 2. MapSqlParameterSource

`Map`과 유사한데, SQL 타입을 지정할 수 있는 등 SQL에 좀 더 특화된 기능을 제공한다. `SQLParameterSource` 인터페이스의 구현체이다. `MapSqlParameterSource`는 메서드 체인을 통해 편리한 사용법도 제공한다.

`update()`에서 확인할 수 있다.

```java
SqlParameterSource param = new MapSqlParameterSource()
        .addValue("itemName", updateParam.getItemName())
        .addValue("price", updateParam.getPrice())
        .addValue("quantity", updateParam.getQuantity())
        .addValue("id", itemId);

template.update(sql, param);
```

#### 3. BeanPropertySqlParameterSource

다음의 예시와 같이, 자바빈 프로퍼티 규약을 통해서 자동으로 파라미터 객체를 생성한다.

```
getXxx() -> xxx
getItemName() -> itemName
```

예를 들어서 `getItemName()`, `getPrice()`가 있으면 다음과 같은 데이터를 자동으로 만들어낸다.

```
key=itemName, value=상품명 값
key=price, value=가격 값
```

`SqlParameterSource` 인터페이스의 구현체이다.

`save()`, `findAll()` 코드에서 확인할 수 있다.

```java
SqlParameterSource param = new BeanPropertySqlParameterSource(item);
KeyHolder keyHolder = new GeneratedKeyHolder();
template.update(sql, param, keyHolder);
```

위 코드를 살펴보면 `BeanPropertySqlParameterSource`가 많은 것을 자동화 해주기 때문에 가장 좋아보이지만, `BeanPropertySqlParameterSource`를 항상 사용할 수 있는 것은 아니다.

예를 들어서 `update()`에서는 SQL에 `:id`를 바인딩 해야 하는데, `update()`에서 사용하는 `ItemUpdateDto`에는 `itemId`가 없다.

따라서 `BeanPropertySqlParameterSource`를 사용할 수 없고, 대신에 `MapSqlParameterSource`를 사용했다.

## BeanPropertyRowMapper

이번 코드엔 `V1`과 비교해서 변화된 부분이 하나 더 있다. 바로 `BeanPropertyRowMapper`를 사용해서 `RowMapper`를 생성한 것이다.

```java title="V1"
private RowMapper<Item> itemRowMapper() {
	return (rs, rowNum) -> {
		Item item = new Item();
		item.setId(rs.getLong("id"));
		item.setItemName(rs.getString("item_name"));
		item.setPrice(rs.getInt("price"));
		item.setQuantity(rs.getInt("quantity"));
		return item;
	};
}
```

```java title="V2"
private RowMapper<Item> itemRowMapper() {
    return BeanPropertyRowMapper.newInstance(Item.class);
}
```

`BeanPropertyRowMapper`는 `ResultSet`의 결과를 받아서 자바빈 규약에 맞추어 데이터를 변환한다.
예를 들어서 데이터베이스에서 조회한 결과가 `SELECT id, price`라고 하면 다음과 같은 코드를 작성해준다.

> 실제로는 리플렉션 같은 기능을 사용한다.

```java
Item item = new Item();
item.setId(rs.getLong("id"));
item.setPrice(rs.getInt("price"));
```

데이터베이스에서 조회한 결과 이름을 기반으로 `setId()`, `setPrice()`처럼 자바빈 프로퍼티 규약에 맞춘 메서드를 호출하는 것이다.

**별칭**

그런데 `SELECT item_name`의 경우 `setItem_name()`이라는 메서드가 없다. 이런 경우 개발자가 조회 SQL을 다음과 같이 SQL의 별칭 문법을 통해 고치면 된다.

```sql
SELECT item_name as itemName
```

별칭 `as`를 사용해서 SQL 조회 결과의 이름을 변경하는 것이다. 실제로 이 방법은 자주 사용된다. 컬럼 이름과 객체 이름이 완전히 다를 때의 문제를 해결할 수 있다.

이렇게 데이터베이스 컬럼 이름과 객체의 이름이 다를 때 별칭(as)을 사용해서 문제를 많이 해결한다. `JdbcTemplate`은 물론이고, `MyBatis` 같은 기술에서도 자주 사용된다.

**관례의 불일치**

자바 객체는 카멜(camel case) 표기법을 사용한다. `itemName`처럼 중간에 낙타 봉이 올라와 있는 것 같다해서 카멜 표기법이라 한다.

반면에 관계형 데이터베이스에서는 주로 언더스코어를 사용하는 스네이크 케이스(snake case)를 사용한다. `item_name`처럼 중간에 언더스코어를 사용하는 표기법이다.

이 부분을 관례로 많이 사용하다 보니 `BeanPropertyRowMapper`는 **언더스코어 표기법을 카멜로 자동 변환해준다.** 따라서 `SELECT item_name`으로 조회해도 `setItemName()`에 문제 없이 값이 들어간다.

> 정리하면 `snake_case`는 자동으로 해결되니 그냥 두면 되고, 컬럼 이름과 객체 이름이 완전히 다른 경우에는 조회 SQL에서 별칭을 사용하면 된다.

## JdbcTemplate - 이름 지정 파라미터 3

이제 이름 지정 파라미터를 사용하도록 구성하고 실행해본다. 다음과 같이 `JdbcTemplateV2Configure`를 만들어서 이제 `itemRepository`가 `JdbcTemplateItemRepositoryV2`를 반환해서 스프링 빈으로 등록하도록 고친다.

```java
@Configuration
@RequiredArgsConstructor
public class JdbcTemplateV2Configure {
    private final DataSource dataSource;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JdbcTemplateItemRepositoryV2(dataSource);
    }
}
```

그리고 다음과 같이 현재 스프링 어플리케이션의 엔트리 포인트인 클래스 `ItemServiceApplication`에 `@Import`를 변경한다.

```java
@Import(JdbcTemplateV2Configure.class)
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")
public class ItemServiceApplication {
	...
}
```

어플리케이션을 실행해보면 정상적으로 잘 작동한다.

> [!summary] 정리
> `V1`과 비교해 달라진 점은 SQL 파라미터 바인딩을 순서에 의존하는 것이 아니라 파라미터 이름으로 지정해서 휴먼 에러를 방지하고 `SqlParamterSource`,` BeanPropertyRowMapper`를 통해 개발 편의성을 챙겼다는 점이다.

## JdbcTemplate - SimpleJdbcInsert

`JdbcTemplate`은 `INSERT SQL`을 직접 작성하지 않아도 되도록 `SimpleJdbcInsert`라는 편리한 기능을 제공한다.

먼저 `SimpleJdbcInsert`를 사용하기 위해서는 데이터소스를 넘겨줘야 한다.

```java
private final NamedParameterJdbcTemplate template;
private final SimpleJdbcInsert jdbcInsert;

public JdbcTemplateItemRepositoryV3(DataSource dataSource) {
    this.template = new NamedParameterJdbcTemplate(dataSource);
    this.jdbcInsert = new SimpleJdbcInsert(dataSource)
            .withTableName("item")
            .usingGeneratedKeyColumns("id");
		// .usingColumns("item_name", "price", "quantity"); // 생략 가능
}
```

생성자를 살펴보면 `this.jdbcInsert = new SimpleJdbcInsert(dataSource)`를 통해서 데이터소스를 넘겨주는 것을 알 수 있다. 물론 `SimpleJdbcInsert`를 스프링 빈으로 직접 등록하고 주입 받아도 된다.

```java
this.jdbcInsert = new SimpleJdbcInsert(dataSource)
            .withTableName("item")
            .usingGeneratedKeyColumns("id");
            // .usingColumns("item_name", "price", "quantity"); // 생략 가능
```

- `withTableName`: 데이터를 저장할 테이블 명을 지정한다.
- `usingGeneratedKeyColumns`: `key`를 생성하는 PK 컬럼 명을 지정한다.
- `usingColumns`: INSERT SQL에 사용할 컬럼을 지정한다. 특정 값만 저장하고 싶을 때 사용한다. 생략할 수 있다.

`SimpleJdbcInsert`는 생성 시점에 데이터베이스 테이블의 메타 데이터를 조회한다. 따라서 어떤 컬럼이 있는지 확인 할 수 있으므로 `usingColumns`를 생략할 수 있다.

다음은 직접 `INSERT`를 실행하는 메서드인 `save()`를 살펴보자.

```java
@Override
public Item save(Item item) {
    SqlParameterSource param = new BeanPropertySqlParameterSource(item);
    Number key = jdbcInsert.executeAndReturnKey(param);

    item.setId(key.longValue());
    return item;
}
```

`SimpleJdbcInsert`를 사용하면 위와 같이 `jdbcInsert.executeAndReturnKey(param)`을 통해서 삽입 SQL을 실행하고, 생성된 키 값도 매우 편리하게 조회할 수 있다.

나머지 코드엔 삽입 SQL이 없으므로 기존의 코드와 동일하다. 이제 이 `V3`를 실행해본다. 스프링 빈을 교체하고 `@Import`를 변경한다.

## JdbcTemplate 기능 정리

`JdbcTemplate`의 기능을 간단히 정리해본다.

#### 주요 기능

- `JdbcTemplate`
  - 순서 기반 파라미터 바인딩을 지원한다.
- `NamedParameterJdbcTemplate`
  - 이름 기반 파라미터 바인딩을 지원한다. (권장)
- `SimpleJdbcInsert`
  - INSERT SQL을 편리하게 사용할 수 있다.
- `SimpleJdbcCall
  - 스토어드 프로시저를 편리하게 호출할 수 있다.

데이터소스를 통해 커넥션을 획득하고 사용하기 때문에 데이터소스를 주입 받아야 한다.

## JdbcTemplate 사용법 정리

`JdbcTemplate`에 대한 사용법은 스프링 고식 메뉴얼에 자세히 소개되어 있다. 여기서는 스프링 공식 메뉴얼이 제공하는 예제를 통해 `JdbcTemplate`의 기능을 간단히 정리해본다.

> [스프링 공식 문서 - Using JdbcTemplate](https://docs.spring.io/spring-framework/reference/data-access/jdbc/core.html#jdbc-JdbcTemplate)

#### 조회

**단건 조회 - 숫자 조회, 파라미터 바인딩**

```java
int countOfActorsNamedJoe = jdbcTemplate
.queryForObject("select count(*) from t_actor where first_name = ?",
Integer.class, "Joe");
```

하나의 로우를 조회할 때는 `queryForObject()`를 사용하면 된다. 지금처럼 조회 대상이 객체가 아니라 단순 데이터(예: 숫자) 하나라면 타입을 `Integer.class`, `String.class`와 같이 지정해주면 된다.

**단건 조회 - 숫자 조회, 파라미터 바인딩**

```java
String lastName = jdbcTemplate
.queryForObject(
"select last_name from t_actor where id = ?",
String.class, 1212L);
```

**단건 조회 - 객체 조회**

```java
Actor actor = jdbcTemplate.queryForObject(
"select first_name, last_name from t_actor where id = ?",
(resultSet, rowNum) -> {
	Actor newActor = new Actor();
	newActor.setFirstName(resultSet.getString("first_name"));
	newActor.setLastName(resultSet.getString("last_name"));
	return newActor;
},
1212L);
```

객체 하나를 조회한다. 결과를 자바 객체로 매핑해야 하므로 `RowMapper`를 사용한다. 여기서는 람다를 직접 사용해서 매핑하고 있다.

**목록 조회 - 객체**

```java
List<Actor> actors = jdbcTemplate.query(
"select first_name, last_name from t_actor",
(resultSet, rowNum) -> {
	Actor actor = new Actor();
	actor.setFirstName(resultSet.getString("first_name"));
	actor.setLastName(resultSet.getString("last_name"));
	return actor;
});
```

여러 로우를 조회할 때는 `query()`를 사용하면 된다. 결과를 리스트로 반환한다. 결과를 객체로 매핑해야 하므로 `RowMapper`를 사용해야 한다. 여기서는 람다를 사용했다.

참고로 `JdbcTemplate`이 결과 로우 하나마다 로우 매퍼를 반복해서 적용한다.

#### 변경 (삽입, 수정, 삭제)

데이터를 변경할 때는 `jdbcTemplate.update()`를 사용하면 된다. 참고로 `int` 값을 반환하는데 이는 SQL 실행 결과에 영향 받은 로우 수를 반환한다.

```java
jdbcTemplate.update(
"insert into t_actor (first_name, last_name) values (?, ?)",
"Leonor", "Watling");
```

#### 기타 기능

임의의 SQL을 실행할 때는 `execute()`를 사용하면 된다. 예를 들어 테이블을 생성하거나 프로시저를 호출하는데 사용할 수 있다.

```java
jdbcTemplate.execute("create table mytable (id integer, name varchar(100))");
```

```java
jdbcTemplate.execute(
	"call SUPPORT.REFRESH_ACTORS_SUMMARY(?)",
	Long.valueOf(unionId));
```

> [!summary] 정리
> 실무에서 가장 간단하고 실용적인 방법으로 SQL을 사용하려면 `JdbcTemplate`을 사용하면 된다.
>
> `JPA`와 같은 `ORM`을 사용하면서 동시에 SQL을 직접 작성해야 할 때가 있는데, 그때도 `JdbcTemplate`를 함께 사용하면 된다.
>
> 그런데 `JdbcTemplate`의 최대 단점이 있는데, 바로 동적 쿼리 문제를 해결하지 못한다는 점이다. 그리고 SQL을 자바 코드로 작성하기 때문에 SQL 라인이 코드를 넘어갈 때 마다 문자 더하기를 해주어야 하는 단점도 있다.
>
> 동적 쿼리 문제를 해결하면서 동시에 SQL도 편리하게 작성할 수 있게 도와주는 기술이 바로 `MyBatis`다.

---

References: 김영한의 스프링 DB 2편

Links to this page: [[JdbcTemplate]], [[스프링 부트의 자동 리소스 등록]], [[템플릿 메서드 패턴과 콜백 패턴]]
