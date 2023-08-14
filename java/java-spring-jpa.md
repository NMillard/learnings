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

## Entity Manager and Repositories
Having a dependency on JPA allows you to fetch data using the `EntityManager` class and the `JpaRepository<>` interfaces.

Say we have an entity like this, with all the required annotations.

```java
@Entity
@Table(name = "employees", schema = "demo")
public class Employee {

    protected Employee() {} // to make JPA happy

    @Id
    private UUID id;

    @Column(name = "fullname", nullable = false)
    private String name;

    @ColumnTransformer(write = "?::json")
    @Column(nullable = false, columnDefinition = "json")
    private String preferences; // Say we store the preferences as JSON 

    @Column
    @Convert(converter = LongListConverter.class) // we'll get to this soon
    private List<Long> phoneNumbers; // Image these are stored as comma-delimited text in one column

    // Getters, setters, etc...
}
```
Now, this employee type is almost fully ready to be queried and persisted with JPA. We just need to implement a few more classes.  
We have a list of phonenumbers stored as text in the database, but in Java that column is converted to a list of `Long`. So, we need a converter for this.

```java
@Converter
public class LongListConverter implements AttributeConverter<List<Long>, String> {
    @Override
    public String convertToDatabaseColumn(List<Long> attribute) {
        boolean noValue = attribute == null || attribute.isEmpty();
        if (noValue) return null;

        return attribute.stream()
                .map(Object::toString)
                .collect(Collectors.joining(","));
    }

    @Override
    public List<Long> convertToEntityAttribute(String dbData) {
        boolean noValue = dbData == null || dbData.isEmpty();
        if (noValue) return new ArrayList<>();

        return Arrays.stream(dbData.split(","))
                .map(Long::valueOf)
                .toList();
    }
}
```

This allows us to both query and persist that list without problems.

To perform database operations on this `Employee` entity, we'll create an interface. I've added a few custom queries to the interfaces, just to demonstrate some techniques beyond the very basics.

```java
@Repository
interface JpaEmployeeRepository extends JpaRepository<Employee, UUID> { 

    // Simple "where" query.
    @Query("select e from Employee e where e.name = :name")
    List<Employee> simpleWhere(@Param("name") String name);

    @Query("""
            select e from Employee e
            where e.name = :#{search.name}
            and e.phoneNumbers in :#{search.phoneNumbers}
            """)
    List<Employee> usingObjectWhere(@Param("search") SearchCriteria search);
}

class SearchCriteria {
    private String name;
    private List<Long> phoneNumbers;

    // getters, setters...
}
```

There's created a concrete class behind the scenes at runtime.


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
However, when you experiment, you'd rather want to use a container that doesn't shut down once the test exits. We'll look at this scenario soon.

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

## Experimenting using a real database
Using testcontainers allows you to run the tests with a real database in a build environment. But, you'll most often be experimenting a lot, so, you need to connect to a local database, and have state persisted even when the tests exit.

The approach is much like with testcontainers.

First, create a dedicated application yaml file that is used only for experimentations. This shouldn't be added to git.

```yaml
# ./src/test/resources/application-local.yml
spring:
  datasource:
    username: postgres
    password: postgres
    url: jdbc:postgresql://localhost:5432/postgres
```

Then add the test class itself.

```java
@ActiveProfiles("local")
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@EnableAutoConfiguration
@ContextConfiguration(classes = { // The classes that are added to the dependency container
        YouRepository.class,
})
@EntityScan(basePackageClasses = {/*specific classes in the main project if needed*/})
@Transactional(propagation = Propagation.NOT_SUPPORTED) // Allows you to persist state after text is done 
class EjerskifteHandlerTest {

    @Autowire
    YouRepository repository;
}
```