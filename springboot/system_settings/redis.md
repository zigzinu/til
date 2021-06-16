https://ozofweird.tistory.com/entry/Spring-Boot-Redis-Lettuce%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EA%B0%84%EB%8B%A8%ED%95%9C-API-%EC%A0%9C%EC%9E%91

참고하면 lettuce host port 세팅 잇음


---

기본에 충실한 구성으로 시작

https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-redis.html

https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-redis.html

---
Lettuce 는 Redis 에 접근하는 비동기 클라이언트다.

`LettuceConnectionFactory`를 설계하려고 보니, 생성자들을 참고하여
- ```
  public LettuceConnectionFactory(RedisStandaloneConfiguration configuration) {
    this(configuration, new MutableLettuceClientConfiguration());
  }
  ```
- ```
  public LettuceConnectionFactory(String host, int port) {
    this(new RedisStandaloneConfiguration(host, port), new MutableLettuceClientConfiguration());
  }
  ```
- ```
  public LettuceConnectionFactory(RedisSentinelConfiguration sentinelConfiguration) {
    this(sentinelConfiguration, new MutableLettuceClientConfiguration());
  }
- ```
  public LettuceConnectionFactory(RedisClusterConfiguration clusterConfiguration) {
    this(clusterConfiguration, new MutableLettuceClientConfiguration());
  }
  ```
등의 설계가 있는 것을 확인. 나는 Cluster 구성을 설계할 것이기 때문에 마지막 코드를 참고한다.

https://coding-start.tistory.com/129

여기서 적용 코드를 참고했다.

---

password 도 설정해주었다. `setPassword()`

---

Spring 에서 redis 에 세션을 저장했을 때 저장하는 key 들을 다음과 같다.

```
127.0.0.1:7002: spring:session:sessions:expires:b3358f77-5290-4a75-b805-a9d5c4a8b6d3
127.0.0.1:7001: spring:session:sessions:b3358f77-5290-4a75-b805-a9d5c4a8b6d3
127.0.0.1:7000: spring:session:expirations:1623811860000
```

