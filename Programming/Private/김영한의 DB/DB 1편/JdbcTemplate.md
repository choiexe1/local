---
title: 
tags:
  - java
  - programming
  - spring
  - jdbc
  - database
publish: true
date: 2024-12-14 00:02
---

## JDBC 반복 문제 해결 - JDBC Template

지금까지 서비스 계층의 순수함을 유지하기 위해 수 많은 노력을 햇고, 덕분에 서비스 계층의 순수함을 유지하게 되었다. 이번에는 리포지토리에서 JDBC를 사용하기 때문에 발생하는 반복 문제를 해결해본다.

**반복 문제**

- 커넥션 조회, 커넥션 동기화
- `PreparedStatement` 생성 및 파라미터 바인딩
- 쿼리 실행
- 결과 바인딩
- 예외 발생 시 스프링 예외 변환기 실행
- 리소스 종료

리포지토리의 각 메서드를 살펴보면 많은 부분이 반복된다. 이런 반복을 효과적으로 처리하는 방법이 바로 앞서 서비스 레이어의 트랜잭션에서도 사용했던 **템플릿 콜백 패턴**이다.

스프링은 이 `JDBC`의 반복 문제를 해결하기 위해서 `JdbcTemplate`라는 템플릿을 제공한다. `JdbcTemplate`에 대한 자세한 사용법은 뒤에서 설명한다. 따라서 지금은 전체 구조와 이 기능을 사용해서 반복 코드를 제거할 수 있다는 것에 초점을 맞춘다.

```java title="MemberRepositoryV5.java"
/**
 * JDBC Template 사용
 */
@Slf4j
@RequiredArgsConstructor
public class MemberRepositoryV5 implements MemberRepository {
    private final DataSource dataSource;
    private final JdbcTemplate template;

    public MemberRepositoryV5(DataSource dataSource) {
        this.dataSource = dataSource;
        this.template = new JdbcTemplate(dataSource);
    }

    @Override
    public Member save(Member member) {
        String sql = "INSERT INTO member(member_id, money) VALUES(?, ?)";
        template.update(sql, member.getMemberId(), member.getMoney());
        return member;
    }

    @Override
    public Member findById(String memberId) {
        String sql = "SELECT * FROM member WHERE member_id = ?";
        return template.queryForObject(sql, memberRowMapper(), memberId);
    }

    @Override
    public void update(String memberId, int money) {
        String sql = "UPDATE member SET money = ? WHERE member_id = ?";
        template.update(sql, money, memberId);
    }

    @Override
    public void delete(String memberId) {
        String sql = "DELETE FROM member WHERE member_id = ?";
        template.update(sql, memberId);
    }

    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setMemberId(rs.getString("member_id"));
            member.setMoney(rs.getInt("money"));
            return member;
        };
    }
}
```

여태까지 작성했던 코드는 도대체 무엇이었는지 허망할정도로 코드가 매우 깔끔해졌다.

`JdbcTemplate`는 `JDBC`로 개발할 때 발생하는 반복을 대부분 해결해준다.

> 그 뿐만 아니라 지금까지 학습했던 **트랜잭션을 위한 커넥션 동기화**는 물론이고, 예외 발생 시 **스프링 예외 변환기**도 자동으로 실행해준다.

> [!summary] 정리
> 완성된 코드를 확인해보자.
>
> - 서비스 계층의 순수성
>   - 트랜잭션 추상화 + 트랜잭션 AOP 덕분에 서비스 계층의 순수성을 최대한 유지하면서 서비스 계층에서 트랜잭션을 사용할 수 있다.
>   - 스프링이 제공하는 예외 추상화와 예외 변환기 덕분에 데이터 접근 기술이 변경되어도 서비스 계층의 순수성을 유지하면서 예외도 사용할 수 있다.
> - 리포지토리에서 JDBC를 사용하는 반복 코드가 `JdbcTemplate`으로 대부분 제거되었다.

---

References: 김영한의 스프링 DB 1편

Links to this page:
