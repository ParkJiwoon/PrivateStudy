# JPA 1 + N 문제와 페이징 처리

# Overview

토이 프로젝트로 인스타그램 클론 코딩을 진행 중인데 여기서 1 + N 문제를 만났습니다.

인스타그램에 접속했을 때 사용자에게 보여주는 첫 화면은 **내가 팔로우 중인 사람들의 최신 게시글 목록**입니다.

내가 팔로우 중인 사람은 여러명이고 또 그 여러명이 여러개의 게시글을 작성할 수 있으므로 1 + N 문제가 발생합니다.

<br>

# 1. Domain

사용 중인 Entity 를 간단히 정의합니다.

자잘한 건 생략하고 `Member`, `Post`, `Follow` 세 개의 엔티티만 정의하겠습니다.

<br>

## 1.1. Member

```java
@Table(name = "member")
@Entity
@Getter
@NoArgsConstructor
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    private String email;

    @OneToMany(mappedBy = "member")
    private List<Post> posts = new ArrayList<>();

    @OneToMany(mappedBy = "fromMember")
    private List<Follow> followings = new ArrayList<>();

    @OneToMany(mappedBy = "toMember")
    private List<Follow> followers = new ArrayList<>();
}
```

- `email` 정보만 갖고 있는 간단한 `Member` 엔티티입니다.
- 멤버는 여러 개의 게시글을 가질 수 있고, 여러명을 팔로우 할 수 있으며 반대로 여러 명에게 팔로우 당할 수 있으므로 전부 `@OneToMany` 로 매핑해줍니다.

<br>

## 1.2. Follow

```java
@Table(name = "follow")
@Entity
@Getter
@NoArgsConstructor
public class Follow {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "follow_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "from_member_id")
    private Member fromMember;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "to_member_id")
    private Member toMember;
}
```

- `Follow` 에는 팔로우 하는 사람 (from) 과 팔로우 대상 (to) 의 Member ID 만 존재합니다.
- 사용자는 팔로우라는 액션에 대해서 N:M 관계입니다.
- 그래서 `Follow` 라는 관계 테이블을 하나 생성해서 `@ManyToOne` 으로 매핑해줍니다.

<br>

## 1.3. Post

```java
@Table(name = "post")
@Entity
@Getter
@NoArgsConstructor
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "post_id")
    private Long id;

    @Column(columnDefinition = "text")
    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;
}
```

- `content` 정보만 갖고 있는 `Post` 엔티티입니다.
- 멤버 한명은 여러 개의 게시글을 작성할 수 있기 때문에 `@ManyToOne` 으로 매핑해줬습니다.

<br>
<br>

# 2. 명세

제가 구하고자 하는건 인스타 피드입니다.

로그인 한 내 정보를 갖고 있고, 내가 팔로우한 사용자들을 `follow` 테이블에서 가져와야 합니다.

그리고 그 사용자들의 `member_id` 와 일치하는 데이터들을 `post` 테이블에서 뽑아야 합니다.

하지만 내가 팔로우 하는 대상이 엄청 많고 또 그 사람들이 작성한 글도 엄청 많다면 어떻게 될까요?

클라이언트에게 데이터를 내려주는 것은 물론이고 DB 에서 조회하는 것부터 오래걸릴 겁니다.

따라서 모든 데이터를 한번에 내려주지 말고 당장 필요한 데이터만 짤라서 전달해주는 페이지네이션 기법이 필요합니다.

인스타는 페이지 형식이 아니라 무한 스크롤 형식이므로 `lastPostId` 를 받아와서 offset 으로 사용하고 `Post` 데이터는 최대 5 개만 내려주도록 구현하려고 합니다.

<br>

## 2.1. 단순하게 Entity Collection 호출

```java
@Service
@RequiredArgsConstructor
public class PostService {
    public List<Post> getFeeds(Long lastPostId) {
        Member currentMember = getCurrentMember();

        List<Post> posts = new ArrayList<>();

        // 내가 팔로우 하는 "모든" 대상들이 작성한 "모든" 게시글을 가져옴
        currentMember.getFollowings()
                .stream()
                .map(Follow::getToMember)
                .forEach(member -> posts.addAll(member.getPosts()));

        return posts.stream()
                .filter(post -> post.getId() < lastPostId)
                .sorted((a, b) -> (int) (b.getId() - a.getId()))
                .limit(5L)
                .collect(Collectors.toList());
    }
}
```

- 가장 심플하게 조회하는 방법입니다.
- `currentMember()` 는 로그인 된 내 정보를 가져옵니다.
- 내가 팔로우 하고 있는 대상들을 `getFollowings()` 메서드로 구합니다.
- 멤버 정보를 조회하여 `getPosts()` 메서드로 게시글들을 가져옵니다.
- 구한 모든 게시글들을 필터링 하고 최신순으로 만큼만 잘라서 리턴합니다.

<br>

