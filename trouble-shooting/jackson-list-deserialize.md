# Spring Boot 에서 Redis Cache 사용 시 List 역직렬화 에러 (GenericJackson2JsonRedisSerializer)

# 상황

Redis Cache 를 사용해서 `List<?>` 를 저장하려고 했습니다.

직렬화해서 데이터 저장까지는 잘 되었는데 다시 역직렬화 하려고 하니 에러가 발생하며 실패했습니다.

<br>

```java
@Cacheable(cacheNames = "members", key = "'all'")
public List<Member> findAll() {
    List<Member> members = store.values().stream().toList();
    return members;
}
```

캐싱한 데이터는 위와 같습니다.

`List<Member>` 를 응답으로 내려주고 `members` 라는 캐시의 `all` 이라는 키값으로 저장됩니다.

Redis 설정으로 Value 는 `GenericJackson2JsonRedisSerializer` 를 사용하여 직렬화했습니다.

<br>

![](https://raw.githubusercontent.com/ParkJiwoon/PrivateStudy/master/trouble-shooting/images/screen_2023_04_08_05_41_42.png)

Redis 를 확인해보면 제대로 저장된 것을 확인할 수 있습니다.

<br>

# 에러 로그

```java
com.fasterxml.jackson.databind.exc.MismatchedInputException: Unexpected token (START_OBJECT), expected VALUE_STRING: need JSON String, Number of Boolean that contains type id (for subtype of java.lang.Object)
 at [Source: (byte[])"[{"@class":"com.example.springbootcache.model.Member","id":1,"name":"ChulSoo","age":50}]"; line: 1, column: 2]
```

<br>

# 원인

List<?> 를 그대로 저장해서 그렇습니다.

사실 정확한 원인은 저도 모릅니다.

그래도 확신은 없지만 나름대로 추측을 해보겠습니다.

`GenericJackson2JsonRedisSerializer` 는 직렬화 할 때 `@class` 라는 Key 값에 클래스의 패키지 정보까지 전부 저장됩니다.

그런데 List 를 통째로 저장하면 위 사진과 같이 `{ "@class": "..." }` 이 아니라 `[{ "@class": "..."}]` 로 저장되어 찾지 못해서 발생하는 이슈 같습니다.

<br>

# 해결

List 를 감싸는 Wrapper 클래스를 만들어 주면 해결됩니다.

<br>

## Members 클래스 정의

```java
@Getter
@NoArgsConstructor
public class Members {
    private List<Member> members = new ArrayList<>();

    public Members(List<Member> members) {
        this.members = members;
    }
}
```

<br>

## 캐싱 대상 변경

```java
@Cacheable(cacheNames = "members", key = "'all'")
public Members findAll() {
    List<Member> members = store.values().stream().toList();
    return new Members(members);
}
```

<br>

## Redis 저장 확인

![](https://raw.githubusercontent.com/ParkJiwoon/PrivateStudy/master/trouble-shooting/images/screen_2023_04_08_05_41_57.png)