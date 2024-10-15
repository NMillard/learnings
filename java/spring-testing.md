# Testing Java Applications

## Integration test

Consider using the `@ContextConfiguration(classes = {})` instead of `@SpringBootTest`, since you'll only rely on the specified classes rather than configuring the full application context.


```java
@ExtendWith({SpringExtension.class, MockitoExtension.class})
@ContextConfiguration(classes = { UserController.class })
class UserControllerTest {

    @Autowired
    UserController sut;

    @MockBean
    UserRepository repository;

    @Test
    void demo() {
        // Arrange
        doReturn(List.of()).when(repository).getAll();

        // Act
        ResponseEntity<List<User>> result = sut.getUsers();

        // Assert
        assertThat(result.getBody()).isEmpty();
    }
}
```

Considering test execution speed, you may want to eliminate relying on the spring context completely. If that's the case, then consider this approach instead:

```java
@ExtendWith(MockitoExtension.class)
class UserControllerTest {

    @InjectMocks
    UserController sut;

    @Mock
    UserRepository repository;

    @Test
    void demo() {
        // Arrange
        doReturn(List.of()).when(repository).getAll();

        // Act
        ResponseEntity<List<User>> result = sut.getUsers();

        // Assert
        assertThat(result.getBody()).isEmpty();
    }
}
```

### Using a database

Integration tests are also useful to verify your database interactions work as expected. If using JPA, then don't test the default methods unless you need to verify data integrity, joins, etc.