```sql
-- 팔로우 하는 모든 대상 구하기
SELECT * FROM follow WHERE follow.from_member_id = ?

-- 첫 번째 팔로우 유저의 정보를 가져와서 게시글 가져오기
SELECT * FROM member WHERE member.member_id = ?
SELECT * FROM post WHERE post.member_id = ?

-- 두 번째 팔로우 유저의 정보를 가져와서 게시글 가져오기
SELECT * FROM member WHERE member.member_id = ?
SELECT * FROM post WHERE post.member_id = ?

-- 팔로우 대상들 만큼 반복..
-- ...
```

- 내가 팔로우 중인 대상의 수 * 2 만큼 추가 쿼리가 나갑니다.

<br>

## 2.2. Fetch Join

1 + N 문제를 해결하기 위한 가장 일반적인 방법입니다.

여러 개의 쿼리를 날리지 않고 쿼리 한번에 데이터를 가져옵니다.

<br>

### 2.2.1. 사용

```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {

    @Query(value = "SELECT p" +
            " FROM Post p" +
            " JOIN FETCH p.member m" +
            " JOIN FETCH m.followers f" +
            " WHERE f.fromMember.id = :memberId AND p.id < :lastPostId")
    List<Post> findByFetchJoin(@Param("memberId") Long memberId, @Param("lastPostId") Long lastPostId, Pageable pageable);
}
```

- 쿼리를 날리는 `PostRepository` 입니다.
- 인스타 피드 조건에 맞는 데이터를 쿼리로 한번에 뽑아오기 때문에 추가 쿼리가 발생하지 않습니다.
- Fetch Join 은 연관된 테이블끼리만 사용 가능한데, `Follow` 와 `Post` 는 직접적인 연관 관계가 없기 때문에 `Member` 까지 같이 조인해줘야 합니다.

<br>

```java
@Service
@RequiredArgsConstructor
public class PostService {
    private final PostRepository postRepository;

    public List<Post> getFeeds(Long lastPostId) {
        Member currentMember = getCurrentMember();
        PageRequest pageRequest = PageRequest.of(0, 5, Sort.by("id").descending());

        return postRepository.findByFetchJoin(currentMember.getId(), lastPostId, pageRequest);
    }
}
```

- `PageRequest.of(0, 5)` 을 사용해서 일정한 사이즈 만큼의 데이터를 가져옵니다.
- 무한 스크롤은 `lastPostId` 를 사용해서 데이터를 필터링 하기 때문에 무조건 0 번째 페이지를 가져오도록 설정했습니다.

<br>

### 2.2.2. Fetch Join Paging 결과가 실제로는 어떻게 나올까?

```html
o.h.h.internal.ast.QueryTranslatorImpl   : HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
```

- Fetch Join 과 `Pageable` 을 함께 사용하면 나오는 로그입니다.
- 둘을 같이 사용하면 LIMIT 쿼리가 제대로 적용되지 않습니다.
- DB 에서 Fetch Join 한 결과물을 모두 가져온 후 애플리케이션 메모리에서 직접 골라내기 때문에 데이터 수가 많다면 OutOfMemory 에러가 발생할 가능성이 높아집니다.

<br>

```sql
SELECT *
FROM post
INNER JOIN member ON post.member_id = member.member_id
INNER JOIN follow ON member.member_id = follow.to_member_id
WHERE follow.from_member_id = ? AND post.post_id < ?
ORDER BY post.post_id DESC
```

- 실제로 날라가는 쿼리입니다.
- 분명 `Pageable` 을 사용했음에도 불구하고 LIMIT 조건이 추가되지 않는 것을 볼 수 있습니다.
- 왜 이런 결과가 발생하는지는.. 구글링을 통해서 확인하실 수 있습니다.
- 어쨌뜬 **Fetch Join 과 Pageable 은 함께 사용 불가능**합니다.

<br>

## 2.3. IN 조회

`post` 테이블에 `member_id` 가 존재하는데 굳이 테이블 조인을 해야하나?

해당하는 `member_id` 리스트를 구한 다음에 `IN` 쿼리를 쓰면 조인할 필요도 없고 간단하지 않을까?

하는 생각에서 시도해봤습니다.

<br>

```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findByIdLessThanAndMemberIn(Long lastPostId, List<Member> members, Pageable pageable);
}
```

- JPA 메서드로 한번에 작성했습니다.
- `ByIdLessThan` : `lastPostId` 보다 Post ID 값이 작은 게시글들만 가져옵니다.
- `MemberIn` : `post.member_id` 기준으로 `IN` 쿼리를 추가합니다.

<br>

```java
@RequiredArgsConstructor
@Service
public class PostService {
    private final PostRepository postRepository;

    public List<Post> getFeeds(Long lastPostId) {
        Member currentMember = getCurrentMember();

        List<Member> followings = currentMember.getFollowings()
                .stream()
                .map(Follow::getToMember)
                .collect(Collectors.toList());

        PageRequest pageRequest = PageRequest.of(0, 5, Sort.by("id").descending());
        return postRepository.findByIdLessThanAndMemberIn(lastPostId, followings, pageRequest);
    }
}
```

