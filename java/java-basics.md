# Java basics


- [Java basics](#java-basics)
  - [Records](#records)
    - [Basic syntax](#basic-syntax)
    - [Record canonical constructor](#record-canonical-constructor)
    - [Record compact constructor](#record-compact-constructor)
  - [Annotations](#annotations)
  - [Generics](#generics)
    - [Wildcards](#wildcards)
  - [Thread dumps](#thread-dumps)

## Records

Records are immutable and implicitly final. Records cannot extend classes but they can implement interfaces.

### Basic syntax
```java
public record Product(UUID id, String name) {
    // The compiler translates this to a final class with
    // private final fields.

    // The constructor is generated and the fields are initialized.
    // Public accessor methods for the fields are generated as well.
    // But instead of the expected "getId()" we'd get 
    // just an "id()" method.

    // It's also possible to override accessor methods. Remember to
    // use the @Override annotaton.

    @Override
    public String name() {
        // implementation
    }
}
```
You can add methods to a `record` but not instance fields.

If you do override an accessor method, then it's a good practice to always return the actual value for the field which has its accessor method overridden.

### Record canonical constructor

You can add input validation in the record's canonical constructor and make your own defensive copies in case the record takes a reference type or list as argument.

```java
public record Product(UUID id, String name, List<String> tags) {

    public Product(UUID id, String name) {
        // Perform validation 

        this.id = id;
        this.name = name;
        this.tags = List.copyOf(tags); // <- make immutable
    }
}
```

### Record compact constructor
There's an other syntax for creating records, that is more concise. Notice the fields are set without having to set them yourself. The tags argument--and not instance field--is reassigned to an immutable list.

```java
public record Product(UUID id, String name, List<String> tags) {

    public Product {
        // Perform validation 
        tags = List.copyOf(tags); // <- make immutable
    }
}
```
The compact constructor is preferred over the canonical constructor.


## Annotations

Simply put: annotations adds metadata to source code.

The three main use cases for annotations are:
- Additional information to the compiler.
- Writing annotation processors to generate code.
- Reading annotations at runtime.

## Generics

### Wildcards
The wildcard `<?>` is used to refer to a family of types.
- `?`: Unbounded - all types
- `? extends AType`: upper bounded. Refers to subtypes of `AType`. 
- `? super AType`: lower bounded. Refers to supertypes of `AType`. 

## Thread dumps

A thread dump is essentially just text that represents the runtime state of a program at a specific point in time. Thread dumps are also referred to as Snapshots.

This means, the thread dump contains all the call stacks of all threads being tracked by the JVM.
