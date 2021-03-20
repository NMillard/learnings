# Software Development Learnings

## Courses

Tactical Design Patterns in .NET Control Flow and Making your C# Code More OO

## Branching

Avoid branching using IF-ELSE statements. Use conditionals to test conditions, and nothing else. But keep them out of client (calling) code.

Use state objects to keep track of another object’s state. This is done using interfaces and creating state objects that implements the interface.

Also, do implement special case objects - e.g. null objects.

## Infrastructural Code

Wrap this code in separate objects – but only if it is likely to never change in the future.

Loops themselves are infrastructure, and should not be displayed in business logic code. Operations inside the loop are the logic.

Use Object substitution instead of conditional branch execution.

## Abstraction

Abstract work away from the client calling the code – place in separate objects.

Remove loops from calling code – place in separate objects that knows how to deal with the loops etc.

Client code should only deal with customer requirements – all code that is concerned with supporting those requirements should go into separate objects.

‘Never’ rely on concrete classes. ‘Always’ use interfaces.

## Dealing with Future Requirements

Use strategy interfaces and objects to deal with volatile requirements – don’t change existing classes. Add one more class instead. Replace objects at run-time to select implementation of a requirement.

Generalize classes as much as possible.

## Message Chaining

A message-chain is when a class knows about another class’ collaborators, and make calls to those collaborators. This should be avoided.
Fix by creating new methods on the immediate collaborator’s class that return values from its own collaborators.

## Layered Achitecture

Use layered architecture that will use the correct level of abstraction at each layer.

Layers may be (going from the calling client to the abstract)

Presentation layer (View)  
Application layer (Controller)  
Service layer (Services – Business logic)  
Policy layer (Policies – Business rules)  
Models layer (Entities)  
Data Access layer (Repositories)

## Creating Objects - Client-First approach

Start by looking at the client, and simulate how the ‘perfect’ call would be e.g. in a controller – and then start to dig into which objects that might hide behind this perfect call.
Think in terms of which business problem this call is related to and which objects that need to be in place in order to solve it.

This approach will keep one focused at what’s important to solve.

## 3rd Party Code

Wrap third party code into separate objects. Might seem inefficient and not needed at the beginning, but is a good way to protect own code and avoid different coding styles.

## Database Seeding

Always create seperate seeding files--e.g. SQL files--containing the seeding data.
Never make seed data explicitly rely on specifc foregin key IDs, as these might differ from database to database.

## Unit tests as Documentation

Writing unit tests may be the best kind of documentation, because you see the real code in action and how it's meant to be used.

## API Design

Think of what the user of the API wants to achieve and how they want to.
API consumers experience is the most important measure of quality.

### HTTT API components

- Resource: representation of a certain domain. This can be an object on which actions can be performed.
- Collections: a set of resources
- Endpoint: the path through which the resource can be located.

### General HTTP Verbs' use

GET: request a resource without producing any side-effects.
POST: request to create a resource.
PUT: request to update an exisiting resource, or update if it already exists.
DELETE: request to remove a resource (or safe delete it)

### Task-based interface

- Build simple end points that handle just one functionality.
- Split DTOs into several - this avoids having unused DTO properties in some scenarios.
- Use Commands and Queries tasks.
  - Query: will only retrieve items, and has no side-effects.
  - Command: will only perform an action, and returns nothing but perhaps a status.

# Authentication and Authorization

Let one service authenticate the user, and get the claims from another (context specific) service

# Defensive coding

Actively minimize the chances of bugs by writing code that is:

- Clean and understandable
- Testable
- Predictable

# Extensible code

Writing code that can be extended without modifying existing code, but instead, by writing new classes and modules.

- Write interfaces that can be refereced
- Write implmentations of the interfaces
- Write provider classes that read config files to know which class you instantiate
