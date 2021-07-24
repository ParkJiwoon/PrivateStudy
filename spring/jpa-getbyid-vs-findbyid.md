# JPA 의 getById vs findById

# 1. Overview

JPA 를 사용할 때 `ID` 값으로 엔티티를 가져오는 두 가지 메소드가 존재합니다.

비슷하지만 다른 이 두가지 메소드의 차이점에 대해서 알아봅시다.

<br>

## 1.1. getById

```java
@Override
public T getById(ID id) {

    Assert.notNull(id, ID_MUST_NOT_BE_NULL);
    return em.getReference(getDomainClass(), id);
}
```

`getById()` 는 원래 `getOne()` 이었으나 해당 메소드가 Deprecated 되고 대체되었습니다.

내부적으로 `EntityManager.getReference()` 메소드를 호출하기 때문에 엔티티를 직접 반환하는게 아니라 프록시만 반환합니다.

프록시만 반환하기 때문에 실제로 사용하기 전까지는 DB 에 접근하지 않으며, 만약 나중에 프록시에서 DB 에 접근하려고 할 때 데이터가 없다면 `EntityNotFoundException` 이 발생합니다.

<br>

## 1.2. findById

```java
@Override
public Optional<T> findById(ID id) {

    Assert.notNull(id, ID_MUST_NOT_BE_NULL);

    Class<T> domainType = getDomainClass();

    if (metadata == null) {
        return Optional.ofNullable(em.find(domainType, id));
    }

    LockModeType type = metadata.getLockModeType();

    Map<String, Object> hints = new HashMap<>();
    getQueryHints().withFetchGraphs(em).forEach(hints::put);

    return Optional.ofNullable(type == null ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints));
}
```

우리가 잘 알고 있는 메소드입니다.

실제 DB 에 요청해서 엔티티를 가져옵니다.

정확히 말하면 영속성 컨텍스트의 1차 캐시를 먼저 확인하고 데이터가 없으면 실제 DB 에 데이터가 있는지 확인합니다.

<br>

# 2. 차이점

`getById()` 는 해당 엔티티를 사용하기 전까진 DB 에 접근하지 않기 때문에 성능상으로 좀더 유리합니다.

따라서 특정 엔티티의 ID 값만 활용할 일이 있다면 DB 에 접근하지 않고 프록시만 가져와서 사용할 수 있습니다.

<br>

# 3. Test

실제로 테스트 하면서 SQL 쿼리문이 어떻게 날라가는지 확인해봅니다.

<br>

## 3.1. Domain

```java
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;
}
```

```java
@Entity
@Setter
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "post_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;
}
```

- `Member` 엔티티와 `Post` 엔티티를 정의합니다.
- `Member` 는 `Post` 를 작성할 수 있는 관계입니다.

<br>

## 3.2. Service Layer

```java
@Service
@Transactional
@RequiredArgsConstructor
public class PostService {

    private final MemberRepository memberRepository;
    private final PostRepository postRepository;

    // Member 생성
    public Member createMember() {
        Member member = new Member();
        return memberRepository.save(member);
    }

    // getById 로 Member 를 가져와서 Post 생성
    public void createPostByGet(Long memberId) {
        Member member = memberRepository.getById(memberId);
        Post post = new Post();
        post.setMember(member);
        postRepository.save(post);
    }

    // findById 로 Member 를 가져와서 Post 생성
    public void createPostByFind(Long memberId) {
        Member member = memberRepository.findById(memberId).get();
        Post post = new Post();
        post.setMember(member);
        postRepository.save(post);
    }
}
```

- `getById()` 를 사용하는 버전과 `findById()` 를 사용하는 두 가지 버전의 메소드를 정의합니다.

<br>

## 3.3. Test Code

```java
@SpringBootTest
public class PostServiceTest {

    @Autowired
    private PostService postService;

    @Test
    void testGetById() {
        Member member = postService.createMember();
        postService.createPostByGet(member.getId());
    }

    @Test
    void testFindById() {
        Member member = postService.createMember();
        postService.createPostByFind(member.getId());
    }
}
```

- 두 개의 메소드를 따로 실행해서 SQL 쿼리문을 확인해본다.

<br>

## 3.4. SQL Query

```sql
-- getById() 사용한 경우 member 테이블 조회하는 쿼리가 날아가지 않음

Hibernate: 
    insert 
    into
        member
        (member_id) 
    values
        (null)
Hibernate: 
    insert 
    into
        post
        (post_id, member_id) 
    values
        (null, ?)
```

- `getById()` 는 실제 테이블을 조회하는 대신 프록시 객체만 가져옵니다.
- 프록시 객체만 있는 경우 ID 값을 제외한 나머지 값을 사용하기 전까지는 실제 DB 에 액세스 하지 않기 때문에 SELECT 쿼리가 날아가지 않습니다.

<br>

```sql
-- findById() 사용한 경우 post 테이블을 조회

Hibernate: 
    insert 
    into
        member
        (member_id) 
    values
        (null)
Hibernate: 
    select
        member0_.member_id as member_i1_2_0_ 
    from
        member member0_ 
    where
        member0_.member_id=?
Hibernate: 
    insert 
    into
        post
        (post_id, member_id) 
    values
        (null, ?)
```

- `findById()` 를 사용하면 DB 에 바로 액세스해서 데이터를 가져옵니다.

<br>

# 4. Conclusion

실제 DB 에 접근하냐 하지 않냐는 성능에 영향이 갈 수 있습니다.

위의 예시처럼 단순히 특정 엔티티의 ID 값만 필요한 경우에는 모든 데이터를 가져올 필요가 없습니다.

따라서 상황에 따라 적절한 메소드를 사용하면 됩니다.

<br>

# Reference

- [Difference between getOne and findById in Spring Data JPA?](https://www.javacodemonk.com/difference-between-getone-and-findbyid-in-spring-data-jpa-3a96c3ff)