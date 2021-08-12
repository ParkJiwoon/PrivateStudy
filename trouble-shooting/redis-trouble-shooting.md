# RedisRepository 가 Bean 으로 등록되지 않는 이슈

해결하고 나니 굉장히 사소한 실수였습니다.

```text
Error creating bean with name 'tokenRedisRepository' defined in TokenRedisRepository defined in @EnableRedisRepositories declared on RedisRepositoriesRegistrar.EnableRedisRepositoriesConfiguration: 
Invocation of init method failed; 
nested exception is org.springframework.data.mapping.MappingException: Entity com.example.login.entity.
Token requires to have an explicit id field. Did you forget to provide one using @Id?
```

- RedisRepository 를 적용하는 도중 위와 같은 에러가 발생
- Redis 에 사용할 객체의 `@Id` 어노테이션 임포트를 잘못해서 발생 (JPA 에서 사용하는 `@Id` 를 임포트함)
- `javax.persistence.Id` 대신 `org.springframework.data.annotation.Id` 을 임포트 해서 해결