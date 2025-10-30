# Core Software Design principles

I'm listing a series of principles that I tend to follow when designing software.
These principles are gathered through hard-earned real-life experiences, reading
articles, and books such as "A Philosphy of Software Design", "Code Complete 2",
"Domain Driven Design", and so forth. Only principles I have found
to actually work in real projects are included here.

We also need to consider these principles' scope and context. Many of them are aimed
at developing software for collaborative scalability and applications with a longer
life-span. One-off projects and short-lived ones that only last, say a year, probably won't live long enough
to reap the benefits the following principles provide.


## Reduce complexity.
Complexity is anything that makes understanding and modifying a system hard.

## Design interfaces for the common case.


## Design for the future, build for today.

I generally tend to design for extensibility from the get-go. Asking questions such as
"what's the likelihood of this having to change in the next 2 years?" and "what would
the impact of such change be?" can go a long way. When building the software, however,
I always focus on current needs. Focusing on current needs with extensibility in mind
allows you to free yourself from the concrete implementations of today.