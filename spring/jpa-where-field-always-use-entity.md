# Overview

토이 프로젝트를 하면서 개발 초반에 작성했던 코드를 보았습니다.

JPA 메서드로 조건을 걸어서 가져오는 로직인데 실제로 쿼리가 날라가는 걸 확인해보니 제가 생각한 것과 다르게 동작하는 걸 알게 되었습니다.

<br>

## 1. Entity

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    @Column
    private String email;

    @OneToMany(mappedBy = "member")
    private List<Post> posts = new ArrayList<>();
}


@Entity
public class Post {

    @Id @GeneratedValue
    @Column(name = "post_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @Column
    private String content;
}
```

일반적인 두 개의 Entity 가 존재합니다.

사용자 `Member` 는 여러 개의 게시글 `Post` 를 작성할 수 있으므로 1:N 관계입니다.

<br>

## 2. Repository

```java
public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findAllByMember(Member member);
    List<Post> findAllByMemberId(Long memberId);
}
```

이제 `post` 테이블에서 `post.member_id` 를 기준으로 일치하는 데이터들을 가져오는 메서드를 작성합니다.

가능한 메서드는 두 종류가 있습니다.

1. `Member` 라는 엔티티를 조건으로 검색하는 `findAllByMember`
2. `memberId` 라는 필드값을 조건으로 검색하는 `findAllByMemberId`

결론부터 말하자면 1 번 메서드를 사용해야 합니다.

<br>

## 3. Test

두 메서드를 사용했을 때 실제로 쿼리가 어떻게 날라가는 지 확인해보겠습니다.

<br>

### 3.1. Member ID 로 조회

```java
@Test
void testFindPosts() {
    Long memberId = 2L;
    postRepository.findAllByMemberId(memberId);
}
```

제가 처음에 작성했던 코드입니다.

- 이유는 단순하게 `memberId` 값은 이미 알고 있는 데 굳이 `Member` 엔티티를 한번 조회해야 할 필요가 있을까??
- `Post` 에도 `member_id` 라는 필드가 있으니까 바로 조회하자!

라는 생각으로 사용했습니다.

`SELECT * FROM post WHERE post.member_id = ?` 를 기대했지만 실제로 날라가는 쿼리는 달랐습니다.

<br>

```sql
select
    post0_.post_id as post_id1_5_,
    post0_.content as content5_5_,
    post0_.member_id as member_i8_5_,
from
    post post0_ 
left outer join
    member member1_ 
        on post0_.member_id=member1_.member_id 
where
    member1_.member_id=?
```

예상과는 달리 `LEFT OUTER JOIN` 쿼리가 발생합니다.

<br>

### 3.2. Member 엔티티로 조회

```java
@Test
void testFindPosts() {
    Long memberId = 2L;
    Member member = memberRepository.findById(memberId).get();
    postRepository.findAllByMember(member);
}
```

`Member` 를 한번 조회한 후에 조건으로 엔티티를 넣어봅니다.

<br>

```sql
select
    post0_.post_id as post_id1_5_,
    post0_.content as content5_5_,
    post0_.member_id as member_i8_5_,
from
    post post0_ 
where
    post0_.member_id=?
```

우리가 원하던 쿼리가 정상적으로 날라갑니다.

<br>

### 3.3. Query 를 직접 짜서 조회

```java
@Query(value = "SELECT p FROM Post p WHERE p.member.id = :memberId")
List<Post> findAllByMemberIdQuery(@Param("memberId") Long memberId);
```

만약 극한의 성능 최적화를 해야 한다면 이렇게 직접 쿼리를 짜는 방법도 있습니다.

정상적으로 동작합니다.

<br>

# Conclusion

처음에는 단순하게 `memberId` 정보를 내가 갖고 있고 `Post` 엔티티에도 `memberId` 필드가 존재하는데 바로 조회하면 되지 않을까?

하는 생각에서 코드를 짰었습니다.

그런데 실제로 쿼리가 날라가는 걸 확인하니 비효율적으로 동작하고 있었던 걸 알 수 있었네요.

보통 `Member` 데이터에 대해 검증을 하기 위해 조회를 한번 하니 쿼리 한번 아깝다고 생각하지 말고 엔티티로 조건을 거는 게 좋아보입니다.

**JPA 에서  `@ManyToOne` 인 필드로 조건을 걸 때는 항상 Entity 를 사용하자**

**극한의 성능 최적화가 필요하다면 JPQL 이나 QueryDSL 로 직접 짜자**