# Practices of Designing Applications

## Intangible qualities of good design
Good design feels flawless. It is the absense of awkwardness and ambiquity. It inspires _understanding_. Good design removes the clutter and deepens the readers knowledge. Pieces just effortlessly "snap" together. Good design enables the IDE to help its author. You have a light, impactful, and productive feeling when working with perfectly designed software.

No guessing. No what the f-moments. Just lots of positive surprises.

You'll notice that you're asking yourself "can it really be that easy?". And yes, yes it can.

## Identify SOLID violations
- Code Fragility  
Software breaks in many places with every change. Unrelated parts of the project starts to behave differently when a change is introduced.

- Code Rigidity  
Software that is difficult to change--ripple effects to other classes. The codebase typically lacks abstractions. Modules and classes are highly coupled.

- Technical debt  
Typically the symptom of prioritizing feature delivery over code quality for extended periods. You should minimize technical debt by incorporating a refactor-loop into your regular workflow.

## Reduce message chaining
Poor encapsulation is often seen when an object knows about its collaborating objects' dependencies and directly accesses them. This practice should be agressively reduced. This is one of the quickest ways to introduce high coupling, shotgun surgery refactoring, unforeseen ripple effects, poor estimates, and code rot.

It's fine to call methods on collaborting classes, but never to access internal state or collaborating objects.

Luckily, there's an easy fix. Create methods on immediate collaborators that return values from its own collaborators.

## Duplication
There's a general rule of thumb that writing the same piece of code twice may be okay, but no more than that. However, there are some things that shouldn't be consolidated into sharable code:
- Domain specifics  
Keep domain specific code in its correct place. Just because two separate services or applications need a `User` class it doesn't necessarily mean the `User` should be packaged in a separate project and shared between the services.
- Temporal similarities  
Sometimes you will see code that is identical in two different places. The junior developer will quickly refactor this, and often leads to static helpers, and so. But, many duplications are only temporal, and it makes sense to keep two--though identical--separate implementations.

## Manage orchestrations
We see orchestrations happening in many `Service` classes. They take a bunch of dependencies and orchestrate their interaction. E.g. fetch an object from the database, pass it to another object, inspect the result, maybe run some business logic, and then return a result.

Orchestrators often need to notify other services that something happenend, and take each service as a dependency. This coupling can be eliminated just by publishing an event, and let interested services listen for that event.

## Managing constructor dependencies
Inversion of control combined with dependency injection is an immensely valuable practice. Though, it may lead to constructor bloat if you're not careful.

I often see classes that take dependencies through the constructor, and only uses the dependency in a single method. In this case, consider moving the dependency from the constructor to the method argument instead.

## Avoid Util and Helper classes
Creating Utility and Helper classes is nearly always a poor practice, and a symptom of an ineffective architecture or facile design efforts.  

Modules using Utility and Helper classes become highly coupled with the internal workings, and you cannot easily swap implementations. Utility and Helper classes are ever-growing and generally have many reasons to change. Changes that causes ripples throughout the codebase.

## Allowing things to change
Classes should be open for extension but closed for modification, as in accordance with the Open-Closed principle. This principle goes hand-in-hand with dependency injection and inversion of control.

Decide which parts of a class that should be "swappable" or extensible, and then build the mechanics to achieve that.

New functionality is followed by new classes, not modifications to existing classes. It's always safer to introduce a class, because nothing generally uses it--though, if you use techniques such as service discovery, then new classes may actually be picked up automatically by reflection.

Allowing things to change is also about enabling and disabling functionality. With that in mind, you'll want to introduce ways to toggle modules, classes, and so, on and off. Additionally, you need to make this happen at runtime. As a rule of thumb, modern software shouldn't be switched off to enable or disable functionality that is already implemented.

By doing this, you'll:
- Introduce new functionality with minimal risk.
- Fewer regression bugs since you're not touching existing code.
- Enforcing decoupling through abstractions. Your classes depend on contracts rather than concrete implementations.
- Speed up development time by focusing on the task at hand rather than understanding previous work.
- Bettering the developer experience by letting the developer focus on abstractions' intent rather than all the concrete implementations.

## Managing validation rules
It's incredibly common in business-line applications to have a series of business rules that needs to be validated as part of a business policy.

Say, our business has a policy that says "all usernames must be valid", and then we have a bunch of business rules that checks username validity.

Each rule become a class in its own right, ideally performing only a single validation check and returns a result.

Don't fear having lots of classes with a single responsibility. Fear having a single class with lots of responsibilities.

## Set clearly defined scopes
Not every application, module, or class is created equal. Be explicit about a module's scope. Is it used as part of a framework? Do other developer need to instantiate it? Is it a building block in a larger setup? Is it long or short lived? Is it client-facing and thereby public or purely internal to the module?

Knowing the answer to each and every of these questions let you design better software.

## Intention-revealing names
We've all noticed just how hard it is to name things. Admittedly, we sometimes resort til the "quick" and "safe" naming strategy, such as appending `Handler`, `Service`, and `Manager` to any class and hope our fellow craftsmen know what we meant.

 Names such as `Handler`, `Service`, and `Manager` denotes a broad range of categories, and are mostly too abstract to be actually helpful five years down the line.