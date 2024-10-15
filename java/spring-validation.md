# Spring validation

- [Spring validation](#spring-validation)
  - [Dependency](#dependency)
  - [Considerations](#considerations)
  - [Example](#example)
  - [Returning ProblemDetails](#returning-problemdetails)
  - [Handling validation](#handling-validation)
  - [Hibernate validator](#hibernate-validator)

[View associated code here.](https://github.com/NMillard/spring-training)

## Dependency
Enable validation by importing this package:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Then you can add `@Valid` to endpoint method arguments.

```java
public ResponseEntity<?> someEndpoint(@Valid SomeModel model) {
    // 
}
```

Make sure to use the `jakarta` and not `javax` annotations and classes.

## Considerations

When validating data at the application API level, that is, controller level. You should try avoid validating business rules, since these belong to the business logic layer.

A quick note on this: Postel's Robustness principle says that we should be "liberal in what you accept and conservative in what you do." However, this doesn't mean we should accept every and any thing.

But, it does mean that in controllers - the interface to our clients - we want to accept their requests as long as they conform to a set of expectations.

Write validation for the following:
- Data types.
- Codes - allowed values, such as enum values.
- Range - for instance a number is between x and y.
- Format - for instance a date is in ISO date format.
- Consistency - relations between input data.
- Uniqueness - the same data isn't provided more times, say in a list of items.

As you can see, these validations are tied to the input itself and not business rules. Incorporating business rule validation into endpoint request validation means we might have to duplicate the validation in multiple places, and quickly get out of sync.

## Example

Say we have the following endpoint and we want to validate the data provided to us.

```java
public record CreateUserRequest(String name, String lastName, int age) {}

// Endpoint in a controller
@PostMapping("") // POST api/users
public ResponseEntity<Void> createUser(@RequestBody CreateUserRequest request) {
    return ResponseEntity.ok().build();
}
```

The `@RequestBody` performs data type validation.

> [!NOTE]
> No other requirements is added to the `CreateUserRequest` yet.

## Returning ProblemDetails

Issues with a user-made request should be addressed with a "Problem Details" response as in accordance with RFC 7807.

- Status: HTTP Status.
- Type: A URL that identifies the problem and how to mitigate it.
- Title: Short summary of the problem.
- Detail: Human readable problem explanation.
- Instance: URI of the service where this problem occurred.

Use the `ErrorResponse` or `ProblemDetail` as the return type for handling exceptions.

It may be a good idea to implement some global error handling to not expose underlying information that an exception carries.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ErrorResponse handleAllExceptions(Exception ex) {
        return ErrorResponse
                .builder(ex, ProblemDetail.forStatusAndDetail(
                        HttpStatus.INTERNAL_SERVER_ERROR,
                        ex.getMessage())
                )
                .build();
    }
}
```

This exeception handler method is invoked whenever the spring web application throws an exception during execution. 

## Handling validation
Say we update our request to the following:

```java
public record CreateUserRequest(
        @NotNull @NotEmpty String name,
        String lastName,

        @Range(min = 1, max = 120)
        int age) {
}
```

To validate the `jakarta.validation.constraints.*` annotstions, we'll need to update the endpoint as well, with an `@Valid` parameter annotation.

```java
@PostMapping("") // POST api/users
public ResponseEntity<Void> createUser(@Valid @RequestBody CreateUserRequest request) {
    return ResponseEntity.ok().build();
}
```

Also, if more than one validation fails, we'd like to summarize that in our Problem Details response. So, let's add another exception handler.

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ProblemDetail handleMethodArgumentNotValidException(MethodArgumentNotValidException ex) {
    var violations = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, "Validation error");
    List<String> v = ex.getFieldErrors().stream()
            .map(violation -> "%s: %s". formatted(violation.getField(), violation.getDefaultMessage()))
            .toList();
    violations.setProperty("violations", v);
    return violations;
}
```

## Hibernate validator

Hibernate validator is a popular implementation of JSR380 Bean Validation 2.0 which provides a standardized way of validating constraints.

Hibernate validator is already included in the `spring-boot-starter-validation` package.