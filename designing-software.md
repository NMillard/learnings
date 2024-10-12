---
title: Practices of Designing Applications
seo: Practices and approaches to well designed application code
tags: engineering, software development, software engineering, programming, technology
---

## Intangible qualities of good design

Good design feels flawless. It is the absence of awkwardness and ambiguity. It inspires _understanding_. Good design removes the clutter and deepens the reader's knowledge. Pieces just effortlessly "snap" together. Good design enables the IDE to help its author. You have a light, impactful, and productive feeling when working with perfectly designed software.

No guessing. No, what the f-moments. Just lots of positive surprises.

I've found the best code to invoke a "can it really be that easy?" response.

---

The following concepts have served me well when designing software and reviewing others' code. It provides a nice, practical foundation to ground your arguments in real-life experiences and best practices.

## Identify SOLID violations

It seems to have become a trend to bash on SOLID, a battle-tested, (almost) multi-decade old collection of design practices. Criticism aimed at SOLID is typically in the ballpark of "it leads to over-engineering," or "it creates overly abstract code." While that may be occasionally true, knowing when you're violating SOLID principles can help you enormously.

- Code Fragility<br/>
  Software breaks in many places with every change. Unrelated parts of the project start to behave differently when a change is introduced.

- Code Rigidity<br/>
  Software that is difficult to change--ripple effects to other classes. The codebase typically lacks abstractions. Modules and classes are highly coupled.

- Technical debt<br/>
  Typically the symptom of prioritizing feature delivery over code quality for extended periods. You should minimize technical debt by incorporating a refactor-loop into your regular workflow.

## Reduce message chaining

Poor encapsulation is often seen when an object knows about its collaborating objects' dependencies and directly accesses them. This practice should be aggressively reduced. This is one of the quickest ways to introduce high coupling, shotgun surgery refactoring, unforeseen ripple effects, poor estimates, and code rot.

It's fine to call methods on collaborating classes, but never to access internal state or collaborating objects.

Luckily, there's an easy fix. Create methods on immediate collaborators that return values from its own collaborators.

## Set clearly defined scopes

Not every application, module, or class is created equal. Be explicit about a module's scope. Is it used as part of a framework? Do other developers need to instantiate it? Is it a building block in a larger setup? Is it long or short-lived? Is it client-facing and thereby public or purely internal to the module?

Knowing the answer to each and every of these questions lets you design better software.

## Avoid clustered aggregates

A lot of the discussion in Domain Driven Design (DDD) revolves around "aggregates" and "aggregate root". An aggregate is a transactional boundary.

TK - more on aggregates and why clustered aggregates are considered poor practice. False invariants and compositional convenience.

## Duplication

There's a general rule of thumb that writing the same piece of code twice may be okay, but no more than that. However, there are some things that shouldn't be consolidated into shared code:

- Domain specifics<br/>
  Keep domain-specific code in its correct place. Just because two separate services or applications need a `User` class it doesn't necessarily mean the `User` should be packaged in a separate project and shared between the services.

- Temporal similarities<br/>
  Sometimes you will see code that is identical in two different places. The junior developer will quickly refactor this, and often leads to static helpers, and so on. But many duplications are only temporal, and it makes sense to keep two--though identical--separate implementations.

- Fields and properties<br/>
  It's normal to have many classes that share common traits and thereby have the same fields and properties--sadly, this also leads to base classes that are nothing more than glorified property bags. Opt for duplication over inheritance if one class can't be a direct substitute for the base type.

## Manage orchestrations

We see orchestrations happening in many `Service` classes. They take a bunch of dependencies and orchestrate their interaction. E.g., fetch an object from the database, pass it to another object, inspect the result, maybe run some business logic, and then return a result.

Orchestrators often take a dependency on other services solely to notify them that something happened. This
coupling can be eliminated just by publishing an in-process event and let interested services listen to that event.

## Managing constructor dependencies

Inversion of control combined with dependency injection is an immensely valuable practice. Though, it may lead to constructor bloat if you're not careful.

I often see classes that take dependencies through the constructor, and only use the dependency in a single method. In this case, consider moving the dependency from the constructor to the method argument instead--or, even better, perform a class extract refactoring, by creating a new class with a single method and constructor dependency.

## Lean over fat interfaces

Prefer to have many lean interfaces over a few fat ones. A lean interface has a defined scope, few methods, and high cohesion. You can often spot poorly designed interfaces by:

- Having many unrelated methods.
- Implementing classes throws an exception rather than implementing the method,
- or has an empty method body.
- Client classes have access to irrelevant methods that come with the bloated interface.

It's better to have many smaller interfaces, even if there's only one class implementing them all. It allows for future flexibility and adaptability.

