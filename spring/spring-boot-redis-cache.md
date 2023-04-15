# Spring Boot 에서 Redis Cache 사용하기

# Overview

베이스 코드로 [Spring Boot Cache 적용](https://bcp0109.tistory.com/385)에 있던 코드들을 재활용할 예정이라 앞의 글을 먼저 읽어보는걸 추천합니다.

단순하게 Redis Cache 설정만 알고 싶다면 상관 없습니다.

코드를 직접 실행해보려면 [로컬에 Redis 를 설치](https://bcp0109.tistory.com/327)해야 합니다.

만약 별도의 Redis 서버를 운영 중이라면 해당 서버를 사용해도 됩니다.

<br>

# 1. Dependency 추가

```java
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
implementation 'org.springframework.boot:spring-boot-starter-cache'
```

`spring-boot-starter-cache` 에 이어 `spring-boot-starter-data-redis` 설정을 추가합니다.

Spring Data Redis 설정을 추가하면 자동으로 기본 캐시가 `ConcurrentMapCache` 에서 `RedisCache` 로 설정됩니다.

<br>

# 2. Redis Configuration

```yml
spring:
  data:
    redis:
      host: 127.0.0.1
      port: 6379
```

<br>

```java
@Configuration
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(
                new RedisStandaloneConfiguration(host, port)
        );
    }
}
```

Redis 연결을 위한 기본 설정을 추가합니다.

<br>

# 3. Redis Cache Configuration

```java
@EnableCaching
@Configuration
public class CacheConfig {

    /**
     * Spring Boot 가 기본적으로 RedisCacheManager 를 자동 설정해줘서 RedisCacheConfiguration 없어도 사용 가능
     * Bean 을 새로 선언하면 직접 설정한 RedisCacheConfiguration 이 적용됨
     */
    @Bean
    public RedisCacheConfiguration redisCacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(60))
                .disableCachingNullValues()
                .serializeKeysWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer())
                )
                .serializeValuesWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer())
                );
    }

    /**
     * 여러 Redis Cache 에 관한 설정을 하고 싶다면 RedisCacheManagerBuilderCustomizer 를 사용할 수 있음
     */
    @Bean
    public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
        return (builder) -> builder
                .withCacheConfiguration("cache1",
                        RedisCacheConfiguration.defaultCacheConfig()
                                .computePrefixWith(cacheName -> "prefix::" + cacheName + "::")
                                .entryTtl(Duration.ofSeconds(120))
                                .disableCachingNullValues()
                                .serializeKeysWith(
                                        RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer())
                                )
                                .serializeValuesWith(
                                        RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer())
                                ))
                .withCacheConfiguration("cache2",
                        RedisCacheConfiguration.defaultCacheConfig()
                                .entryTtl(Duration.ofHours(2))
                                .disableCachingNullValues());
    }
}
```

Redis Cache 설정을 추가합니다.

`CacheManager` 를 사용했던 `ConcurrentMapCache` 와는 다르게 Redis 는 간단하게 Redis Cache 설정을 적용할 수 있습니다.

우선 Spring Data Redis 를 사용한다면 Spring Boot 가 `RedisCacheManager` 를 자동으로 설정해줍니다.

하지만 Redis 는 직렬화/역직렬화 때문에 별도의 캐시 설정이 필요하고 이 때 사용하는게 `RedisCacheConfiguration` 입니다.

`RedisCacheConfiguration` 설정은 Redis 기본 설정을 오버라이드 한다고 생각하면 됩니다.

- `computePrefixWith`: Cache Key prefix 설정
- `entryTtl`: 캐시 만료 시간
- `disableCachingNullValues`: 캐싱할 때 null 값을 허용하지 않음 (`#result == null` 과 함께 사용해야 함)
- `serializeKeysWith`: Key 를 직렬화할 때 사용하는 규칙. 보통은 String 형태로 저장
- `serializeValuesWith`: Value 를 직렬화할 때 사용하는 규칙. Jackson2 를 많이 사용함

<br>

만약 캐시이름 별로 여러 세팅을 하고 싶다면 `RedisCacheManagerBuilderCustomizer` 를 선언해서 사용할 수 있습니다.

위 코드에서는 `cache1`, `cache2` 두 가지 캐시를 설정했으며 만약 다른 이름의 캐시를 사용하려 한다면 기본 설정인 `RedisCacheConfiguration` 를 따라갑니다.

`GenericJackson2JsonRedisSerializer` 를 사용할 때 주의할 점은 여러개의 데이터를 한번에 저장할 때 `List` 를 사용하지 말고 별도의 일급 컬렉션을 선언해서 사용해야 합니다.

자세한 이슈는 [Spring Boot 에서 Redis Cache 사용 시 List 역직렬화 에러 (GenericJackson2JsonRedisSerializer)](https://bcp0109.tistory.com/384) 글에 정리해두었습니다.

<br>

# 4. Redis 데이터 확인

API 호출 후 Redis CLI 에서 직접 저장된 데이터를 확인해봅니다.

<br>

## 4.1. cache1 데이터 확인

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_04_15_23_37_06.png?raw=true)

`cache1` 은 설정한 대로 prefix 가 붙은 key 값이 사용됩니다.

<br>

## 4.2. members 데이터 확인

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_04_15_23_37_59.png?raw=true)

별도의 이름을 설정하지 않은 `members` 캐시는 그냥 일반 키값으로 저장됩니다.

<br>

# Conclusion

Redis Cache 는 일반적으로 가장 많이 사용되는 글로벌 캐시입니다.

직접 `RedisTemplate` 을 호출해서 구현할 수도 있지만 Spring Boot 에서 제공하는 설정을 알아두면 나중에 유용하게 사용할 일이 있을 겁니다.

<br>

# Reference

- [Github Sample 코드](https://github.com/ParkJiwoon/spring-boot-redis-cache-sample/)
- [Baeldung - Spring Boot Cache with Redis](https://www.baeldung.com/spring-boot-redis-cache)