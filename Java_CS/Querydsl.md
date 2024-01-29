# Querydsl

- 복잡한 쿼리와 동적 쿼리 문제를 해결하기 위해 필요함

## 특징

- 쿼리를 자바 코드로 작성
- 문법 오류를 컴파일 시점에 잡아준다.
    - JPQL의 경우 사용자가 실행해서 메소드를 호출한 뒤에 오류를 발견할 수 있다.
- 동적 쿼리 문제 해결
- 쉬운 SQL 스타일 문법
- 단순 반복 X, 쿼리도 자바 코드로 작성

## 사용법

Entity 생성 → 인텔리제이 우측 상단 Gradle → Tasks → other → compile.java실행

build - generated - Q(Entity이름) 붙은 파일 생성되었는지 확인

### jpql과 querydsl 비교

```java
public void jpql() {
	String username = "kim";
	Stirng query = "select m form Member m" +
						" where m.username = :username";
	List<Member> result = em.createQuery(query, Member.class)
			.getResultList();
}
// 코드에 오류가 있을 시 실행 시에 오류 잡을 수 있음

@Test
public void querydsl() {
	String username = "kim";
	List<Member> result = queryFactory
				.select(member)
				.from(member)
				.where(member.username.like)
				.fetch();

// 중간중간 오류가 발생하면 빨간 줄로 확인 가능, 자동완성 가능
```

### QuerydslTest 코드

```java
package study.querydsl;

import com.querydsl.jpa.impl.JPAQueryFactory;
import jakarta.persistence.EntityManager;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;
import study.querydsl.entity.Member;
import study.querydsl.entity.QMember;
import study.querydsl.entity.Team;
import static org.assertj.core.api.Assertions.*;
import static study.querydsl.entity.QMember.member;

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Autowired
    EntityManager em;

    JPAQueryFactory queryFactory;

    @BeforeEach
    public void before() {
        queryFactory = new JPAQueryFactory(em);
        Team teamA = new Team("temaA");
        Team teamB = new Team("temaB");
        em.persist(teamA);
        em.persist(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 20, teamA);

        Member member3 = new Member("member3", 30, teamB);
        Member member4 = new Member("member4", 40, teamB);

        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);

    }

    @Test
    public void startJPQL() {
        // member1을 찾아라.
        Member findMember = em.createQuery("select m from Member m " +
                        "where m.username = :username", Member.class)
                .setParameter("username", "member1")
                .getSingleResult();

        assertThat(findMember.getUsername()).isEqualTo("member1");

    }

    @Test
    public void startQuerydsl() {
//        QMember m = new QMember("m");
//        QMember m = QMember.member;
        Member findMember = queryFactory
                .select(member)
                .from(member)
                .where(member.username.eq("member1"))//알아서 파라미터 바인딩 처리해준다.
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```

## 검색 조건

```java
@Test
public void search() {
	Member findMember = queryFactory
		 .selectFrom(member)
		 .where(member.username.eq("member1")
		 .and(member.age.eq(10)))
		 .fetchOne();
		 assertThat(findMember.getUsername()).isEqualTo("member1");
}
```

- 검색 조건은 .and(), .or()를 메서드 체인으로 연결할 수 있다.

### JPQL이 제공하는 모든 문법 제공

```java
member.username.eq("member1") // username = 'member1'
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'
member.username.isNotNull() //이름이 is not null
member.age.in(10, 20) // age in (10,20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) //between 10, 30
member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30
member.username.like("member%") //like 검색
member.username.contains("member") // like ‘%member%’ 검색
member.username.startsWith("member") //like ‘member%’ 검색
```

### AND 조건을 파라미터로 처리

```java
@Test
    public void searchAndParam() {
        Member findMember = queryFactory
                .selectFrom(member)
                .where(
                        member.username.eq("member1"),
                        member.age.eq(10)
                )
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
```

- where()에 파라미터로 검색조건을 추가하면 AND 조건이 추가됨
- 이 경우 null 값은 무시 → 메서드 추출을 활용해서 동적 쿼리를 깔끔하게 만들 수 있음

