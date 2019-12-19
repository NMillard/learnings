# Domain Driven Design (DDD)

DDD's should only be applied to software that has a lot of complex business rules.
The concept of DDD revoles around business logic complexity, and nothing else.

## General Principals

Follow YAGNI to avoid trying to predict future needs of the business.
Code only what is needed
This shortens the development time

Follow KISS to avoid creating overly complex solutions
E.g. only begin to use inheritance and generics when there's actually a use-case for them
This results in more maintainable code

## Ubiquitous language

Use the language of the business.
Classes, tables, events, etc. should all derive their names from the terms in the ubiquitous language.

## Testing

Be pragmatic.
At first, cover only the most essential parts.

## Core Concepts

### Entities

Have it's own identity and can live independently of other objects.

### Value objects

Must be immutable. If value objects have the same member values, they can be treated interchangably.
Value objects cannot have a life on its own. Must live within an entity.

### Equality

There are generally three types of equality of objects:

1. Identifier
   If the unique identifier of two objects are the same, they are considered to be representing the same object
2. Reference
   If two object reference the same address in memory, they are same
3. Structural
   If all members of the two objects match, they are the same

Entities and value objects differ in the way they are tested for equality.
Entities has equal identifiers whereas value objects has structural equality.
