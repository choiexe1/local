---
title: 
tags:
  - java
  - programming
  - spring
  - database
  - mybatis
publish: true
date: 2024-12-16 00:00
---

## MyBatis 소개

`MyBatis`는 앞서 설명한 `JdbcTemplate`보다 더 많은 기능을 제공하는 `SQL Mapper`이다. 기본적으로 `JdbcTemplate`이 제공하는 대부분의 기능을 제공한다.

`JdbcTemplate`와 비교해서 `MyBatis`의 가장 매력적인 점은 SQL을 XML에 편리하게 작성할 수 있고 동적 쿼리를 매우 편리하게 작성할 수 있다는 점이다.

#### JdbcTemplate - SQL 여러 줄

```java
String sql = "update item " +
"set item_name=:itemName, price=:price, quantity=:quantity " +
"where id=:id";
```

#### MyBatis - SQL 여러 줄

```xml
<update id="update">
update item
set item_name=#{itemName},
	price=#{price},
	quantity=#{quantity}
where id = #{id}
</update>
```

`MyBatis`는 XML에 작성하기 때문에 라인이 길어져도 문자 더하기에 대한 불편함이 없다. 다음으로 상품을 검색하는 로직을 통해 동적 쿼리를 비교해보자.

#### JdbcTemplate - 동적 쿼리

```java
String sql = "select id, item_name, price, quantity from item";

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
```

#### MyBatis - 동적 쿼리

```xml
<select id="findAll" resultType="Item">
	select id, item_name, price, quantity
	from item
	<where>
		<if test="itemName != null and itemName != ''">
			and item_name like concat('%',#{itemName},'%')
		</if>
		<if test="maxPrice != null">
			and price &lt;= #{maxPrice}
		</if>
	</where>
</select>
```

`JdbcTemplate`은 자바 코드로 직접 동적 쿼리를 작성해야 한다. 반면에 `MyBatis`는 동적 쿼리를 매우 편리하게 작성 할 수 있는 다양한 기능들을 제공해준다.

**설정의 장단점**

`JdbcTemplate`은 스프링에 내장된 기능이고, 별도의 설정 없이 사용할 수 있다는 장점이 있다. 반면에 `MyBatis`는 약간의 설정이 필요하다.

