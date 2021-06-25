읽어볼반: https://pasudo123.tistory.com/379

1. build.gradle
```
    implementation 'org.springframework.session:spring-session-data-redis:2.5.1'
    implementation 'io.lettuce:lettuce-core:6.1.2.RELEASE'
```
2. application.properties

```
spring.session.store-type=redis
spring.session.redis.flush-mode=on_save
spring.session.redis.namespace=spring:session
spring.redis.cluster.nodes=172.26.1.4:7051,172.26.1.5:7051,172.26.1.6:7051
```

3. Lettuce Config

```
  private final RedisClusterConfigurationProperties clusterProperties;
  
	@Bean
	public LettuceConnectionFactory connectionFactory() {
        RedisClusterConfiguration clusterConfiguration = new RedisClusterConfiguration(clusterProperties.getNodes());
		return new LettuceConnectionFactory(clusterConfiguration); 
	}
```

3-1. Cluster Property 에서 nodes ip들 읽어오기. 
```
@Component
@ConfigurationProperties(prefix = "spring.redis.cluster")
public class RedisClusterConfigurationProperties {
    
    /**
     * spring.redis.cluster.nodes[0]=127.0.0.1:6379
     * spring.redis.cluster.nodes[1]=127.0.0.1:6380
     * spring.redis.cluster.nodes[2]=127.0.0.1:6381
     */
    List<String> nodes;
 
    public List<String> getNodes() {
        return nodes;
    }
 
    public void setNodes(List<String> nodes) {
        this.nodes = nodes;
    }
       
}
```

4. Serializable 문제 해결

- UserDao 를 Serialize 하려고 한다.
```
public class UserDao implements Serializable {
```
- DefaultSerializer 를 사용하면 실패한다.
```
    @Bean
    public RedisTemplate<Object, Object> sessionRedisTemplate() {
        RedisTemplate<Object, Object> template = new RedisTemplate<Object, Object>();
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());

        template.setDefaultSerializer(new GenericJackson2JsonRedisSerializer());

        template.setConnectionFactory(connectionFactory());
        return template;
    }
```

출처: https://stackoverflow.com/questions/38532754/spring-session-redis-serializer-serializationexception

5. SetTimeout vs maxInactiveIntervalInSeconds

- Redis 쓰기 전에는 `application.properties`에서 `server.servlet.session.timeout=1m` 으로 세팅하면 `session.getMaxInactiveInterval()` 값도 `60`으로 맞춰졌는데, `@EnableRedisHttpSession`의 `maxInactiveIntervalInSeconds`의 기본값은 `1800`이다.
- 아래처럼 세팅을 추가해주면, inactive 시간이 지나서 세션이 만료되는 것을 확인할 수 있다.
```
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 60)
```
- timeout 이랑 inactiveInterval 의 관계는 더 알아볼 필요가 있는데, inactiveInterval 로 timeout 세팅을 오버라이드하는 것 같다.

출처: 
- https://docs.spring.io/spring-session/docs/current/api/org/springframework/session/data/redis/config/annotation/web/http/EnableRedisHttpSession.html#saveMode--
- https://pasudo123.tistory.com/379






## 쿠키

- `JSESSIONID`에서 `SESSION`로 바뀜
- Instead of using Tomcat’s HttpSession, we persist the values in Redis. Spring Session creates a cookie named SESSION in your browser. That cookie contains the ID of your session.

출처: https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-redis.html

---
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

- 어떤 의미를 하는지? https://deveric.tistory.com/76

타입 조회

```
127.0.0.1:7000> auth fabric
OK
127.0.0.1:7000> type "spring:session:sessions:3fe57b8d-0ee9-471b-817d-b8cbc7f6d978"
hash
127.0.0.1:7000> type "spring:session:expirations:1623819300000"
set
127.0.0.1:7000> type "spring:session:sessions:expires:3fe57b8d-0ee9-471b-817d-b8cbc7f6d978"
-> Redirected to slot [8477] located at 127.0.0.1:7001
(error) NOAUTH Authentication required.
127.0.0.1:7001> auth fabric
OK
127.0.0.1:7001> type "spring:session:sessions:expires:3fe57b8d-0ee9-471b-817d-b8cbc7f6d978"
string
```

- `spring:session:sessions:세션값`: hash
- `spring:session:expirations:시간Millis`: set
- `spring:session:sessions:expires:세션값`: string

Hash 타입에 대한 조회 (`spring:session:sessions:세션값`)

- 세선 생성 시간 / 마지막 조회 시간 / 최대 타임아웃 허용 시간 / 세션에 저장한 값들
- 을 field 로 이용하여 value 를 조회할 수 있다.

```
127.0.0.1:7000> hkeys "spring:session:sessions:3fe57b8d-0ee9-471b-817d-b8cbc7f6d978"
1) "creationTime"
2) "sessionAttr:SPRING_SECURITY_SAVED_REQUEST"
3) "lastAccessedTime"
4) "maxInactiveInterval"
127.0.0.1:7000> hget "spring:session:sessions:3fe57b8d-0ee9-471b-817d-b8cbc7f6d978" "creationTime"
"\xac\xed\x00\x05sr\x00\x0ejava.lang.Long;\x8b\xe4\x90\xcc\x8f#\xdf\x02\x00\x01J\x00\x05valuexr\x00\x10java.lang.Number\x86\xac\x95\x1d\x0b\x94\xe0\x8b\x02\x00\x00xp\x00\x00\x01z\x13\x10\x02\x19"
```

- `hkeys <key>`
- `hget <key> <field>`

Set 타입에 대한 조회 (`spring:session:expirations:시간Millis`)

- 해당 시간에 만료되는 session 들의 집합이다. 시간이 되면 모두 조회해서 삭제한다.

```
127.0.0.1:7000> smembers "spring:session:expirations:1623819300000"
1) "\xac\xed\x00\x05t\x00,expires:3fe57b8d-0ee9-471b-817d-b8cbc7f6d978"
```

- expire 에 대한 값들이 들어있는 것을 확인

String 타입에 대한 조회 (`spring:session:sessions:expires:세션값`)

```
127.0.0.1:7001> get "spring:session:sessions:expires:3fe57b8d-0ee9-471b-817d-b8cbc7f6d978"
""
```

- `string` 타입은 key와 value의 관계가 1:1 이다.

**정리하면**
- `spring:session:sessions:b3358f77-5290-4a75-b805-a9d5c4a8b6d3` 은 Hash 타입으로 세션에 관한 
여러가지 `field`에 대한 `value`를 조회할 수 있다.
- `spring:session:sessions:expires:b3358f77-5290-4a75-b805-a9d5c4a8b6d3` 은 String 타입으로 
단순히 만료되어야하는 세션 값을 저장한다.
- `spring:session:expirations:1623811860000` 은 Set 타입으로 해당 시간에 만료되어야 하는 
세션(string)들의 집합이다.

127.0.0.1:7002: spring:session:sessions:expires:b3358f77-5290-4a75-b805-a9d5c4a8b6d3
`spring:session:sessions:b3358f77-5290-4a75-b805-a9d5c4a8b6d3`
127.0.0.1:7000: spring:session:expirations:1623811860000


