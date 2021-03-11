# Logging

Logs should ideally be captured in an easily searched, pre-defined structure.

## Types of log entries
**Usage**
How often are certain features used.

**Performance**
How's our application performance - do we have bottle necks?

**Errors**
Which errors are thrown and when. What were the parameters?
Who called the function? What credentials, etc. did the user have?

**Diagnostics**
Adhoc troubleshooting.

## Information to log
**Place in our code**
What's the log's context.

**Relevant user information**
Anon or authenticated, authorizations, roles or groups.

**Additional**
Time elapsed, method parameters, class types, browser used, etc.

## Logging strategies
Consider using one-logger-per-type strategy. Allows for simple development, and intent-revealing code.
Such as, Log.PerformanceLog(message), Log.Error(message), Log.Usage(message), etc.
