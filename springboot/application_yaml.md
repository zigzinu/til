### Springboot v2.4.x 이후 profile 설정

- **group** 기능이 추가되었다. 여러가지 profile 을 조합한 profile 로 선택 가능하다.

```
spring:
  profiles:
    active: foo
    group:
      foo: "server-A, db-A"
      bar: "server-A, db-B"

---
spring:
  config:
    activate:
      on-profile: server-A
---
spring:
  config:
    activate:
      on-profile: db-A
---
spring:
  config:
    activate:
      on-profile: db-B
```
