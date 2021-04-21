# BatchSize

# Overview

Batch Size 는 Fetch Join 과 함께 1 + N 문제를 해결하는 방법 중 하나입니다.

Batch Size 는 **1 + N 에서 데이터가 많지 않다는 사실이 확실히 보장되는 경우**에 간단한 설정 추가만으로 최적화가 가능합니다.

<br>

# 1. 개념

1 + N 에서의 문제점은 1 개의 쿼리로 가져온 특정 데이터를 조건으로 하여 N 개의 추가 쿼리를 날리는 것이 문제입니다.

여기서 N 개의 쿼리란 일반적으로 `SELECT * FROM aaa WHERE aaa.bbb_id = ?` 와 같은 조회 쿼리죠.

하지만 N 개의 쿼리가 모두 똑같은 쿼리에 `aaa.bbb_id` 조건만 바뀐다면 굳이 이렇게 할 필요가 없습니다.

`SELECT * FROM aaa WHERE aaa.bbb_id IN (?, ?, ...)` 와 같이 IN 쿼리로 바꿔버리면 한번에 모든 데이터를 가져올 수 있습니다.

처음에는 이 개념이 잘 이해되지 않았습니다.

어차피 특정 Member 의 Post 리스트를 구하려는 건데 한번에 모든 데이터를 가져오면 코드에서 또 분리해줘야 하지 않을까? 하는 생각이 들었기 때문이죠.

<br>

# 2. 동작

`@OneToMany` 필드, 즉 `Collection` 에 `@BatchSize(size = 100)` 을 추가하면 엔티티에서 해당 필드를 호출할 때 JPA 가 알아서 최적화해서 값을 가져옵니다.

예를 들어, `Member` 엔티티에서 `getPosts()` 를 호출합니다.

```java
Member member1 = memberRepository.findById(2L).get();
Member member2 = memberRepository.findById(3L).get();

// Posts 를 실제로 사용해야 쿼리가 나가기 때문에 size() 까지 호출
int size1 = member1.getPosts().size();
int size2 = member2.getPosts().size();
```

<br>

위 쿼리에서 `Post` 를 구하는 쿼리는 정직하게 2 번 호출됩니다.

```sql
SELECT * FROM member WHERE member.member_id = 2
SELECT * FROM member WHERE member.member_id = 3

SELECT * FROM post WHERE post.member_id = 2
SELECT * FROM post WHERE post.member_id = 3
```

<br>

만약 `Member` 의 갯수가 2 개가 아니라 100 개라면 어떻게 될까요?

`Post` 엔티티를 구하기 위해 100 번의 호출, 즉 1 + N 문제가 발생합니다.

하지만 `Member` 엔티티에 있는 `@OneToMany` 컬렉션에 배치 사이즈를 걸어둔다면 이 문제를 해결할 수 있습니다.

<br>

```java
@BatchSize(size = 100)
@OneToMany(mappedBy = "member")
private List<Post> posts = new ArrayList<>();
```

이렇게 `@BatchSize` 만 추가하고 처음 코드를 다시 실행시키면..

<br>

```sql
SELECT * FROM member WHERE member.member_id = 2
SELECT * FROM member WHERE member.member_id = 3

SELECT * FROM post WHERE post.member_id IN (2, 3)
```

`post` 를 구할 때는 `IN` 쿼리로 실행하는 것을 확인할 수 있습니다.

참고로 `@BatchSize` 의 `size` 값을 설정한 만큼 `IN` 사이즈를 조절합니다.

위 코드에서는 100 으로 설정했기 때문에 데이터가 250 개라면 1 ~ 100, 101 ~ 200, 201 ~ 250 이렇게 세 번에 나누어서 `IN` 쿼리를 날립니다.

(사실 완전히 똑같은 사이즈로 분배해서 날리지는 않고 내부적으로 최적화한 사이즈로 나누어서 날립니다. 여기서는 깊게 알아보진 않습니다)

<br>

# Conclusion

정리하자면 JPA 의 트랜잭션이 끝나는 순간 DB 에 날리게 되는 `@OneToMany` 컬렉션에 대한 쿼리들을 하나로 모아서 날려주는 편리한 기능입니다.