## 결과 조회

- fetch() : 리스트 조회, 데이터 없으면 빈 리스트 봔환
- fetchOne() : 단 건 조회
    - 결과가 없으면 : null
    - 결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException
- fetchFirst() : limit(1).fetchOne()
- fetchResults() : 페이징 정보 포함, total count 쿼리 추가 실행
- fetchCount() : count 쿼리로 변경해서 count 수 조회

### 결과 조회 코드

```java
@Test
    public void resultFetch() {
        // 멤버의 목록을 리스트로 조회
//        List<Member> fetch = queryFactory
//                .selectFrom(member)
//                .fetch();
//
//        // 단 건 조회
//        Member fetchOne = queryFactory
//                .selectFrom(member)
//                .fetchOne();
//
//        //limit(1)거는거
//        Member fetchFirst = queryFactory
//                .selectFrom(member)
//                .fetchFirst();

        // fetchResults -> getTotal을 제공한다.
        QueryResults<Member> results = queryFactory
                .selectFrom(member)
                .fetchResults();

        results.getTotal();
        // results에 대한 data가 나온다. , limit, offset도 제공된다.
        List<Member> content = results.getResults();

				// 카운트만 가져오는 것
        long total = queryFactory
                .selectFrom(member)
                .fetchCount();
    }
```

### 정렬

```java
/*
    * 회원 정렬 순서
    * 1. 회원 나이 내림차순 desc
    * 2. 직원 이름 오름차순 asc
    * 단 2에서 회원 이름이 없으면 마지막에 출력 (nulls last)
    */
    @Test
    public void sort() {
        em.persist(new Member(null, 100));
        em.persist(new Member("member5", 100));
        em.persist(new Member("member6", 100));

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(100))
                .orderBy(member.age.desc(), member.username.asc().nullsLast())
                .fetch();

        Member member5 = result.get(0);
        Member member6 = result.get(1);
        Member memberNull = result.get(2);
        assertThat(member5.getUsername()).isEqualTo("member5");
        assertThat(member6.getUsername()).isEqualTo("member6");
        assertThat(memberNull.getUsername()).isNull();
    }
```

### 페이징

```java
@Test
    public void paging1() {
        List<Member> result = queryFactory
                .selectFrom(member)
                .orderBy(member.username.desc())
								// orderBy를 넣어야 잘 작동하는 지 확인할 수 있음
                .offset(1)
                .limit(2)
                .fetch();

        assertThat(result.size()).isEqualTo(2);
    }

@Test
    public void paging2() {
        QueryResults<Member> queryResults = queryFactory
                .selectFrom(member)
                .orderBy(member.username.desc())
                .offset(1)
                .limit(2)
                .fetchResults();  // 전체 조회 수

        assertThat(queryResults.getTotal()).isEqualTo(4);
        assertThat(queryResults.getLimit()).isEqualTo(2);
        assertThat(queryResults.getOffset()).isEqualTo(1);
        assertThat(queryResults.getResults().size()).isEqualTo(2);
    }
```

### 집합

```java
@Test
    public void aggregation() {
        List<Tuple> result = queryFactory
                .select(
                        member.count(),
                        member.age.sum(),
                        member.age.avg(),
                        member.age.max(),
                        member.age.min()
                )
                .from(member)
                .fetch();

        Tuple tuple = result.get(0);
        assertThat(tuple.get(member.count())).isEqualTo(4);
        assertThat(tuple.get(member.age.sum())).isEqualTo(100);
        assertThat(tuple.get(member.age.avg())).isEqualTo(25);
        assertThat(tuple.get(member.age.max())).isEqualTo(40);
        assertThat(tuple.get(member.age.min())).isEqualTo(10);
    }

// GroupBy
		@Test
    public void group() throws Exception {
        List<Tuple> result = queryFactory
                .select(team.name, member.age.avg())
                .from(member)
                .join(member.team, team)
                .groupBy(team.name)
								// .having(item.price.gt(1000))
                .fetch();

        Tuple teamA = result.get(0);
        Tuple teamB = result.get(1);

//        assertThat(teamA.get(team.name)).isEqualTo("teamA");
//        assertThat(teamA.get(member.age.avg())).isEqualTo(15);  // (10 + 20) / 2
//
//        assertThat(teamB.get(team.name)).isEqualTo("teamB");
//        assertThat(teamB.get(member.age.avg())).isEqualTo(15);  // (30 + 40) / 2
    }
```

