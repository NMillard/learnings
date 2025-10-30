# Busines Policy and Rules

In short, business policies are stable over time and abstract while rules are volatile and specific.

Back as a junior developer, what constitued as business logic, policies, and rules was quite confusing to me. It's all code, so how do determine if some line of code is one or the other?

I will try to shed some light on how I've now come to think of these terms, how they are related and which aspects that are different.

## Business policies
In general, you're well off thinking about policies as overarching guidelines or principles, or, rough constraints within some area or process–essentially a high-level statement.

A policy provides a framework for decision-making and ensures consistency and compliance.

You can rarely implement a business policy since it's simply too abstract. An example would be "usernames must comply with our rules". That statement is not really that helpful to a developer but is at the same time quite precise in a business context.


## Business rules
Rules are the implementation details of a policy, that may change at any time. Over time, a rule can be redefined in its entirety, or perhaps just have its parameters adjusted. Either way, rules are volatile and will change far more often than policies.

Typically, business rules are either trivial or evolving. Trival rules are those which has little reason to change over time and are quite easy to implement, such as in the form of guard clauses.

Due to the slow-changing nature of trivial rules, using dedicated,