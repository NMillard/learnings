# Testing with Junit5

## Parameterized tests

### Naming parameterized tests

We know descriptive names are important for understanding what a test does. When it comes to parameterized tests, 
we typically test a bunch of different scenarios, where each scenario can benefit from having its own description.

```java
class NamedTest {
    
    @MethodSource("scenarios")
    @ParameterizedTest
    void hellos(String value) {
        // Your test
    }

    private static Stream<Arguments> scenarios() {
        return Stream.of(
                // The first argument of "named()" becomes the tests scenario name.
                Arguments.of(named("hello1", "hello")),
                Arguments.of(named("hello2", "hello again"))
        );
    }
}
```