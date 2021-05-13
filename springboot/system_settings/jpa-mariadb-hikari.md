### JPA + MariaDB + HikariCP 를 활용한 DB 연결 세팅

1. `application.yml`

```
spring:
  datasource:
    url: jdbc:mariadb://172.17.0.1:3306/cryptodb
    driver-class-name: org.mariadb.jdbc.Driver
    username: id
    password: pw
    hikari:
      connection-timeout: 60000 #1min
      maximum-pool-size: 20
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create #none/update/validate/create/create-drop
```
- **MariaDB**: 도커 컨테이너로 구동중
- **hikari**: connection pool
- **hibernate**: mapping framework
- **ddl-auto**: 스프링부트 실행/종료시 DB 테이블 업데이트 설정

2. `build.gradle`

```
dependencies {
	implementation (
		// Rest
		'org.springframework.boot:spring-boot-starter-web',
		// JPA
		'org.springframework.boot:spring-boot-starter-data-jpa',
		// MariaDB
		'org.mariadb.jdbc:mariadb-java-client:2.7.2',
		// Lombok
		'org.projectlombok:lombok'
	)
}
```

3. 테이블 클래스

```
@ToString
@Data
@Getter
@NoArgsConstructor
@Entity
public class Expected {
  @Id
  private String market;
  @Column(nullable = false)
  private Double price;

  @Builder
  public Expected(String market, Double price){
    this.market = market;
    this.price = price;
  }
}
```
