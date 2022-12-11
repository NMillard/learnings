# Creating Azure Functions

Az Func is a "serverless application platform" that allows us to run small pieces of code in the cloud.  
Server management is delegated to the cloud provider in Functions as a Service (FaaS) setup.  

Three sections in the exam:
1. Implement function triggers by using data operations, timers, and webhooks.
2. Implement input and output bindings for a function.
3. Implement Azure Durable functions.

### Azure Function App
One or more Azure functions that are developed, deployed, and hosted as a group.  
Different programming languages can be used. C# and JavaScript are the most common ones.  

#### Configuration
C# Function apps starts with two configuration files:
1. [`host.json`](https://docs.microsoft.com/en-us/azure/azure-functions/functions-host-json): configuration options that affect all functions in a function app instance.
2. [`local.settings.json`](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=v4%2Cwindows%2Ccsharp%2Cportal%2Cbash#local-settings-file): used for local development with "Core Tools" to set e.g. environment variables. This file is not copied to the publish folder.
    Application Settings are set inside the `Values` property.

### Hosting
Azure Functions usually runs in an Azure App Service plan.  
Consumption plan: serverless pricing, only costs money when run and scales automatically. Functions can at max run for 5 minutes.  
App Service plan: traditional plan, pay monthly for compute resources.  
Premium plan: enhanced security and performance + reserved instances.  
Docker container: runs in on any container enabled infrastructure.  


## Triggers
- HTTP Request trigger: use for APIs and Webhooks.
- Timer Trigger: scheduled tasks.
- Queue Trigger: run when message is published to queue.
- CosmosDB Trigger: run when new document is created.
- Blob Trigger: trigger on blob storage upload.

HTTP triggers are secured by either:
1. Anonymous: no key required
2. Function: key per function
3. Admin: key per function app

## Durable Functions
An extension to Azure Functions that create stateful, serverless workflows aka. Orchestrations.  
Three types:
- Client (Starter) Function: initiates a new orchestration, using any trigger.
- Orchestration Function: defines the steps in the workflow. Able to handle errors. An orchestrator doesn't perform any of the steps itself.
- Activity Function: implement a step in the workflow.

Requires a dependency to `Microsoft.Azure.WebJobs.Extensions.DurableTask` nuget.

### Patterns afforded by orchestrations

**Function chaining**: execute a sequence of activity functions in a specified order. The orchestration function keeps track of where we are in the sequence.

**Fan-out Fan-in**: run multiple activity functions in parallel and wait for them to finish. Activity results can be aggregated to compute a final result.

**Asynchronous HTTP APIs**: e.g. repeatedly pull an API for progress. Orchestration can manage the API pulling and stops when completed or timed-out.

**Monitor pattern**: implements a reoccuring process in a workflow. e.g. look for change in state, such as repeatedly calling an activity function to check if some conditions are met.

**Human interaction**: e.g. request approval and wait for staff to take action.


## Bindings
**Input bindings** is how we get data into our functions.  

**Output bindings** how we send data, messages, add documents, etc.

## Deployment
First you need the published version, by running `dotnet publish -c Release -o pub`.  
Then, zip the `pub` folder and run `az functionapp deploy --src-path pub.zip --name az-func-name -g rg-name`