### 조인

```java
		/*
    * 팀 A에 소속된 모든 회원
    */
    @Test
    public void join() {
        QMember member = QMember.member;
        QTeam team = QTeam.team;

        List<Member> result = queryFactory
                .selectFrom(member)
                .join(member.team, team)
								// leftJoin(member.team, team)
                .where(team.name.eq("teamA"))
                .fetch();

        assertThat(result)
                .extracting("username")
                .containsExactly("member1", "member2");
    }
```

- 조인의 기본 문법은 첫 번째 파라미터에 조인 대상을 지정하고, 두 번째 파라미터에 별칭(alias)로 사용할 Q 타입을 지정하면 된다.

> join(조인 대상, 별칭으로 사용할 Q타입)
> 

> join(), innerJoin() : 내부 조인(inner join)
> 

> leftJoin() : left 외부 조인(left outer join)
> 

> rightJoin() : right 외부 조인(right outer join)
> 

### 세타 조인

```java
		/*
    * 세타 조인
    * 회원의 이름이 팀 이름과 같은 회원 조회
    */
    @Test
    public void theta_join() {
        em.persist(new Member("teamA"));
        em.persist(new Member("teamB"));

        List<Member> result = queryFactory
                .select(member)
                .from(member, team)
                .where(member.username.eq(team.name))
                .fetch();

        assertThat(result)
                .extracting("username")
                .containsExactly("teamA", "teamB");
    }
```

- 연관관계가 없는 필드로 조인
- from 절에 여러 엔티티를 선택해서 세타 조인
- 외부 조인 불가능 → 다음에 설명할 조인 on을 사용하면 외부 조인 가능

### 조인 - on절

### 조인 대상 필터링

ex) 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인, 회원은 모두 조회

```java
 		/*
    * ex) 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인, 회원은 모두 조회
    * JPQL : select m, t from Member m left join m.team t on t.name = 'teamA'
    */
    @Test
    public void join_on_filtering() {
        List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(member.team, team).on(team.name.eq("teamA"))
                .fetch();

        for(Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }

// 결과
t=[Member(id=3, username=member1, age=10), Team(id=1, name=teamA)]
t=[Member(id=4, username=member2, age=20), Team(id=1, name=teamA)]
t=[Member(id=5, username=member3, age=30), null]
t=[Member(id=6, username=member4, age=40), null]
```

> 참고 : on 절을 활용해 조인 대상을 필터링 할 때, 외부조인이 아니라 내부조인(inner join)을 사용하면, where 절에서 필터링 하는 것과 기능이 동일하다. 따라서 on 절을 활용한 조인 대상 필터링을 사용할 때, 내부조인 이면 익숙한 where 절로 해결하고, 정말 외부조인이 필요한 경우에만 이 기능을 사용하자.
> 

### 연관관계 없는 엔티티 외부 조인

ex) 회원의 이름과 팀의 이름이 같은 대상 **외부 조인**

```java
/**
     * 2. 연관관계 없는 엔티티 외부 조인
     * 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인
     * JPQL: SELECT m, t FROM Member m LEFT JOIN Team t on m.username = t.name
     * SQL: SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.username = t.name
     */
    @Test
    public void join_on_no_relation() throws Exception {
        em.persist(new Member("teamA"));
        em.persist(new Member("teamB"));
        List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(team).on(member.username.eq(team.name))
                .fetch();
        for (Tuple tuple : result) {
            System.out.println("t=" + tuple);
        }
    }

// 결과
t=[Member(id=3, username=member1, age=10), null]
t=[Member(id=4, username=member2, age=20), null]
t=[Member(id=5, username=member3, age=30), null]
t=[Member(id=6, username=member4, age=40), null]
t=[Member(id=7, username=teamA, age=0), Team(id=1, name=teamA)]
t=[Member(id=8, username=teamB, age=0), Team(id=2, name=teamB)]
```

- 문법 주의 **leftJoin**() 부분에 일반 조인과 다르게 엔티티 하나만 들어간다.
    - 일반조인 : leftJoin(member.team, team)
    - on 조인 : from(member).leftJoin(team).on(xxx)

### 조인 - 페치 조인

페치 조인은 SQL에서 제공하는 기능은 아니다. SQL 조인을 활용해서 연관된 엔티티를 SQL 한번에 조회하는 기능이다. 주로 성능 최적화에 사용하는 방법이다.

### 페치 조인 미적용

지연로딩으로 Member, Team SQL 쿼리 각각 실행

```java
 		@PersistenceUnit
    EntityManagerFactory emf;

    @Test
    public void fetchJoinNo() throws Exception {
        em.flush();
        em.clear();
        Member findMember = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1"))
                .fetchOne();
        boolean loaded =
                emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
        assertThat(loaded).as("페치 조인 미적용").isFalse();
    }
```

### 페치 조인 적용

즉시로딩으로 Member, Team SQL 쿼리 조인으로 한번에 조회

```java
@Test
    public void fetchJoinUse() throws Exception {
        em.flush();
        em.clear();

        Member findMember = queryFactory
                .selectFrom(member)
                .join(member.team, team).fetchJoin()
                .where(member.username.eq("member1"))
                .fetchOne();

        boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
        assertThat(loaded).as("페치 조인 적용").isTrue();
    }
```

- join(), leftJoin() 등 조인 기능 뒤에 fetchJoin() 이라고 추가하면 된다.

> 참고 : 페치 조인에 대한 자세한 내용은 JPA 기본편이나, 활용 2편을 참고하자
> 

### 서브 쿼리

com.querydsl.jpa.JPAExpressions 사용

서브 쿼리 eq 사용

```java
    /*
    * 나이가 가장 많은 회원 조회
    * */
    @Test
    public void subQuery() throws Exception {
        QMember memberSub = new QMember("memberSub");
        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(
                        JPAExpressions
                                .select(memberSub.age.max())
                                .from(memberSub)
                ))
                .fetch();
        assertThat(result).extracting("age")
                .containsExactly(40);
    }
```

```java
/*
    *  나이가 평균 나이 이상인 회원
    * */
    @Test
    public void subQueryGoe() throws Exception {
        QMember memberSub = new QMember("memberSub");

        List<Member> result = queryFactory.
                selectFrom(member)
                .where(member.age.goe(
                        JPAExpressions
                                .select(memberSub.age.avg())
                                .from(memberSub)
                ))
                .fetch();
        assertThat(result).extracting("age")
                .containsExactly(30, 40);
    }
```

서브쿼리 여러 건 처리, in 사용

```java
/*
    * 서브쿼리 여러 건 처리, in 사용
    * */
    @Test
    public void subQueryIn() throws Exception {
        QMember memberSub = new QMember("memberSub");

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.in(
                        JPAExpressions
                                .select(memberSub.age)
                                .from(memberSub)
                                .where(memberSub.age.gt(10))
                ))
                .fetch();
        assertThat(result).extracting("age")
                .containsExactly(20, 30, 40);
    }
```

select 절에 subquery

```java
// select 절에 subquery
    @Test
    public void selectSubQuery(){
        QMember memberSub = new QMember("memberSub");
        List<Tuple> result = queryFactory
                .select(member.username,
                        JPAExpressions
                                .select(memberSub.age.avg())
                                .from(memberSub)
                ).from(member)
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }
```