## Avoid Util and Helper classes entirely

Creating Utility and Helper classes is nearly always a poor practice, and a symptom of an ineffective architecture or facile design efforts.

Modules using Utility and Helper classes become highly coupled with the internal workings, and you can't easily swap implementations. Utility and Helper classes are ever-growing and generally have many reasons to change. Changes that cause ripples throughout the codebase.

If you find yourself relying on utility and helper classes, you should consider taking a step back, analyze why you need them and how you can get your codebase's health back on track.

## Allow things to change

Classes should be open for extension but closed for modification, as in accordance with the Open-Closed principle. This principle goes hand-in-hand with dependency injection and inversion of control.

TK link to article on change

Decide which parts of a class that should be "swappable" or extensible, and then build the mechanics to achieve that.

New functionality is followed by new classes, not modifications to existing classes. It's always safer to introduce a class, because nothing generally uses it--though, if you use techniques such as service discovery, then reflection may actually pick up new classes automatically.

Allowing things to change is also about enabling and disabling functionality. With that in mind, you'll want to
introduce ways to toggle modules, classes, and so, on and off. Additionally, you need to make this happen at runtime. As a rule of thumb, modern software shouldn't be switched off to enable or disable functionality that is already implemented.

By doing this, you'll:

- Introduce new functionality with minimal risk.
- Fewer regression bugs since you're not touching existing code.
- Enforcing decoupling through abstractions. Your classes depend on contracts rather than concrete implementations.
- Speed up development time by focusing on the task at hand rather than understanding previous work.
- Bettering the developer experience by letting the developer focus on abstractions' intent rather than all the concrete implementations.

## Shield off external changes

Don't let third-party changes drip into your codebase. Create an anti-corruption layer around your domain and application code by having specialized data-transfer objects at the border of your application.

This is typically done by having one or more interfaces that dictate the interaction with external systems. This allows you to easily create a new concrete implementation in case of changes to the external system.

The anti-corruption layer should be in place anywhere you receive unmanaged data into your application. This gives you time to:

- **inspect** what the data looks like,
- **validate** its correctness according to your expectations,
- and **decide** if the data should make its way into your application, by being mapped to domain classes.

Not only does this allow you to isolate your application from unwanted changes, but it also allows you to mock external calls and interactions, making unit testing easier and more reliable.

## Managing validation rules

It's incredibly common in business-line applications to have a series of business rules that needs to be validated as part of a business policy.

Say, our business has a policy that says "all usernames must be valid," and then we have a bunch of business rules that checks username validity.

Each rule becomes a class in its own right, ideally performing only a single validation check and returns a result. So, we'll have the following player categories (classes): `Policy`, `Rule`, and `ValidationError`.

Don't fear having lots of classes with a single responsibility. Fear having a single class with lots of
responsibilities.

## Apply adhoc polymorphism

While real object polymorphism is great and allows for concise code, don't forget about adhoc polymorphism; that is method overloading and overriding.

## Use intention-revealing names

We've all noticed just how hard it is to name things. Admittedly, we sometimes resort to the "quick" and "safe" naming strategy, such as appending `Handler`, `Service`, and `Manager` to any class and hope our fellow craftsmen know what we meant.

Names such as `Handler`, `Service`, and `Manager` denotes a broad range of categories, and are mostly too abstract to be actually helpful five years down the line.

## Perfectly documented code

There's just something about code that has high quality documentation. It's neat and it speeds up development tremendously.

Imagine learning ASP.NET or React without using their documentation sites or reading the accompanying code
documentation. Good luck. You'll probably get absolutely nowhere.

High-quality code documentation strikes a balance between brevity and clarity. It covers the most common use cases and may even explain _why_ it exists.

## Know you code will proliferate

Any code committed to a repository will at some point be copied–for better or worse–by developers on your team or other teams. You can't really fight it, nor should you. But, you'll have to focus on ensuring the quality that you and others commit adheres to a high standard.

Here are a few rules of thumb that I go by whenever I evaluate if code is good enough, and if I'd like it to be copied by others:

- The code must be idiomatic. Follow the language's modern conventions. Conventions change as time goes by and being stuck with "what-was" as opposed to "what-is" is an important consideration. Stick by the modern conventions.
- Only commit code that is analyzed by a static analysis tool such as SonarQube. You can easily spin up a local SonarQube container instance and run your code through it.
- Don't allow code that allows invalid states. For instance, having a public default constructor.
- Remove dependencies that are used just to gain access to this one class or constants that you need. When adding a dependency, rest assured that others will use it.


## Resources
- https://humanwhocodes.com/blog/2015/05/14/the-bunny-theory-of-code/ by Nicholas C. Zakas