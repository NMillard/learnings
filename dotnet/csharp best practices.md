Living document that should be updated as new best practices are discovered or becoming outdated

# Be patient
Don't rush into coding
Understand the domain before coding
Do diagrams and object-oriented analysis up front


# Process
Use a client-first approach - starting from the most outer entrypoint


# Use the domain language
Name classes, veriables, etc. something meaningful in relation to the domain.


# Objects
All objects should have a private constructor
If an object is responsible for its own creation, a factory method is preferred over the constructor
    This allows you to name the "constructor"--ie factory method--something meaningful in regards to the domain
All classes should be restricted to internal access
Only interfaces should be publicly accessed
Code against an interface, not concrete implementations
Make interfaces small
Let the client decide the interface


# Domain objects
Objects must model some part of the domain
If an object can live independently and two instantiations of a class with same properties are still considered different in the domain, it is an entity
Entities should have an identity property
Identity properties may never be updated
Use immutabile value objects to represent domain specifc elements where two instantiations of the same class with same properties are considered equal
Do not write code to satisfy other libraries - e.g. using attributes on properties to satisfy EntityFrameworkCore

Don't try to completely eliminate immutable state - just try to minimize it


# Object creation
Avoid using constructors
Prefer using factory methods or classes
Factory methods must have descriptive names relating to the domain


# Methods
Avoid returning null
Use "null-objects" instead of actually returning null - this may e.g. be returning a instance of EmptyResult


# Flexible code
Flexible code is great - but don't write code that is anticipating future needs if not verified
Try always to list constraints for Generic parameters


# Solution and project structure
It's prefered to have many projects within a solution, than few projects with many responsibilities
A project should only cover one part of a domain

The typical solution structure
- DataLayer
- Domain
- Application (with separation of e.g. front-end client, WebAPI, MessageQueue deamon, etc.)

Folder structure may look like this:
Root
- src/projects
- tests/test projects

# Exception handling
Only throw exceptions if something truely exceptional happend
Strive to handle all exceptions, except for the base Exception class


# Branching
Simple branching is okay
Prefer polymorphism over branching code

# Asyncronous programming