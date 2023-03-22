# Spring Boot 에서 Redis 사용하기

# 1. Overview

Spring Boot 에서 `spring-data-redis` 라이브러리를 활용해서 Redis 를 사용하는 방법을 알아봅니다.

Redis 에 대한 개념과 로컬 설치 방법은 [Redis 설치 및 명령어](https://bcp0109.tistory.com/327) 글을 확인해주세요.

<br>

# 2. Java 의 Redis Client

Java 의 Redis Client 는 크게 두 가지가 있습니다.

Jedis 와 Lettuce 인데요.

원래 Jedis 를 많이 사용했으나 여러 가지 단점 (멀티 쓰레드 불안정, Pool 한계 등등..) 과 Lettuce 의 장점 (Netty 기반이라 비동기 지원 가능) 때문에 Lettuce 로 추세가 넘어가고 있었습니다.

그러다 결국 Spring Boot 2.0 부터 Jedis 가 기본 클라이언트에서 deprecated 되고 Lettuce 가 탑재되었습니다.

- [Spring Session 에서 Jedis 대신 Lettuce 를 사용하는 이유](https://github.com/spring-projects/spring-session/issues/789) 참고

<br>

# 3. Spring Boot 에서 Redis 설정

Spring Boot 에서 Redis 를 사용하는 방법은 `RedisRepository` 와 `RedisTemplate` 두 가지가 있습니다.

그 전에 먼저 공통 세팅이 필요합니다.

<br>

```json
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

- `build.gradle` 에 `spring-boot-starter-data-redis` 추가하고 빌드해줍니다.

<br>

```yml
spring:
  redis:
    host: localhost
    port: 6379
```

- `application.yaml` 에 host 와 port 를 설정합니다.
- `localhost:6379` 는 기본값이기 때문에 만약 Redis 를 `localhost:6379` 로 띄웠다면 따로 설정하지 않아도 연결이 됩니다.
- 하지만 일반적으로 운영 서버에서는 별도의 Host 를 사용하기 때문에 값을 이렇게 별도의 값을 세팅하고 Configuration 에서 Bean 에 등록해줍니다.

<br>

```java
@Configuration
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(host, port);
    }
}
```

- Redis 사용을 위한 기본 Configuration 입니다.
- `application.yaml` 에 설정한 값을 `@Value` 어노테이션으로 주입합니다.

<br>

## 3.1. RedisRepository

Spring Data Redis 의 Redis Repository 를 이용하면 간단하게 Domain Entity 를 Redis Hash 로 만들 수 있습니다.

다만 트랜잭션을 지원하지 않기 때문에 만약 트랜잭션을 적용하고 싶다면 `RedisTemplate` 을 사용해야 합니다.

<br>

**Entity**

```java
@Getter
@RedisHash(value = "people", timeToLive = 30)
public class Person {

    @Id
    private String id;
    private String name;
    private Integer age;
    private LocalDateTime createdAt;

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
        this.createdAt = LocalDateTime.now();
    }
}
```

- Redis 에 저장할 자료구조인 객체를 정의합니다.
- 일반적인 객체 선언 후 `@RedisHash` 를 붙이면 됩니다.
  - `value` : Redis 의 keyspace 값으로 사용됩니다.
  - `timeToLive` : 만료시간을 seconds 단위로 설정할 수 있습니다. 기본값은 만료시간이 없는 -1L 입니다.
- `@Id` 어노테이션이 붙은 필드가 Redis Key 값이 되며 `null` 로 세팅하면 랜덤값이 설정됩니다.
  - keyspace 와 합쳐져서 레디스에 저장된 최종 키 값은 `keyspace:id` 가 됩니다.

<br>

**Repository**

```java
public interface PersonRedisRepository extends CrudRepository<Person, String> {
}
```

- `CrudRepository` 를 상속받는 `Repository` 클래스 추가합니다.

<br>

**Example**

```java
@SpringBootTest
public class RedisRepositoryTest {

    @Autowired
    private PersonRedisRepository repo;

