# Domain Driven Design (DDD)

DDD's should be applied to software that has a lot of complex business rules.
The concept of DDD revoles around business logic complexity, and nothing else.  

Simple, data-centric apps may not benefit much from DDD, as they're simply used to
shovel data into a database.  

DDD is essentially a philosophy and mindset you apply, where programming tasks are only
carried out once a throrough understanding of the domain is achieved. Practicing DDD is centered
around the "what" and "why". Not the "how".

## Velocity and Lines-of-Code
Focusing on what and why is fine, but you'll at some point need to write code.  
Writing DDD-ish code is slower. You need to consider invariants, and if the models 
accurately represent the real domain. It's an iterative process where verifiablity is
in high demand.

## Ubiquitous language

Use the language of the business.
Classes, tables, events, etc. should all derive their names from the terms in the ubiquitous language.

If you ever find a concept existing in code, but not in the business language, you either rename it or
evaluate if it needs to exist. Rogue concepts become sources of maintainability nightmares.

### Domain Events

A domain event is an event significant to the domain.  
Used to decouple bounded contexts, or, allow uni-directional references between contexts.
Should be named in past tense due to it's something that has happend in the domain.
Don't use domain classes to represent event data.  
Do use primitive types to represent event data.

## General Principals

Follow YAGNI to avoid trying to predict future needs of the business.
Code only what is needed. This shortens the development time.

Follow KISS to avoid creating overly complex solutions
E.g. only begin to use inheritance and generics when there's actually a use-case for them
This results in more maintainable code

## Testing

Be pragmatic.  
At first, cover only the most essential parts.  
Use the structure Arrange, Act, Assert.

## Core Concepts

### Entities

Have it's own identity and can live independently of other objects.  
Equality checks should be implemented in the entity class itself, or, in an Entity base class.

### Value objects

Must be immutable. If value objects have the same member values, they can be treated interchangably.  
Value objects cannot have a life on its own. Must live within an entity.  
Prefer Value Objects to Entities, and place most business loginc here.

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