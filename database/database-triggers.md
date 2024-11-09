# Database triggers

> [!NOTE]
> These notes are related to how triggers work in postgresql.

Triggers offer a means to react to events that occur in your application.

Triggers are created on tables and trigger functions have access to special local variables named
`TG_something`. The `something` part can for instance be `op`, which tells which action triggered
the function, such as `INSERT`, `UPDATE`, etc.

## Good practices

Since triggers fire and execute code when an event occurs, it can very quickly become opaque to the 
client about what is happening behind the scenes.

I follow these practices to reduce the potential headaches that triggers may cause.

1. No trigger on transaction/entity tables.
2. Triggers may only be created on special "trigger" tables so that the client is aware that
   inserts/update/deletes will cause side effects.
3. Triggers must be well-documented.

## Links
- https://www.postgresql.org/docs/current/plpgsql-trigger.html