    @Test
    void test() {
        Person person = new Person("Park", 20);

        // 저장
        repo.save(person);

        // `keyspace:id` 값을 가져옴
        repo.findById(person.getId());

        // Person Entity 의 @RedisHash 에 정의되어 있는 keyspace (people) 에 속한 키의 갯수를 구함
        repo.count();

        // 삭제
        repo.delete(person);
    }
}
```

- JPA 와 동일하게 사용할 수 있습니다.
- 여기서는 `id` 값을 따로 설정하지 않아서 랜덤한 키값이 들어갑니다.
- 저장할때 `save()` 를 사용하고 값을 조회할 때 `findById()` 를 사용합니다.

<br>

**redis-cli 로 데이터 확인**

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/spring-redis-01.png?raw=true" width="80%">

- id 를 따로 설정하지 않은 `null` 값이라 랜덤한 키값이 들어갔습니다.
- 데이터를 저장하면 `member` 와 `member:{randomKey}` 라는 두개의 키값이 저장되었습니다.
- `member` 키값은 Set 자료구조이며, `Member` 엔티티에 해당하는 모든 Key 를 갖고 있습니다.
- `member:{randomKey}` 값은 Hash 자료구조이며 테스트 코드에서 작성한 값대로 field, value 가 세팅한 걸 확인할 수 있습니다.
- `timeToLive` 를 설정했기 때문에 30초 뒤에 사라집니다. `ttl` 명령어로 확인할 수 있습니다.

<br>

## 3.2. RedisTemplate

`RedisTemplate` 을 사용하면 특정 Entity 뿐만 아니라 여러가지 원하는 타입을 넣을 수 있습니다.

`template` 을 선언한 후 원하는 타입에 맞는 `Operations` 을 꺼내서 사용합니다.

<br>

**config 설정 추가**

```java
@Configuration
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(host, port);
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate() {
        RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        return redisTemplate;
    }
}
```

- `RedisTemplate` 에 `LettuceConnectionFactory` 을 적용해주기 위해 설정해줍니다.

<br>

**Example**

```java
@SpringBootTest
public class RedisTemplateTest {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Test
    void testStrings() {
        // given
        ValueOperations<String, String> valueOperations = redisTemplate.opsForValue();
        String key = "stringKey";

        // when
        valueOperations.set(key, "hello");

        // then
        String value = valueOperations.get(key);
        assertThat(value).isEqualTo("hello");
    }


    @Test
    void testSet() {
        // given
        SetOperations<String, String> setOperations = redisTemplate.opsForSet();
        String key = "setKey";

        // when
        setOperations.add(key, "h", "e", "l", "l", "o");

        // then
        Set<String> members = setOperations.members(key);
        Long size = setOperations.size(key);

        assertThat(members).containsOnly("h", "e", "l", "o");
        assertThat(size).isEqualTo(4);
    }

    @Test
    void testHash() {
        // given
        HashOperations<String, Object, Object> hashOperations = redisTemplate.opsForHash();
        String key = "hashKey";

        // when
        hashOperations.put(key, "hello", "world");

        // then
        Object value = hashOperations.get(key, "hello");
        assertThat(value).isEqualTo("world");

        Map<Object, Object> entries = hashOperations.entries(key);
        assertThat(entries.keySet()).containsExactly("hello");
        assertThat(entries.values()).containsExactly("world");

        Long size = hashOperations.size(key);
        assertThat(size).isEqualTo(entries.size());
    }
}
```

- 위에서부터 차례대로 Strings, Set, Hash 자료구조에 대한 Operations 입니다.
- `redisTemplate` 을 주입받은 후에 원하는 Key, Value 타입에 맞게 Operations 을 선언해서 사용할 수 있습니다.
- 가장 흔하게 사용되는 `RedisTemplate<String, String>` 을 지원하는 `StringRedisTemplate` 타입도 따로 있습니다.

<br>

# Reference

- [Github 전체 코드](https://github.com/ParkJiwoon/practice-codes/tree/master/spring-redis)
- [Lettuce Reference Guide](https://lettuce.io/core/release/reference/)
- [Spring Data Redis Reference](https://docs.spring.io/spring-data/redis/docs/2.3.3.RELEASE/reference/html/#redis)
- [Spring Data Redis Reference - Redis Repositories](https://docs.spring.io/spring-data/redis/docs/2.3.3.RELEASE/reference/html/#redis.repositories)