> [!summary] 정리
> 프로젝트에서 동적 쿼리와 복잡한 쿼리가 많다면 `MyBatis`를 사용하고, 단순한 쿼리들이 많으면 `JdbcTemplate`를 선택해서 사용하면 된다. 물론 둘을 함께 사용해도 된다. 하지만 `MyBatis`를 선택했다면 그것으로 충분할 것이다.
>
> 강의에서는 `MyBatis`의 기능을 자세하게 다루지는 않는다. `MyBatis`를 왜 사용하는지 그리고 주로 사용하는 기능 위주로 다룬다. `MyBatis`는 기능도 단순하고 공식 사이트가 한글로 잘 번역 되어 있어서 원하는 기능을 편리하게 찾아볼 수 있다.
>
> [MyBatis 공식 문서](https://mybatis.org/mybatis-3/ko/index.html)

## MyBatis 설정

`mybatis-spring-boot-starter` 라이브러리를 사용하면 `MyBatis`를 스프링과 통합하고, 설정도 아주 간단히 할 수 있다. `mybatis-spring-boot-starter` 라이브러리를 사용해서 간단히 설정하는 방법을 알아보자.

`build.gradle`에 다음 의존 관계를 추가한다.

```gradle title="build.gradle"
implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0'
```

다음으로 `application.properties`에 다음 설정을 추가한다. 웹 어플리케이션을 실행하는 `main`, `test` 각 위치의 `application.properties`를 모두 수정해야 한다.

```properties
# MyBatis
mybatis.type-aliases-package=hello.itemservice.domain
mybatis.configuration.map-underscore-to-camel-case=true
logging.level.hello.itemservice.repository.mybatis=trace
```

- `mybatis.type-aliases-package=hello.itemservice.domain`
  - 마이바티스에서 타입 정보를 사용할 때는 패키지 이름을 적어주어야 하는데, 여기에 명시하면 패키지 이름을 생략할 수 있다.
  - 지정한 패키지와 그 하위 패키지가 자동으로 인식된다.
  - 여러 위치를 지정하려면 `,` 또는 `;`로 구분하면 된다.
- `mybatis.configuration.map-underscore-to-camel-case=true`
  - `JdbcTemplate`의 `BeanPropertyRowMapper`에서처럼 스네이크 케이스를 카멜 케이스로 자동 변환해주는 기능을 활성화한다.
- `logging.level.hello.itemservice.repository.mybatis=trace`
  - `MyBatis`에서 실행되는 쿼리 로그를 확인할 수 있다.

## MyBatis 적용 1 - 기본

이제부터 본격적으로 `MyBatis`를 사용해서 데이터베이스에 데이터를 저장해본다. XML에 작성한다는 점을 제외하고는 `JDBC` 반복을 줄여준다는 점에서 기존 `JdbcTemplate`과 거의 유사하다.

```java title="ItemMapper.java"
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

@Mapper
public interface ItemMapper {
    void save(Item item);

    void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto itemUpdateDto);

    List<Item> findAll(ItemSearchCond itemSearchCond);

    Optional<Item> findById(Long id);
}
```

- 마이바티스 매핑 XML을 호출해주는 매퍼 인터페이스이다.
- 이 인터페이스에는 `@Mapper` 어노테이션을 붙여주어야 한다. 그래야 마이바티스에서 인식할 수 있다.
- 이 인터페이스의 메서드를 호출하면 다음에 보이는 XML의 해당 SQL을 실행하고 결과를 돌려준다.
- `ItemMapper` 인터페이스의 구현체에 대한 부분은 뒤에 별도로 설명한다.

이제 같은 위치에 실행할 SQL이 있는 XML 매핑 파일을 만들어주면 된다. 참고로 자바 코드가 아니기 때문에 `src/main/resources` 하위에 만들되, 패키지 위치는 맞춰줘야 한다.

```xml title="main/resources/hello/itemservice/repository/mybatis/ItemMapper.xml"
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="hello.itemservice.repository.mybatis.ItemMapper">

    <insert id="save" useGeneratedKeys="true" keyProperty="id">
        INSERT INTO item (item_name, price, quantity)
        VALUES (#{itemName}, #{price}, #{quantity})
	</insert>

    <update id="update">
        UPDATE item
        SET item_name = #{updateParam.itemName},
			price     = #{updateParam.price},
			quantity  = #{updateParam.quantity}
		WHERE id = #{id}
	</update>

    <select id="findById" resultType="Item">
        SELECT id,
               item_name,
               price,
			   quantity
	    FROM item
		WHERE id = #{id}
	</select>

    <select id="findAll" resultType="Item">
        SELECT id, item_name, price, quantity
        FROM item
        <where>
            <if test="itemName != null and itemName != ''">
                AND item_name like concat('%', #{itemName}, '%')
            </if>
            <if test="maxPrice != null">
                AND price &lt;= #{maxPrice}
            </if>
        </where>
    </select>

</mapper>
```

`<mapper> 태그 namespace` 속성: 앞서 만든 매퍼 클래스의 인터페이스를 지정한다.
`main/resources/hello/itemservice/repository/mybatis/ItemMapper.xml`: 경로와 파일 이름이 정확히 매핑되어야 한다.

> [!tip] XML 파일 경로 수정하기
> XML 파일을 원하는 위치에 두고 싶으면 `application.properties`에 다음과 같이 설정하면 된다.
> `mybatis.mapper-locations=classpath:mapper/**/*.xml`
>
> 이렇게 하면 `resources/mapper`를 포함한 그 하위 폴더에 있는 XML을 XML 매핑 파일로 인식한다. 이 경우 파일 이름은 자유롭게 설정해도 된다.
>
> 참고로 테스트의 `application.properties`도 함께 수정해야 테스트를 실행할 때 인식할 수 있다.

#### INSERT - save()

```xml
<insert id="save" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO item (item_name, price, quantity)
    VALUES (#{itemName}, #{price}, #{quantity})
</insert>
```

- INSERT SQL은 `<insert>` 태그를 사용한다.
- `id`에는 매퍼 인터페이스에 설정한 메서드 이름을 지정하면 된다. 여기서는 메서드 이름이 `save()`이므로 `save`로 지정한다.
- 파라미터는 `#{}` 문법을 사용하면 된다. 그리고 매퍼에서 넘긴 객체의 프로퍼티 이름을 적어주면 된다.
- `#{}` 문법을 사용하면 내부적으로 `PreparedStatement`를 사용한다. 따라서 `JDBC`의 `?`를 치환하는 것과 동일하다.
- `useGeneratedKeys`는 데이터베이스가 키를 생성해주는 `Identity` 전략일 때 사용한다. `keyProperty`는 생성되는 키의 속성 이름을 지정한다. INSERT가 끝나면 `item` 객체의 `id` 속성에 생성된 값이 입력된다.

#### UPDATE - update

```xml
<update id="update">
        UPDATE item
        SET item_name = #{updateParam.itemName},
			price     = #{updateParam.price},
			quantity  = #{updateParam.quantity}
		WHERE id = #{id}
	</update>
```

- UPDATE SQL은 `<update>` 태그를 사용한다.
- 여기서 매퍼의 파라미터가 `Long id, ItemUpdateDto updateParam`으로 2개이다. 파라미터가 1개만 있으면 `@Param`을 지정하지 않아도 되지만, 파라미터가 2개 이상이면 `@Param`으로 이름을 지정해서 파라미터를 구분해야 한다.

#### SELECT - findById

```xml
<select id="findById" resultType="Item">
        SELECT id,
               item_name,
               price,
			   quantity
	    FROM item
		WHERE id = #{id}
</select>
```

- SELECT SQL은 `<select>` 태그를 사용하면 된다.
- `resultType`은 반환 타입을 명시하면 된다. 여기서는 결과를 `Item` 객체에 매핑한다.
  - 앞서 `application.properties`에 `mybatis.type-aliases-package=hello.itemservice.domain` 속성을 지정한 덕분에 모든 패키지명을 다 적지는 않아도 된다. 그렇지 않으면 모든 패키지 명을 다 적어야 한다.
  - `JdbcTemplate`의 `BeanPropertyRowMapper`처럼 SELECT SQL의 결과를 편리하게 객체로 바로 변환해준다.
  - `mybatis.configuration.map-underscore-to-camel-case=true` 속성을 지정한 덕분에 언더 스코어를 카멜 표기법으로 자동으로 처리해준다.

#### SELECT - findAll

```xml
<select id="findAll" resultType="Item">
        SELECT id, item_name, price, quantity
        FROM item
        <where>
            <if test="itemName != null and itemName != ''">
                AND item_name like concat('%', #{itemName}, '%')
            </if>
            <if test="maxPrice != null">
                AND price &lt;= #{maxPrice}
            </if>
        </where>
</select>
```

- 마이바티스는 `<where>` 태그와 `<if>` 같은 동적 쿼리 문법을 통해 편리한 동적 쿼리를 지원한다.
- `<if>`는 해당 조건이 만족하면 구문을 추가한다.
- `<where>`절은 적절하게 `where` 문장을 만들어준다.
  - 예제에서 `<if>`가 모두 실패하면 SQL`WHERE`를 만들지 않는다.
  - 예제에서 `<if>`가 하나라도 성공하면 처음 나타나는 `AND`를 `WHERE`로 변환 해준다.

#### XML 특수문자

그런데 가격을 비교하는 조건을 살펴보자

- `and price &lt;= #{maxPrice}`
  여기에 보면 `<=` 를 사용하지 않고 `&lt;=`를 사용한 것을 확인할 수 있다. 그 이유는 XML에서 태그가 시작하거나 종료할 때 `<`와 `>` 같은 특수문자를 사용하기 때문이다.

XML에서는 데이터 영역에 `<`, `>` 같은 특수문자를 사용할 수 없기 때문이다.

```
< : &lt;
> : &gt:
& : &amp;
```

다음과 같이 특수문자를 대체하는 `CDATA` 구문 문법도 존재한다.

```xml
<if test="maxPrice != null">
	<![CDATA[
	and price <= #{maxPrice}
	]]>
</if>
```

## MyBatis 적용 2 - 설정과 실행

먼저 다음과 같이 리포지토리를 구성한다.

```java
@Repository
@RequiredArgsConstructor
public class MyBatisItemRepository implements ItemRepository {
    private final ItemMapper itemMapper;

    @Override
    public Item save(Item item) {
        itemMapper.save(item);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        itemMapper.update(itemId, updateParam);
    }

    @Override
    public Optional<Item> findById(Long id) {
        return itemMapper.findById(id);
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        return itemMapper.findAll(cond);
    }
}
```

- 마이바티스는 앞서 구현했던 `ItemMapper` 인터페이스의 구현체를 자동으로 만든다.
- `MyBatisItemRepository`는 주입 받은 `ItemMapper`에 기능을 위임한다.

다음으로 설정 클래스를 만들고, 해당 설정을 스프링 부트 설정으로 사용한다.

```java titile="MyBatisConfigure.java"
@Configuration
@RequiredArgsConstructor
public class MyBatisConfigure {
    private final ItemMapper itemMapper;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new MyBatisItemRepository(itemMapper);
    }

}
```

```java title="ItemServiceApplication.java"
@Import(MyBatisConfigure.class)
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")
@MapperScan("hello.itemservice.repository.mybatis")
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

`@MapperScan("hello.itemservice.repository.mybatis")`은 `MyBatisConfigure`의 `ItemMapper`를 주입 받을 때 제대로 찾지 못해서 넣어준 설정 어노테이션이다.

테스트를 실행해보면 정상적으로 잘 작동하는 것을 알 수 있다.

## MyBatis 적용 3 - 분석

생각해보면 지금까지 진행한 부분 중 이상한 부분이 있다. `ItemMapper`의 구현체가 없는데 어떻게 동작하는 것일까?

```java
@Mapper
public interface ItemMapper {
    void save(Item item);

    void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto itemUpdateDto);

    List<Item> findAll(ItemSearchCond itemSearchCond);

    Optional<Item> findById(Long id);
}
```

이 부분은 `MyBatis` 스프링 연동 모듈에서 자동으로 처리해주는데 다음과 같다.

![[mybatis-1.png]]

1. 어플리케이션 로딩 시점에 `MyBatis` 스프링 연동 모듈은 `@Mapper`가 붙어있는 인터페이스를 조사한다.
2. 해당 인터페이스가 발견되면 동적 프록시 기술을 사용해서 `ItemMapper` 인터페이스의 구현체를 만든다.
3. 생성된 구현체를 스프링 빈으로 등록한다.

로그로 주입 받은 `ItemMapper`의 클래스를 출력해보면 다음과 같다.

```
itemMapper class = class com.sun.proxy.$Proxy66
```

**매퍼 구현체**

- 마이바티스 스프링 연동 모듈이 만들어주는 `ItemMapper`의 구현체 덕분에 인터페이스만으로 편리하게 XML의 데이터를 찾아서 호출할 수 있다.
- 원래 마이바티스를 사용하려면 더 번잡한 코드를 거쳐야 하는데, 이런 부분을 인터페이스 하나로 매우 깔끔하고 편리하게 사용할 수 있다.
- 매퍼 구현체는 예외 변환까지 해준다. 마이바티스에서 발생한 예외를 스프링 예외 추상화인 `DataAccessException`에 맞게 변환해서 반환해준다. `JdbcTemplate`이 제공하는 예외 변환 기능을 여기서도 제공한다.

> [!summary] 정리
>
> - 매퍼 구현체 덕분에 마이바티스를 스프링에 편리하게 통합해서 사용할 수 있다.
> - 매퍼 구현체를 사용하면 스프링 예외 추상화도 함께 적용된다.
> - 마이바티스 스프링 연동 모듈이 많은 부분을 자동으로 설정해주는데, 데이터베이스 커넥션, 트랜잭션과 관련된 기능도 마이바티스와 함께 연동하고, 동기화 해준다.

## MyBatis 기능 정리 1 - 동적 쿼리

마이바티스에서 자주 사용하는 주요 기능을 공식 메뉴얼이 제공하는 예제를 통해 간단히 정리해본다.

- [마이바티스 공식 메뉴얼](https://mybatis.org/mybatis-3/ko/index.html)
- [마이바티스 스프링 공식 메뉴얼](https://mybatis.org/spring/ko/index.html)

#### 동적 SQL

마이바티스가 제공하는 최고의 기능이자 마이바티스를 사용하는 이유는 바로 동적 SQL 기능 때문이다. 동적 쿼리를 위해 제공되는 기능은 다음과 같다.

- `if`
- `choose (when, otherwise)`
- `trim (where, set)`
- `foreach`

**if**

```xml
<select id="findActiveBlogWithTitleLike"
resultType="Blog">
	SELECT * FROM BLOG
	WHERE state = ‘ACTIVE’
	<if test="title != null">
		AND title like #{title}
	</if>
