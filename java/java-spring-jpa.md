# Spring JPA

## Add functionality
Add the OR/M functionality and database driver by referencing the data-jpa and database driver dependencies.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

Set up the application settings to connect to a database:
```yaml
spring:
  datasource:
    username: ${DATASOURCE_USERNAME}
    password: ${DATASOURCE_PASSWORD}
    url: ${DATASOURCE_URL}
    hikari:
      maximum-pool-size: 4
      max-lifetime: 120000
      keepalive-time: 60000
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
```

Then you're ready to create repositories. No need to reiterate how it's done as this guide provides enough information: https://docs.spring.io/spring-data/jpa/docs/current/reference/html/

## Migrations
Add migrations functionality using flyway. An easy to use migrations library.


```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

Now set up the application settings. It's a good idea to let flyway create the scheme in case it doesn't exist already.

```yaml
spring:
  flyway:
    enabled: true
    schemas:
      - scheduler
    locations:
      - classpath:db/migration
    create-schemas: true
  sql:
    init:
      mode: NEVER
```

[Read the documentation](https://flywaydb.org/documentation/)

## Testing

Best practice is to _not_ rely on mocks for database interactions tests. Instead, spin up database containers and connect to them in order to use real SQL queries in tests.

Add the following dependencies to the pom.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

Then set up the application settings in a test profile.
```yaml
# ./src/test/resources/application-test.yml
spring:
  datasource:
    username: postgres
    password: postgres
    url: jdbc:tc:postgresql:10-alpine:///postgres
```

Add some annotations to your tests that enables autowiring repositories and the datasource.
```java
// Your test class
@ActiveProfiles("test") // <- load the test profile
@DataJpaTest // <- add JPA to the application context allowing autowired repositories in the test
@TestInstance(TestInstance.Lifecycle.PER_METHOD)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class ScheduleRunnerTest {
    @Autowired
    YourRepository repository;

    @Autowired
    DataSource dataSource;
}
```

The above test class ensures that a container is started and the flyway migrations are applied to the database.

You can even seed data before running each test.
```java
@Test
@Sql("/initialize-with-data.sql")
void runsDueScheduleSuccessfully() {
    // do test things
}
```
The `initialize-with-data.sql` is placed in `./src/test/resources/` and contains whatever SQL you'd like.