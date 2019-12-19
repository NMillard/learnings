# Async .NET Programming
Off-load tasks to a background thread to free the main thread
Suitable for I/O operations such as disk, memory, web APIs, and database calls

# Async Await
async await creates a state machine

Marking methods async allows the use of await in the method body
Await will return control to the caller
Await can be placed in front of any method returning a Task
Continuation is all code executed after an async operation is completed
Continuation is executed on the original context (ie. thread) that called the async metuyhod
Using ContiueWith() on a Task will have the continuation running on a different thread

## Don'ts
Create async void methods.
    - Only use async void for event handlers
Use .Result or .Wait() as they are blocking the thread

## Dos
Create async Task if nothing is returned from the async method
"Always" await async methods
If async operations are not awaited, validate tasks by chaining on a continuation

## Exceptions
Exceptions won't get propagated back to the caller if await is not used
Exceptions won't get propagated if the method is async void - there's no way to catch exceptions in this case
ContinueWith() will execute even if there was an exception thrown the the original Task

## Cancellation Tokens
Use cancellation tokens to signal a task cancellation
Cancellations must be handled in the Task

## ASP.NET Core
Don't use Task.Run() too often due to it will get a thread from the threadpool which could have been used for serving web requests instead.


# Parallelism .NET
Suitable for CPU-bound operations and independent chunks of data

## Using objects in parallel
Must be thread-safe
ie. ConcurrentBag, ConcurrentDictionary, etc.

## Spawning child tasks
To avoid using the async await keywords, you can use the TaskFactory to create new tasks
This allows you to set the TaskCreationOptions and set it to AttachToParent

## ASP.NET Core
Be careful in ASP.NET apps. A parallel job may take all threads on all cores.

# State machine
Keeps track of tasks
Executes continuation
Ensures continuation executes on correct context
Handles context switching
Report errors