</select>
```

- 해당 조건에 따라 값을 추가할지 말지 판단한다.
- 내부의 문법은 OGNL을 사용한다고 한다. 자세한 내용은 OGNL을 검색해보자.

**choose, when, otherwise**

```xml
<select id="findActiveBlogLike"
resultType="Blog">
	SELECT * FROM BLOG WHERE state = ‘ACTIVE’
	<choose>
		<when test="title != null">
			AND title like #{title}
		</when>
		<when test="author != null and author.name != null">
			AND author_name like #{author.name}
		</when>
		<otherwise>
			AND featured = 1
		</otherwise>
	</choose>
</select>
```

- 자바의 switch 구문과 유사한 구문도 사용할 수 있다.

**trim, where, set**

```xml
<select id="findActiveBlogLike"
resultType="Blog">
	SELECT * FROM BLOG
	WHERE
	<if test="state != null">
		state = #{state}
	</if>
	<if test="title != null">
		AND title like #{title}
	</if>
	<if test="author != null and author.name != null">
		AND author_name like #{author.name}
	</if>
</select>
```

- 위 예제의 문제는 `if`가 모두 만족하지 않을 때 구문 오류가 발생한다.
- `title`만 만족할 때도 문제가 발생한다.
- 결국 `WHERE`를 언제 넣어야 할 지 상황에 따라서 동적으로 달라지는 문제가 있다.
- `<where>`를 사용하면 이런 문제를 해결 할 수 있다.

```xml title="WHERE 태그 사용"
    <select id="findActiveBlogLike" resultType="Blog">
        SELECT * FROM BLOG
        <where>
            <if test="state != null">
                state = #{state}
            </if>
            <if test="title != null">
                AND title LIKE #{title}
            </if>
            <if test="author != null and author.name != null">
                AND author_name LIKE #{author.name}
            </if>
        </where>
    </select>
