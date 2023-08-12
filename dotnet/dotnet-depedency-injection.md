# .NET Dependency Injection

Definition: a set of software design principles and patterns that enable us to develop loosely coupled code.


## DI Patterns
- Constructor injection
- Property injection
- Method injection
- Ambient injection
- Service Locator


## DI Containers
Generally DI containers handle:
- Auto-registration
- Auto-wiring
- Object lifespan


## DI Boostrapper
The bootstrapper is where all the object composition is done.


## Best Practices
DO NOT let objects new up other objects in e.g. the constructor.  
DO take dependencies as constructor parameters.

DO NOT use auto registration
DO be explicit about which concrete types will be registered to interfaces