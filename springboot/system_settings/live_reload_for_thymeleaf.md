# How to apply live reload for thymeleaf?

**Question is** 🤔 when developing frontend page with springboot + thymeleaf, you will like the browser to apply changes on `.html` file without restarting the tomcat server.
It is as simple as adding few lines to your `application.properties` or `application.yml`.

### `application.properties` example

```
# live reload on thymeleaf
spring.thymeleaf.cache=false
spring.thymeleaf.mode=HTML
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.prefix=file:src/main/resources/templates/
spring.resources.static-locations=file:src/main/resources/static/
spring.resources.cache.period=0
```

### `application.properties` example

```
# live reload on thymeleaf
spring:
  thymeleaf: # Thymeleaf
    cache: false
    mode: HTML
    encoding: UTF-8
    prefix: file:src/main/resources/templates/
  resources: # Static resources
    static-locations: file:src/main/resources/static/
    cache:
      period: 0
```

reference
- https://stackoverflow.com/questions/58275418/live-reload-for-thymeleaf-template