```

`<where>`는 문장이 없으면 `<where>`를 추가하지 않는다. 문장이 있으면 `<where>`를 추가한다. 만약 `and`가 먼저 시작된다면 `and`를 지운다.

참고로 다음과 같이 `trim`이라는 기능으로 사용해도 된다. 이렇게 정의하면 `<where>`와 같은 기능을 수행한다.

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
	...
</trim>
```

**foreach**

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
    SELECT *
    FROM POST P
    <where>
        <foreach item="item" index="index" collection="list"
                 open="ID IN (" separator="," close=")">
            #{item}
        </foreach>
    </where>
</select>
```

- 컬렉션을 반복 처리할 때 사용한다. `WHERE IN (1, 2, 3)`과 같은 SQL을 쉽게 완성할 수 있다.
- 파라미터로 `List`를 전달하면 된다.

> 동적 쿼리에 대한 자세한 내용은 [마이바티스 공식 문서 - 동적 SQL](https://mybatis.org/mybatis-3/ko/dynamic-sql.html)을 참고하자.

## MyBatis 기능 정리 2 - 기타 기능

#### 어노테이션으로 SQL 작성

```java
@Select("select id, item_name, price, quantity from item where id=#{id}")
Optional<Item> findById(Long id);
```

- `@Insert`, `@Update`, `@Delete`, `@Select` 어노테이션이 제공된다.
- 매퍼 인터페이스의 메서드에 어노테이션을 적용한다.
- 이 경우 XML에는 `<select id="findById">..</select>` 태그는 제거해야 한다.
- 동적 SQL이 해결되지 않으므로 간단한 경우에만 사용한다.

> 어노테이션으로 SQL에 대한 더 자세한 내용은 [마이바티스 공식 문서 - 자바 API](https://mybatis.org/mybatis-3/ko/java-api.html)를 참고하자.

#### 문자열 대체(String Substitution)

`#{}` 문법은 `?`를 넣고 파라미터를 바인딩하는 `PreparedStatement`를 사용한다. 때로는 파라미터 바인딩이 아니라 문자 그대로를 처리하고 싶은 경우도 있다. 이 때는 `${}`를 사용하면 된다.

