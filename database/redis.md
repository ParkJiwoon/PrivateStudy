# Redis (with Spring Boot)

# Redis 란?

# Redis 사용

## Redis 설치

## Redis 명령어

# Java 의 Redis Client

Java 의 Redis Client 는 크게 두 가지가 있습니다.

Jedis 와 Lettuce 인데요.

원래 Jedis 를 많이 사용했으나 여러 가지 단점 (멀티 쓰레드 불안정, Pool 한계 등등..) 과 Lettuce 의 장점 (Netty 기반이라 비동기 지원 가능) 때문에 Lettuce 로 추세가 넘어가고 있었습니다.

그러다 결국 Spring Boot 2.0 부터 Jedis 가 기본 클라이언트에서 deprecated 되고 Lettuce 가 탑재되었습니다.

- [Spring Session 에서 Jedis 대신 Lettuce 를 사용하는 이유](https://github.com/spring-projects/spring-session/issues/789) 참고

<br>

# Spring Boot 에서 Redis 설정

## RedisTemplate

## RedisRepository

# Reference

- [Lettuce Reference Guide](https://lettuce.io/core/release/reference/)