- `getFollowings()` 로 팔로우 대상들을 먼저 가져옵니다.
- `post` 테이블을 조회할 때 대상 ID 들을 `IN` 쿼리에 넣어줍니다.

<br>

```sql
-- 팔로우 대상들 가져오기
SELECT * FROM follow WHERE follow.from_member_id = ?

-- IN 쿼리를 사용해서 팔로우 대상들이 작성한 게시글들 전부 가져오기
SELECT * 
FROM post 
WHERE post.post_id < ? AND post.member_id IN (?, ?, ...) 
ORDER BY post.post_id DESC 
LIMIT ?
```

- 쿼리 한 두번에 데이터를 모두 가져오고 LIMIT 쿼리도 정상적으로 날라갑니다.
- `IN` 쿼리 조건 컬럼에 인덱스만 걸려있다면 성능도 보장됩니다.

<br>

쿼리만 보면 성능적으로 많이 개선되었습니다.

그러나 여기엔 한 가지 함정이 있습니다.

만약 `followings` 의 사이즈가 엄청나게 많다면??

`IN` 쿼리는 분명 효율적이긴 하지만 갯수가 1000 이 넘어가면 역시 성능적인 문제를 피할 수 없습니다.

그래도 `IN` 쿼리를 사용하겠다면 임의로 1000 개씩 자른 후 나누어서 호출해야 합니다.

<br>

## 2.4. Join + Limit

위 코드에서 말했던 것처럼 `IN` 쿼리를 사용하기 위해 구한 `followings` 값 역시 엄청나게 큰 값이 될 수 있습니다.

따라서 다른 방법을 생각해봐야 하는데, 제가 실무에서 JPA 를 사용하지 않으니 어떤 방법으로 해야 할 지 감이 오지 않았습니다.

그래서 인프런에서 JPA 권위자이신 김영한 님에게 질문을 드렸고 답변을 받았습니다.

솔루션은 JPQL 로 Join, Limit 조건을 작성해서 직접 `post` 테이블을 조회하는 거였습니다.

JPA 에서 Fetch Join 은 Pageable 사용이 불가능하지만 일반 Join 은 사용할 수 있습니다.

사실 생각해보면 단순한 거였는데 바보 같이 네이티브 쿼리를 짜지 않는 방법만 찾다 보니 멀리 돌아왔습니다.

<br>

```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
    @Query(value = "SELECT p" +
            " FROM Post p" +
            " JOIN Follow f" +
            " ON p.member.id = f.toMember.id" +
            " WHERE f.fromMember.id = :memberId AND p.id < :lastPostId")
    List<Post> findByJoinFollow(@Param("memberId") Long memberId, @Param("lastPostId") Long lastPostId, Pageable pageable);
}
```

- JPQL 로 직접 짰는데 QueryDSL 을 적용하면 좀더 이쁘게 나올 것 같네요.

<br>

```java
@RequiredArgsConstructor
@Service
public class PostService {
    private final PostRepository postRepository;

    public List<Post> getFeeds(Long lastPostId) {
        PageRequest pageRequest = PageRequest.of(0, 5, Sort.by("id").descending());
        return postRepository.findByJoinFollow(getCurrentMember().getId(), lastPostId, pageRequest);
    }
}
```

- 쿼리는 다 짜놓았기 때문에 호출만 하면 됩니다.

<br>

```sql
SELECT *
FROM post
INNER JOIN follow ON post.member_id = follow.to_member_id
WHERE post.post_id < ? AND follow.from_member_id = ?
ORDER BY post.post_id DESC
LIMIT ?
```

- 단 하나의 쿼리로 원하는 데이터를 가져옵니다 !

<br>

# Conclusion

`@OneToMany` 조건이 걸려있는 컬렉션을 호출할 때는 성능상 문제가 없는 지 고민이 필요합니다.

만약 데이터가 너무 많다면 Limit 조건을 추가해서 DB 에서 가져오는 데이터 크기를 조절해야 합니다.

이런 경우 1 + N 문제의 일반적인 해결책인 Fetch Join 이나 Batch Size 를 사용할 수 없었고 결국 네이티브 쿼리로 해결했습니다.

네이티브 쿼리를 짜지 않고 Entity 내에서 해결하는 것이 JPA 스럽게 작성하는 거라고 생각했는데 잘못 생각하고 있었던 것 같습니다.

성능 최적화를 위해선 복잡한 쿼리도 작성할 수 밖에 없다는 점을 깨달았네요.

JPQL 로 짜고 보니 QueryDSL 의 필요성을 느끼게 되었고, 귀찮은 질문에도 친절하게 답변해주신 영한님의 강의를 구매했습니다.