다음 예제를 살펴보면 `${column}` 부분에서 문자열을 그대로 바인딩한다.

```java
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, @Param("value") String value);
```

> [!error] SQL 인젝션
> `${}`를 사용하면 SQL 인젝션 공격을 당할 수 있다. 따라서 가급적 사용하면 안된다. 사용하더라도 매우 주의깊게 사용해야 한다.

#### 재사용 가능한 SQL 조각

`<sql>` 태그를 사용하면 SQL 코드를 재사용 할 수 있다.

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password</sql>
```

```xml
<select id="selectUsers" resultType="map">
	select
	<include refid="userColumns"><property name="alias" value="t1"/></include>,
	<include refid="userColumns"><property name="alias" value="t2"/></include>
	from some_table t1
		cross join some_table t2
</select>
```

- 프로퍼티 값을 `<property>` 태그를 통해 전달할 수 있고, 해당 값은 `<sql>` 내부에서 사용할 수 있다.

#### Result Maps

결과를 매핑할 때 테이블은 `user_id`이지만 객체는 `id`이다. 이 경우 컬럼명과 객체의 프로퍼티 명이 다르다. 그러면 다음과 같이 별칭(as)을 사용하면 된다.

```xml
<select id="selectUsers" resultType="User">
	select user_id      as "id",
		user_name       as "userName",
		hashed_password as "hashedPassword"
	from some_table
	where id = #{id}
