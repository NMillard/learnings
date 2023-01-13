# ASP.NET Core Best Practices

## Creating APIs
Avoid exposing internal details. An API request and response shouldn't be coupled to:
- Database table schema
- Persistence format
- Business domain entities

An API is considered representation, so, leverage encapsulation and isolate business entities and implementations from the API.  

When building APIs for clients you control, and you can update API and client in tandenm, then let the client dictate what data to return.

### Characteristics of good API models
- Naming: has an intention revealing name. Use model and property names that make sense to clients. Don't reveal internal architecture.
- Factored: isolated from other models. A model per endpoint is a good approach. Have a model for each concept.
- Size: include what you'd expect. Not more, not less.
- Stable: once the API model is public, it doesn't change--if that's not possible, then try only to add properties. Changing or removing properties is a no-go.

### Reduce controller size
Request-Delegate-Response pattern.  
Move logic into business layer.  

Use filters to apply policies, middleware to intercept the request, formatters to apply custom output formatting, and binding to parse input.

Some advice on building controllers:  
- Small controllers are good for scalable code (multiple developers working on the same solution).
- Small controllers are easy to test due to little to no logic.
- Aim for more small controller classes over few large ones.
- Avoid business logic and data access in controllers, no matter the size.
- Move duplicate logic into shared filters, middleware, and policies.
- Avoid branching.

Controller smells (code smells specific to controller design):
- New functionality requires new dependencies.
- Grows over time.
- Low cohesion - the actions don't represent a common concept.
- Dependency per action (endpoint).