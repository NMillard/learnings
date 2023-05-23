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

## Advanced Testcontainers
Sometimes, you'd want your database to have exisiting schemas, tables, views, and whatnot before running the migrations. Migrations may require some database objects to exist before they can be executed (it's not ideal, it's simply realitiy).

You can run scripts against e.g. a postgres database testcontainer before the test spring context (from `@DataJpaTest` or `@SpringTest`) is started. Having flyway enabled will run the migrations as part of the Spring Context startup process.

So, what we do is to configure our testcontainer before hand.

```java
public class InitializedDatabase {
    public InitializedDatabase() {
        container = InitializedPostgresContainer.getInstance();
    }

    @Container
    public static InitializedDatabase.InitializedPostgresContainer container;

    /**
     * Get the actual database settings from the running container and add them to the
     * environment, (akin to how settings are added to the application-test.yml).
     * We need to do this since we don't know the postgres mapped port before hand.
     */
    @DynamicPropertySource
    static void postgresProperties(DynamicPropertyRegistry registry) {
        container.start();

        registry.add("spring.datasource.url", () -> container.getJdbcUrl());
        registry.add("spring.datasource.username", () -> container.getUsername());
        registry.add("spring.datasource.password", () -> container.getPassword());
    }

    /**
     * This class ensures that the postgres database is initialized by running a script located
     * in the test resource folder. This happens before the SpringBoot application is started.
     */
    public static class InitializedPostgresContainer extends PostgreSQLContainer<InitializedDatabase.InitializedPostgresContainer> {

        private InitializedPostgresContainer() {
            super(DockerImageName.parse("postgres:10-alpine"));
        }

        public static InitializedPostgresContainer getInstance() {
            return new InitializedDatabase.InitializedPostgresContainer();
        }

        @Override
        public void start() {
            super.start();
            runInitializationScript();
        }

        private static void runInitializationScript() {
            try (Connection connection = DriverManager.getConnection(
                    container.getJdbcUrl(),
                    container.getUsername(),
                    container.getPassword());
                 Statement statement = connection.createStatement()) {

                // The "some-sql-script.sql is located in the test resource folder.
                InputStream sqlStream = InitializedDatabase.class.getClassLoader().getResourceAsStream("some-sql-script.sql");
                var sql = new String(Objects.requireNonNull(sqlStream).readAllBytes());

                statement.execute(sql);
            } catch (SQLException | IOException e) {
                throw new RuntimeException(e);
            }
        }
    }
}

// Your actual test class
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles(profiles = "test")
@EnableWebMvc
class SomeControllerTest extends InitializedDatabase { // Extend the new class

    @Autowired
    private MockMvc mockMvc;

    @Test
    void callAnEndpoint() throws Exception {
        // Act
    }
}

```

Remember to add the junit testcontainers dependency.
```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.18.1</version>
    <scope>test</scope>
</dependency>
```