</select>
```

그런데 동일한 문제를 별칭을 사용하지 않고도 해결할 수 있다. 다음과 같이 `resultMap`을 선언해서 사용하면 된다.

```xml
<resultMap id="userResultMap" type="User">
	<id property="id" column="user_id" />
	<result property="username" column="user_name"/>
	<result property="password" column="hashed_password"/>
</resultMap>

<select id="selectUsers" resultMap="userResultMap">
	select user_id, user_name, hashed_password
	from some_table
	where id = #{id}
</select>
```

#### 복잡한 결과 매핑

마이바티스도 매우 복잡한 결과에 객체 연관관계를 고려해서 데이터를 조회하는 것이 가능하다. 이때는 `<association>`, `<collection>`등을 사용한다.

이 부분은 성능과 실효성 측면에서 많은 고민이 필요하다. `JPA`는 객체와 관계형 데이터베이스를 `ORM` 개념으로 매핑하기 때문에 이런 부분이 자연스럽지만, 마이바티스에서는 들어가는 공수도 많고, 성능을 최적화 하기도 어렵다. 따라서 해당 기능을 사용할 때는 신중하게 사용해야 한다.

해당 기능에 대한 자세한 내용은 [공식 메뉴얼](https://mybatis.org/mybatis-3/ko/sqlmap-xml.html#Result_Maps)을 참고하자.

---

References: 김영한의 스프링 DB 2편

Links to this page:
