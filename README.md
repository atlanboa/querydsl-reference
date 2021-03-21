# querydsl-reference

# QueryDSL 설정

```groovy
plugins {
   id 'org.springframework.boot' version '2.4.4'
   id 'io.spring.dependency-management' version '1.0.11.RELEASE'
   //querydsl 추가
   id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
   id 'java'
}

group = 'study'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
   compileOnly {
      extendsFrom annotationProcessor
   }
}

repositories {
   mavenCentral()
}

dependencies {
   implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
   implementation 'org.springframework.boot:spring-boot-starter-web'

   //querydsl 추가
   implementation 'com.querydsl:querydsl-jpa'

   compileOnly 'org.projectlombok:lombok'
   runtimeOnly 'com.h2database:h2'
   annotationProcessor 'org.projectlombok:lombok'
   testImplementation 'org.springframework.boot:spring-boot-starter-test'


}

test {
   useJUnitPlatform()
}

//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
   jpa = true
   querydslSourcesDir = querydslDir
}
sourceSets {
   main.java.srcDir querydslDir
}
configurations {
   querydsl.extendsFrom compileClasspath }
compileQuerydsl {
   options.annotationProcessorPath = configurations.querydsl
}
```





# Generated 된 Q 파일은?

git 에서 관리하면 안됌.

이걸 세팅하는 여러가지 방법이 있음.

빌드 파일 밑에 생성하면 기본으로 .gitignore 에 포함되어 있음으로 이 방법이 권장.





# QueryDSL 실습

queryDSL 과 관련된 작업을 할 때 엔티티를 사용하는 것이 아니라 Q 클래스를 사용해야 함.

실행되는지 확인.

```java
package study.querydsl;

import com.querydsl.jpa.impl.JPAQueryFactory;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;
import study.querydsl.entity.Hello;
import study.querydsl.entity.QHello;

import javax.persistence.EntityManager;

@SpringBootTest
@Transactional
class QuerydslApplicationTests {

   @Autowired
   EntityManager em;

   @Test
   void contextLoads() {
      Hello hello = new Hello();
      em.persist(hello);

      JPAQueryFactory query = new JPAQueryFactory(em);
      QHello qHello = QHello.hello;

      Hello result = query
            .selectFrom(qHello)
            .fetchOne();

      Assertions.assertThat(result).isEqualTo(hello);
      Assertions.assertThat(result.getId()).isEqualTo(hello.getId());

   }

}
```

# QueryDSL 라이브러리





# 학습 진행할 때



### @Commit 어노테이션

테스트하며 데이터를 넣어보던 중, 롤백을 안시키고 데이터를 남기는 방법





# JPQL vs QueryDSL

```java
@Test
public void startJPQL() {
    String qlString =
            "select m from Member m " +
            "where m.username = :username";
    Member findMember = em.createQuery(qlString, Member.class)
            .setParameter("username", "member1")
            .getSingleResult();
    assertThat(findMember.getUsername()).isEqualTo("member1");
}

@Test
public void startQuerydsl() {
    JPAQueryFactory queryFactory = new JPAQueryFactory(em);
    // 어떤 QMember 인지 구분하는 이름을 부여.
    QMember m = new QMember("m");

    Member findMember = queryFactory
            .select(m)
            .from(m)
            .where(m.username.eq("member1"))
            .fetchOne();

    assertThat(findMember.getUsername()).isEqualTo("member1");

}
```

QueryDSL 은 컴파일러에서 에러가 다 검출됩니다.

파라미터 바인딩이 자동으로 된다.

code assistance 가 어마어마합니다.

동시성 문제에서 안전합니다.



# QType 활용

```java
QMember qMember = new QMember("m"); //별칭 직접 지정 
QMember qMember = QMember.member; //기본 인스턴스 사용
```



### 별칭을 사용하는 이유?

별칭을 직접 지정하면 JPQL의 alias 로 사용됩니다.

사용할 일이 거의 없지만, 간혹 같은 테이블을 조인해야 되는 경우가 있는데 이 때 별칭을 다르게 지정해줘야 합니다.



# QueryDSL 검색 조건

```text
member.username.eq("member1"); // username = 'member1' 

member.username.ne("member1"); //username != 'member1' 

member.username.eq("member1").not(); // username != 'member1'

member.username.isNotNull(); //이름이 is not null

member.age.in(10, 20); // age in (10,20) 

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





# 결과 조회

```java
@Test
public void resultFetch() {
    // 리스트 조회
    List<Member> fetch = queryFactory
            .selectFrom(member)
            .fetch();

    // 단건 조회
    Member fetchOne = queryFactory
            .selectFrom(QMember.member)
            .fetchOne();

    // 처음 한 건 조회 == limit(1).fetchOne()
    Member fetchFirst = queryFactory
            .selectFrom(QMember.member)
            .fetchFirst();


    // 페이징에서 사용
    QueryResults<Member> results = queryFactory
            .selectFrom(member)
            .fetchResults();

    // count 쿼리로 변경.
    long total = queryFactory
            .selectFrom(member)
            .fetchCount();

    // fetchResults 같은 경우 fetchResult 가 복잡해지면 컨텐츠를 가져오는 쿼리랑 토탈 카운트를 가져오는 쿼리가 다를때가 있는데 
    // 이런 경우에는 사용하면 안되고, 쿼리를 분리해서 날려야 함.
}
```



# 정렬

```java
/**
 * 회원 정렬 순서
 * 1. 회원 나이 내림차순
 * 2. 회원 이름 오름차순
 * 2에서 회원 이름이 없으면 마지막에 출력.
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



# 페이징

```java
@Test
public void paging1() {
    List<Member> result = queryFactory
            .selectFrom(member)
            .orderBy(member.username.desc())
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
            .fetchResults();

    assertThat(queryResults.getTotal()).isEqualTo(4);
    assertThat(queryResults.getLimit()).isEqualTo(2);
    assertThat(queryResults.getOffset()).isEqualTo(1);
    assertThat(queryResults.getResults().size()).isEqualTo(2);
}
```

카운트 쿼리를 따로 작성해야 되는 경우가 발생한다.

그 경우가 언제냐면 컨텐츠 쿼리는 복잡한데, 카운트 쿼리를 단순하게 할 수 있는 경우 이 경우에 성능 최적화를 위해 분리할 수 있음.



# 집합

```java
@Test
public void aggregation() {
    List<Tuple> result = queryFactory
            .select(
                    member.count(),
                    member.age.sum(),
                    member.age.min(),
                    member.age.max(),
                    member.age.avg())
            .from(member)
            .fetch();

    Tuple tuple = result.get(0);
    assertThat(tuple.get(member.count())).isEqualTo(4);
    assertThat(tuple.get(member.age.sum())).isEqualTo(100);
    assertThat(tuple.get(member.age.max())).isEqualTo(40);
    assertThat(tuple.get(member.age.min())).isEqualTo(10);
    assertThat(tuple.get(member.age.avg())).isEqualTo(25);
}

/**
 * 팀의 이름과 각 팀의 평균 연령을 구해라.
 *
 */
@Test
public void group() {
    List<Tuple> result = queryFactory
            .select(team.name, member.age.avg())
            .from(member)
            .join(member.team, team)
            .groupBy(team.name)
      			// having 절도 사용 가능.
            .fetch();

    Tuple teamA = result.get(0);
    Tuple teamB = result.get(1);

    assertThat(teamA.get(team.name)).isEqualTo("teamA");
    assertThat(teamA.get(member.age.avg())).isEqualTo(15);

    assertThat(teamB.get(team.name)).isEqualTo("teamB");
    assertThat(teamB.get(member.age.avg())).isEqualTo(35);
}
```





# 조인

```java
/**
 * 팀 A 에 소속된 모든 회원
 */
@Test
public void join() {
    // 연관 관계가 있을때는 member.team, team.
    List<Member> result = queryFactory
            .selectFrom(member)
            // innerjoin, leftjoin, rightjoin 모두 가능.
            .join(member.team, team)
            .where(team.name.eq("teamA"))
            .fetch();

    assertThat(result)
            .extracting("username")
            .containsExactly("member1", "member2");
}

/**
 * 세타 조인
 * 회원의 이름이 티미 이름과 같은 회원 조회.
 * 
 * 단순하게 아래 방식을 사용하면 외부 조인이 불가능하나 join on 을 사용하면 외부 조인 가능.
 * 
 */
@Test
public void theta_join() {
    em.persist(new Member("teamA"));
    em.persist(new Member("teamB"));
    em.persist(new Member("teamC"));

    // 연관 관계가 없을때.
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



# 조인 on절

1. 조인 대상 필터링
2. 연관 관계 없는 외부 엔티티 조인



```java
/**
     * 회원과 팀을 조인하면서, 팀 이름이 teamA 인 팀만 조인, 회원은 모두 조회
    */
    @Test
    public void join_on_filtering() {
        List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(member.team, team)
                .on(team.name.eq("teamA"))
                .fetch();

        result.stream().forEach(System.out::println);
    }
```

꼭 외부 조인을 사용해야 할 때만 사용하면 된다.



```java
/**
     * 연관 관계가 없는 엔티티 외부 조인
     * 회원의 이름이 팀 이름과 같은 대상 외부 조인
     *
     */
@Test
public void join_on_no_relation() {
  em.persist(new Member("teamA"));
  em.persist(new Member("teamB"));
  em.persist(new Member("teamC"));

  // 연관 관계가 없을때.
  List<Tuple> result = queryFactory
    .select(member, team)
    .from(member)
    .leftJoin(team).on(member.username.eq(team.name))
    // 위의 코드는 id 매칭을 하지 않는다. id 매칭 조인을 원하는 경우 아래와 같이 해야 한다.
    //.leftJoin(member.team, team).on(member.username.eq(team.name))
    .fetch();

  result.stream().forEach(System.out::println);
}
```



`leftJoin()` : 일반 조인과 다르게 엔티티 하나만 들어간다.

**일반 조인** : leftJoin(member.team, team)

**on 조인** : from(member).leftJoin(team).on(xxx)



# 페치 조인

SQL 조인을 활용해서 연관된 엔티티를 SQL 한번에 조회하는 기능

```java
@PersistenceUnit
EntityManagerFactory emf;

@Test
public void fetchJoinNo() {
    em.flush();
    em.clear();

    Member findMember = queryFactory
            .selectFrom(QMember.member)
            .where(QMember.member.username.eq("member1"))
            .fetchOne();

    boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
    assertThat(loaded).as("페치 조인 미적용").isFalse();

}

@Test
public void fetchJoinUse() {
    em.flush();
    em.clear();

    Member findMember = queryFactory
            .selectFrom(QMember.member)
            .join(member.team, team).fetchJoin()
            .where(QMember.member.username.eq("member1"))
            .fetchOne();

    boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
    assertThat(loaded).as("페치 조인 미적용").isTrue();

}
```

# 서브 쿼리

`com.querydsl.jpa.JPAExpressions` 사용

```java
/**
 * 나이가 가장 많은 회원 조
 */
@Test
public void subQuery() {

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

/**
 * 나이가 평균 이상인 회원
 */
@Test
public void subQueryGoe() {

    QMember memberSub = new QMember("memberSub");

    List<Member> result = queryFactory
            .selectFrom(member)
            .where(member.age.goe(
                    JPAExpressions
                            .select(memberSub.age.avg())
                            .from(memberSub)
            ))
            .fetch();

    assertThat(result).extracting("age")
            .containsExactly(30, 40);
}

/**
 * In Query 예제
 */
@Test
public void subQueryIn() {

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

@Test
public void selectSubQuery() {

    QMember memberSub = new QMember("memberSub");

    List<Tuple> result = queryFactory
            .select(member.username,
                    JPAExpressions
                            .select(memberSub.age.avg())
                            .from(memberSub))
            .from(member)
            .fetch();

    result.stream().forEach(System.out::println);
}
```

**from 절의 서브쿼리 한계**

* JPA JPQL 서브쿼리의 한계점으로  from 절의 서브쿼리(인라인 뷰)는 지원하지 않는다.
* 당연히 Querydsl도 지원하지 않는다. 하이버네이트 구현체를 사용하면 select 절의 서브쿼리는 지원한다.
* Querydsl도 하이버네이트 구현체를 사용하면 select 절의 서브쿼리를 지원한다.

**from 절의 서브쿼리 해결방안**

1. 서브쿼리를 join 으로 변경한다. (가능한 상황이 있고, 안되는 상황이 있음)
2. 애플리케이션 쿼리를 2번 분리해서 실행한다.
3. nativeSQL을 사용한다.



> 화면에 맞춰서 데이터를 넘겨주기보다는, 화면에서 처리할 수 있는 부분들은 화면에서 처리해야 함.
> 쿼리들이 복잡해지는거보다 데이터베이스는 데이터만 필터링하고 그룹핑하고 데이터의 최소화해서 가져오는 역할에 중심을 잡는 것이 좋음.
>
> 실시간으로 보여줘야 하는 데이터가 아닌 이상 쿼리를 나눠서 보내는 것이 더 좋을 수도 있다.



# Case 문

```java
@Test
public void basicCase() {
    List<String> result = queryFactory
            .select(member.age
                    .when(10).then("열살")
                    .when(20).then("스무살")
                    .otherwise("기타"))
            .from(member)
            .fetch();

    result.stream().forEach(System.out::println);

}

@Test
public void complexCase() {
    List<String> result = queryFactory
            .select(new CaseBuilder()
                    .when(member.age.between(0, 20)).then("0 ~ 20살")
                    .when(member.age.between(21, 30)).then("21 ~ 30살")
                    .otherwise("기타"))
            .from(member)
            .fetch();

    result.stream().forEach(System.out::println);
}
```

> 데이터베이스에서는 데이터를 추려내고 가져오는 최소한의 역할만 하는 것. 추가적인 작업이 필요한 경우는 애플리케이션 단에서 처리하는 것이 좋음.



# 상수, 문자 더하기

```java
@Test
public void constant() {
    List<Tuple> result = queryFactory
            .select(member.username, Expressions.constant("A"))
            .from(member)
            .fetch();

    result.stream().forEach(System.out::println);
}

@Test
public void concat() {
    List<String> result = queryFactory
            .select(member.username.concat("_").concat(member.age.stringValue()))
            .from(member)
            .where(member.username.eq("member1"))
            .fetch();

    result.stream().forEach(System.out::println);
}
```

상수가 필요할 때는 Expressions.constant 를 사용

문자열을 이을때는 concat 을 사용.

문자가 아닌 경우는 stringValue() 를 사용.



# 중급 문법





# 프로젝션 결과 반환

tuple은 com.querydsl.core 꺼임

repository 계층에서 tuple 을 사용하는 것은 문제가 없으나 tuple 을 서비스나 컨트롤러 계층까지 가져가는건 올바르지 못한 설계

```java
@Test
public void simpleProjection() {
    List<String> result = queryFactory
            .select(member.username)
            .from(member)
            .fetch();
}

@Test
public void tupleProjection() {
    List<Tuple> result = queryFactory
            .select(member.username, member.age)
            .from(member)
            .fetch();
    for (Tuple tuple : result) {
        String username = tuple.get(member.username);
        Integer age = tuple.get(member.age);
        System.out.println("username = " + username);
        System.out.println("age = " + age);
    }
}
```



# JPQL DTO 반환

JPQL 방식은 패키지 명을 다 적어줘야하는 불편함이 있음.

querydsl 은 이 방식은 해결함.

```java
@Test
public void findDtoByJPQL() {
    List<MemberDto> result = em.createQuery("select new study.querydsl.dto.MemberDto(m.username, m.age) from Member m", MemberDto.class)
            .getResultList();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```

# QueryDSL DTO 반환

3가지 방법이 있다.

bean, fields, constructor 방식은 프로퍼티나 필드명이 일치해야 함.

## bean

setter 를 이용하는 방식

setter 를 이용하기 때문에 기본 생성자가 필요.

```java
@Test
public void findDtoBySetter() {
    List<MemberDto> result = queryFactory
            .select(Projections.bean(MemberDto.class,
                    member.username,
                    member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```



## fields

getter setter 필요없이 필드에 바로 값을 넣어버림.

```java
@Test
public void findDtoByField() {
    List<MemberDto> result = queryFactory
            .select(Projections.fields(MemberDto.class,
                    member.username,
                    member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```



## constructor

생성자를 사용하는 방식.

```java
@Test
public void findDtoByConstructor() {
    List<MemberDto> result = queryFactory
            .select(Projections.constructor(MemberDto.class,
                    member.username,
                    member.age))
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```





## 커스텀 DTO 로 조회하는 방식 + 서브 쿼리 결과 alias

ExpressionUtils 사용.

## field 방식

```java
@Test
public void findUserDto() {
    QMember memberSub = new QMember("memberSub");

    List<UserDto> result = queryFactory
            .select(Projections.fields(UserDto.class,
                    QMember.member.username.as("name"),
                    ExpressionUtils.as(
                            JPAExpressions
                            .select(memberSub.age.max())
                            .from(memberSub), "age"
                    )
            ))
            .from(QMember.member)
            .fetch();

    for (UserDto user : result) {
        System.out.println("memberDto = " + user);
    }
}
```

## constructor 방식

생성자만 있다면 그대로 반환 클래스 타입만 변경해주면 그대로 사용 가능.

이 방식의 문제점은 컴파일 에러가 발생하지 않는다는 것.

따라서 이러한 문제점을 해소하는 방법으로  @QueryProjection 을 사용하는 것이 좋음.

# 프로젝션과 결과 반환 - @QueryProjection

```java
package study.querydsl.dto;

import com.querydsl.core.annotations.QueryProjection;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
public class MemberDto {
    private String username;
    private int age;

    @QueryProjection
    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

@QueryProject 을 Q파일이 생성되길 원하는 대상 클래스의 생성자에 추가해주면 해당 Q파일을 생성해준다.

 컴파일 시점에 타입도 되기에 안전한 방식

> @QueryProjection의 단점
>
> 1. Q파일을 생성해야 하는 것
> 2. DTO 가 QueryDSL 라이브러리의 의존성이 생겨, 순수  DTO 의 개념을 벗어나는 부분이 발생



# 동적 쿼리 - BooleanBuilder, Where

두 가지 방식으로 해결함.

## BooleanBuilder

```java
@Test
public void dynamicQuery_BooleanBuilder() {
    String usernameParam = "member1";
    Integer ageParam = 10;

    List<Member> result = searchMember(usernameParam, ageParam);
    assertThat(result.size()).isEqualTo(1);

}

private List<Member> searchMember(String usernameCond, Integer ageCond) {

    BooleanBuilder builder = new BooleanBuilder();
    if (usernameCond != null) {
        builder.and(member.username.eq(usernameCond));
    }

    if (ageCond != null) {
        builder.and(member.age.eq(ageCond));
    }

    List<Member> result = queryFactory
            .selectFrom(member)
            .where(builder)
            .fetch();
    
    return result;
}
```



## Where

실무에서 자주 사용하는 방식

BooleanBuilder 에 비해서 코드가 명확하고 직관적임.

부가적인 로직 코드에 대해서 깊게 살피지 않고도 코드의 목적을 확인할 수 있음.



```java
@Test
public void dynamicQuery_WhereParam() {
    String usernameParam = "member1";
    Integer ageParam = 10;

    List<Member> result = searchMember2(usernameParam, ageParam);
    assertThat(result.size()).isEqualTo(1);
}

private List<Member> searchMember2(String usernameCond, Integer ageCond) {
    return queryFactory
            .selectFrom(member)
            .where(usernameEq(usernameCond), ageEq(ageCond))
            .fetch();
}

private Predicate usernameEq(String usernameCond) {
    if (usernameCond == null) {
        return null;
    }
    return member.username.eq(usernameCond);
}

private Predicate ageEq(Integer ageCond) {
    if (ageCond == null) {
        return null;
    }
    return member.age.eq(ageCond);
}
```



## 위 코드 좀 더 가독성 높은 코드로 변경

```java
@Test
public void dynamicQuery_WhereParam() {
    String usernameParam = "member1";
    Integer ageParam = 10;

    List<Member> result = searchMember2(usernameParam, ageParam);
    assertThat(result.size()).isEqualTo(1);
}

private List<Member> searchMember2(String usernameCond, Integer ageCond) {
    return queryFactory
            .selectFrom(member)
            .where(allEq(usernameCond, ageCond))
            .fetch();
}

private BooleanExpression usernameEq(String usernameCond) {
    return usernameCond != null ? member.username.eq(usernameCond) : null;
}

private BooleanExpression ageEq(Integer ageCond) {
    return ageCond != null ? member.age.eq(ageCond) : null;
}

private BooleanExpression allEq(String usernameCond, Integer ageCond) {
    return usernameEq(usernameCond).and(ageEq(ageCond));
}
```

이처럼 조건들을 다 분리해서 코드를 작성하면 조건에 대한 재사용성이 매우 높아짐.





# 수정, 삭제 배치 쿼리

쿼리 한번으로 대량 데이터 수정.

주의해야 할 점.

벌크 연산은 영속성 컨텍스트와 상관없이 DB 에 바로 쿼리를 날린다.

그래서 영속성 컨텍스트와 DB의 데이터 불일치가 발생한다.  repeatable read.

벌크 연산이 수행되면 항상 영속성 컨텍스트를 초기화해줘야 함.



```java
@Test
public void bulkUpdate() {


    long count = queryFactory
            .update(member)
            .set(member.username, "비회원")
            .where(member.age.lt(28))
            .execute();

    em.flush();
    em.clear();
    assertThat(count).isEqualTo(2L);

    List<Member> result = queryFactory
            .selectFrom(member)
            .fetch();

    result.forEach(System.out::println);
}


```



```java
@Test
public void bulkAdd() {
    long count = queryFactory
            .update(member)
            .set(member.age, member.age.add(1))
            .execute();
    
    em.flush();
    em.clear();
}

@Test
public void bulkDelete() {
    long count = queryFactory
            .delete(member)
            .where(member.age.gt(18))
            .execute();
    
    em.flush();
    em.clear();
}
```





# SQL function 호출하기

Dialect 에 등록된 내용만 호출이 가능.

sql function 을 추가적으로 사용하고 싶으면  H2Dialect 를 상속받고, 해당하는  Dialect 를 직접 설정파일에 추가해서 등록해야 함.

```java
@Test
public void sqlFunction() {
    List<String> result = queryFactory
            .select(Expressions.stringTemplate(
                    "function('replace', {0}, {1}, {2})", member.username, "member", "M"))
            .from(member)
            .fetch();

    result.forEach(System.out::println);
}

@Test
public void sqlFunction2() {
    queryFactory
            .select(member.username)
            .from(member)
            .where(member.username.eq(member.username.lower()))
            .fetch();
}
```

간단한 sqlFunction은  anscii 표준에 등록되어 있음으로 querydsl 에서도 제공된다.





# 순수  JPA 리포지토리와 Querydsl

JPAQueryFactory 는 생성자로 생성해도 되고, Bean 으로 등록해서 사용해도 된다.

JPAQueryFactory의 동시성 문제는 EntityManager 에 의존하는데, EntityManager 는 동시성 문제가 처리되어 있음.

트랜잭션 단위로 분리해서 동작함.

EntityManager 는 Spring 프레임워크에서 프록시 객체를 주입해주기 때문에 문제가 전혀 없음.



```java
package study.querydsl.repository;

import com.querydsl.jpa.impl.JPAQueryFactory;
import org.springframework.stereotype.Repository;
import study.querydsl.entity.Member;

import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;

import static study.querydsl.entity.QMember.member;

@Repository
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJpaRepository(EntityManager em, JPAQueryFactory jpaQueryFactory) {
        this.em = em;
        this.queryFactory = jpaQueryFactory;
    }

    public void save(Member member) {
        em.persist(member);
    }

    public Optional<Member> findById(Long id) {
        Member findMember = em.find(Member.class, id);
        return Optional.ofNullable(findMember);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findAll_querydsl() {
        return queryFactory
                .selectFrom(member)
                .fetch();
    }

    public List<Member> findByUsername(String username) {
        return em.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", username)
                .getResultList();
    }

    public List<Member> findByUsername_querydsl(String username) {
        return queryFactory
                .selectFrom(member)
                .where(member.username.eq(username))
                .fetch();
    }
}
```



```java
@Autowired
EntityManager em;

@Autowired
MemberJpaRepository memberJpaRepository;

@Test
public void basicTest() {
    Member member = new Member("member1", 10);
    memberJpaRepository.save(member);

    Member findMember = memberJpaRepository.findById(member.getId()).get();
    assertThat(findMember).isEqualTo(member);

    List<Member> result1 = memberJpaRepository.findAll();
    assertThat(result1).containsExactly(member);

    List<Member> result2 = memberJpaRepository.findByUsername("member1");
    assertThat(result2).containsExactly(member);
}

@Test
public void basicTestWithQuerydsl() {
    Member member = new Member("member1", 10);
    memberJpaRepository.save(member);

    Member findMember = memberJpaRepository.findById(member.getId()).get();
    assertThat(findMember).isEqualTo(member);

    List<Member> result1 = memberJpaRepository.findAll_querydsl();
    assertThat(result1).containsExactly(member);

    List<Member> result2 = memberJpaRepository.findByUsername_querydsl("member1");
    assertThat(result2).containsExactly(member);
}
```



# 동적 쿼리와 성능 최적화 조회 - Builder 사용

동적 쿼리를 작성할때, 조건이 아무것도 없는 경우 모든 데이터를 가져온다.

즉, 실무에서 쿼리를 작성할 때 이 부분에 대한 제한이 있도록 하거나, 모든 데이터를 긁어오지 않도록 설정하는 것이 중요하다.



```java
package study.querydsl.dto;

import com.querydsl.core.annotations.QueryProjection;
import lombok.Data;

@Data
public class MemberTeamDto {
    private Long memberId;
    private String username;
    private int age;
    private Long teamId;
    private String teamName;

    @QueryProjection
    public MemberTeamDto(Long memberId, String username, int age, Long teamId, String teamName) {
        this.memberId = memberId;
        this.username = username;
        this.age = age;
        this.teamId = teamId;
        this.teamName = teamName;
    }
}
```



```java
package study.querydsl.dto;

import lombok.Data;

@Data
public class MemberSearchCondition {
    private String username;
    private String teamName;
    private Integer ageGoe;
    private Integer ageLoe;
}
```



```java
public List<MemberTeamDto> searchByBuilder(MemberSearchCondition condition) {

    BooleanBuilder builder = new BooleanBuilder();
    if (hasText(condition.getUsername())) {
        builder.and(member.username.eq(condition.getUsername()));
    }
    if (hasText(condition.getTeamName())) {
        builder.and(team.name.eq(condition.getTeamName()));
    }
    if (condition.getAgeGoe() != null) {
        builder.and(member.age.goe(condition.getAgeGoe()));
    }
    if (condition.getAgeLoe() != null) {
        builder.and(member.age.loe(condition.getAgeLoe()));
    }


    return queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name.as("teamName")
            ))
            .from(member)
            .leftJoin(member.team, team)
            .where(builder)
            .fetch();
}
```



```java
@Test
public void searchTest() {
    Team teamA = new Team("teamA");
    Team teamB = new Team("teamB");
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

    MemberSearchCondition condition = new MemberSearchCondition();
    condition.setAgeGoe(35);
    condition.setAgeLoe(40);
    condition.setTeamName("teamB");

    List<MemberTeamDto> result = memberJpaRepository.searchByBuilder(condition);

    assertThat(result).extracting("username").containsExactly("member4");
}
```



# Where 절 파라미터 방식으로.

코드를 재사용할 수 있다는 점에서 큰 장점, 컨디션 조립이 가능.

```java
public List<MemberTeamDto> search(MemberSearchCondition condition) {
    return queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name.as("teamName")
            ))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe())
                    )
            .fetch();
}

private BooleanExpression usernameEq(String username) {
    return hasText(username) ? member.username.eq(username) : null;
}

private BooleanExpression teamNameEq(String teamName) {
    return hasText(teamName) ? team.name.eq(teamName) : null;
}

private BooleanExpression ageGoe(Integer ageGoe) {
    return ageGoe != null ? member.age.goe(ageGoe) : null;
}

private BooleanExpression ageLoe(Integer ageLoe) {
    return ageLoe != null ? member.age.loe(ageLoe) : null;
}
```





# 스프링 데이터 JPA 와  QueryDSL

1. 사용자 정의 인터페이스 작성
2. 사용자 정의 인터페이스 구현
3. 스프링 데이터 레포지토리에 사용자 정의 인터페이스 상속



### 1. 사용자 정의 인터페이스 작성

```java
package study.querydsl.repository;

import study.querydsl.dto.MemberSearchCondition;
import study.querydsl.dto.MemberTeamDto;

import java.util.List;

public interface MemberRepositoryCustom {
    List<MemberTeamDto> search(MemberSearchCondition condition);
}
```

### 2. 사용자 정의 인터페이스 구현

```java
package study.querydsl.repository;

import com.querydsl.core.types.dsl.BooleanExpression;
import com.querydsl.jpa.impl.JPAQueryFactory;
import study.querydsl.dto.MemberSearchCondition;
import study.querydsl.dto.MemberTeamDto;
import study.querydsl.dto.QMemberTeamDto;

import javax.persistence.EntityManager;
import java.util.List;

import static org.springframework.util.StringUtils.hasText;
import static study.querydsl.entity.QMember.member;
import static study.querydsl.entity.QTeam.team;

public class MemberRepositoryImpl implements MemberRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    @Override
    public List<MemberTeamDto> search(MemberSearchCondition condition) {
        return queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")
                ))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe())
                )
                .fetch();
    }

    private BooleanExpression usernameEq(String username) {
        return hasText(username) ? member.username.eq(username) : null;
    }

    private BooleanExpression teamNameEq(String teamName) {
        return hasText(teamName) ? team.name.eq(teamName) : null;
    }

    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe != null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLoe(Integer ageLoe) {
        return ageLoe != null ? member.age.loe(ageLoe) : null;
    }

}
```

### 3. 스프링 데이터 레포지토리에 상속.

```java
package study.querydsl.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import study.querydsl.entity.Member;

import java.util.List;

public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
    List<Member> findByUsername(String username);
}
```



> **참고사항**
>
> 작성하려는 검색 쿼리가 view 에 특화되어 있다면, 분리해서 구현체를 구현하여 레포지토리로 사용하는 것이 권장된다.
>
> eg. MemberQueryRepository



# 스프링 데이터 페이징 활용

스프링 데이터의  Page, Pageable 을 활용한다.

### 단순 페이지 쿼리

```java
@Override
public Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable) {
    QueryResults<MemberTeamDto> results = queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name.as("teamName")
            ))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe())
            )
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetchResults();

    List<MemberTeamDto> content = results.getResults();
    long total = results.getTotal();

    return new PageImpl<>(content, pageable, total);
}
```



### 복잡한 페이지 쿼리

count query 최적화가 필요한 경우, 데이터가 몇천만건 이상인 경우 count query 의 최적화는 애플리케이션 성능 향상에 상당히 도움이 된다.

```java
@Override
public Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable) {
    // content 쿼리는 복잡하나, count 쿼리는 단순한 경우. count 쿼리 최적화.
    List<MemberTeamDto> content = queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name.as("teamName")
            ))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe())
            )
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();

    long total = queryFactory
            .selectFrom(member)
            .leftJoin(member.team, team)
            .where(usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe()))
            .fetchCount();

    return new PageImpl<>(content, pageable, total);
}
```



# Count Query 최적화

때에 따라 카운트 쿼리를 생략할 수 있다.

1. 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
2. 마지막 페이지 일 때 

```java
@Override
public Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable) {
    // content 쿼리는 복잡하나, count 쿼리는 단순한 경우. count 쿼리 최적화.
    List<MemberTeamDto> content = queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    member.age,
                    team.id.as("teamId"),
                    team.name.as("teamName")
            ))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe())
            )
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();

    JPAQuery<Member> countQuery = queryFactory
            .selectFrom(member)
            .leftJoin(member.team, team)
            .where(usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe()));

    return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchCount);
}
```

위처럼 PageableExecutionUtil.getPage 로 반환하면 카운트 쿼리를 생략 가능한 경우에서는 실행하지 않는다.



> **스프링 데이터 정렬**
>
> 스프링 데이터 JPA 는 자신의 정렬을 Querydsl 의 정렬(OrderSpecifier)로 편리하게 변경하는 기능을 제공한다.
>
> ```java
> JPAQuery<Member> query = queryFactory
>             .selectFrom(member);
> for (Sort.Order o : pageable.getSort()) {
>         PathBuilder pathBuilder = new PathBuilder(member.getType(),
>         member.getMetadata());
>         query.orderBy(new OrderSpecifier(o.isAscending() ? Order.ASC : Order.DESC,
>         pathBuilder.get(o.getProperty())));
> }
> List<Member> result = query.fetch();
> ```



**복잡한 Sort 는 Pageable 의  Sort 기능을 사용하기 어려움으로 파라미터를 받아서 직접 처리하는 것을 권장.**



# 스프링 데이터 JPA 가 제공하는 QueryDSL 기능

여러 기능을 제공하나 실무 환경에는 부적합하다.

join 이 안되기 때문에 실무에서 사용하기 어렵다.

클라이언트 코드가 Querydsl 에 의존해야 함.



### 1. 스프링 데이터 JPA 레포지토리에 extends QuerydslPredicateExecutor\<Entity>

```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom, QuerydslPredicateExecutor<Member> {
    List<Member> findByUsername(String username);
}
```

### 2. 사용 예제

```java
Iterable<Member> result = memberRepository.findAll(
        member.age.between(20, 40).and(member.username.like("member%"))
);
```

단순히 JpaRepository 인터페이스가 제공하는 메소드에 위와 같이 조건들을 넣을 수 있게 된다.



# QueryDsl Web 지원

QueryDsl 이 파라미터에 대한 predicate 매핑을 지원하는데, 코드가 복잡하고 불편하기에 아직까지 사용하기엔 한계가 있으며 권장되지 않음.



# QuerydslRepositorySupport

페이지네이션 관련 코드가 줄어들수 있는 점이 있다.

하지만 Sort 가 안된다.

사용하지 말자.

