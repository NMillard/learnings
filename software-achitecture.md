# Architecture

Software architecture is

1. high-level
2. structure and organization of software
3. Layers (vertical)
4. Components (horizontal)
5. Relationships of the above

## Clean Architecture

Ìs layered and easily understood and testable.
Is designed for the "inhabitants" of the system
Is focused only on the essential, avoiding premature optimization.
Is aligned with business goals.

## Domain-centric architecture

Is the opposition to the generic database-centric architecture, ie. 1. Client, 2. Business Logic, and 3. Data Layer
In domain-centric, the database (persistence) is simply an implementation detail.

There is a "generic" layout for domain-centric architectures, and it goes like:

1. Domain
2. Persistence
3. Application
4. User Interface (Presentation)

The first being at the center and then moving outwards.

### Application layer

At its core, the application layer is responsibile to implement use cases.
It knows about the domain, but has no knowledge of persistence, presentation, or infrastructure.
Events, commands, and queries live here.

The application also provides interfaces to e.g. a data service (like DbContext in .NET) which the persistence layer will implement.

### Commands and Queries

These belong to the application layer.
Commands does something - i.e. modifies state in the system. But does not return anything.
Queries answer questions - i.e. no state modification, but returns a value.

### Structuring code

Structuring code according to functional cohesion may be easier to maintain, as it adheres to how
the "inhabitants" think about the system.

Usually, we see structure that follows a specific paradigm, like MVC. We'll have folders called Models, Views, and Controllers.
The structure is too generic to carry any meaning of the intent of the system.

## Problem Domain

Make an effort to understand the problem domain, its entities and workflows.
